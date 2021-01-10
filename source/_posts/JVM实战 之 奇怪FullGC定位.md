---
title: JVM实战 之 奇怪FullGC定位
tags: [Java, JVM, 内存]
date: 2020-10-15 23:54:39
categories: 技术
---

# 一、背景
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015223319828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center>图1 真线环境abc-center 堆使用率</Center>

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015223348400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center>图2 真线环境 abc-center 非堆使用率</Center>


由图1、图2 可见heap和permGen使用率都不高，但是abc-center 2台机器基本上每天会进行2次规律性的FullGC；

<!--more-->

<br>

# 二、问题排查

在线询价真线环境JVM参数配置
```shell
-Xms2g
-Xmx2g
-Xmn448m
-XX:SurvivorRatio=5
-XX:+UseParNewGC
-XX:+UseConcMarkSweepGC
-XX:+CMSClassUnloadingEnabled
-XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=0
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
-XX:+HeapDumpOnOutOfMemoryError
-XX:MetaspaceSize=300M
-XX:MaxMetaspaceSize=300M
```
查看真线JVM配置参数CMSInitiatingOccupancyFraction=75，配置的内存是2G，实际堆内存维持在570M左右，老年代内存1.5G，老年代内存使用率也低于75%，达不到触发FullGC的条件；基本上每次FullGC前后，堆内存也无明显变化；

团队内部分析讨论后，怀疑可能由于System.gc 引起的FullGC。根据pinpoint 触发FullGC的时间点，查看对应时间段的gc-log，确实是由于System.gc引起的FullGC；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015223808943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center> 图3 真线环境 -center gc日志</Center>

排除掉ABC项目中的程序调用System.gc, 开始猜测二方SDK，三方SDK引入的可能，例如nio包，但是没有找到具有参考价值的日志, 无法定位到具体的调用方法；

通过反复观察gc日志，发现这些fullgc日志都是周期性的，且周期为10小时，如图3所示；根据GC日志分析可知，应用每10小时触发Full GC的现象大概率是外部组件调用引起的，根据这个线索，开始索骥Google资料，了解到tomcat、nio、cxf部分版本可能会有类似周期性FullGC问题。有价值博客只有:[https://blog.csdn.net/qq_35963057/article/details/85236268](https://blog.csdn.net/qq_35963057/article/details/85236268)，同样的也是每隔10小时FullGC一次；因此大概率判断是由于apache的cxf包引起的FullGC；

<br>

# 三、定位分析

 根据上述分析，查看ABC的maven依赖，如图4，果然在ABC中找到了间接引入了apache-cxf的二方SDK包
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224357875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)
<Center> 图4 inquiry项目maven依赖树</Center>

由于在apache-cxf下的JDKBugHacks利用反射调用了sun.misc.GC中的requestLatency方法如图5所示，调用该方法会创建一个守护线程如图6；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224512777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center>图5 JDKBugHacks类的doHacks方法</Center>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224548208.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)


<Center>图6 GC类中创建守护线程</Center>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224557199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)


<Center>图7 run方法</Center>

<br>

由图7可知，正是由于在守护线程的run方法中调用了System.gc方法导致了FullGC，同时由GC.lock.wait可知，var1是图5 反射调用传入的36000000ms即10小时，当一次FullGC后，var4会变成0，

因此锁的等待时间是10小时，到这里就基本确定了ABC应用真线是就是由于Apache的cxf包下的JDKBugHacks反射调用导致的；这个GC线程是为了预防部分组件造成的内存泄露问题

(如javax.management.remote.rmi.RMIConnectorServer等会分配堆外内存)，通过调用sun.misc.GC类中requestLatency创建周期为10小时的GC守护线程，定期通知JVM去回收垃圾。

这个线程会先判断是否已有其他组件通过requestLatency来创建了GC线程，如果创建了就跳过，否则新建守护进程。根据类名JDKBugHacks，也能猜测这个类是为了修复一些JDK或组件已有的问题，但这个定期FGC在本应用并不需要；

<br>

由于是依赖的二方包引入的，ABC项目中没有显式的调用相关类，需要继续排查是什么地方调用了JDKBugHacks这个类；

排查发现是由于spring框架在初始化的时候进行了调用，如图8、图9所示(中间省略了几个调用节点)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224857258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center> 图8 PostProcessorRegistrationDelegate类</Center>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015224909735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center>图9   LogUtils类</Center>

<br>

# 四、解决方案

方案1、由图5可知，可以通过控制判断条件，让其不执行if 里面的逻辑

直接在启动项目的JVM参数中加入
-Dorg.apache.cxf.JDKBugHacks.gcRequestLatency=true   来使JDKBugHacks中的判断为false，然后跳过这段逻辑。

<br>
<br>

方案2、由于二方SDK引入的，实际二方包本身并没有使用到cxf包，maven将Apache-cxf包排除即可,如图10所示；


本次修改采用第二种解决方案；


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015225231482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

<Center>图11 真线堆内存</Center>

优化发布上线后，观察近三天确实不在出现FullGC； 

<br>