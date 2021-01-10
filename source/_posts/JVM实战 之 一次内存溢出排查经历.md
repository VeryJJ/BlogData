---
title: JVM实战 之 一次内存溢出排查经历
tags: [Java, JVM, 内存]
date: 2020-10-15 23:09:11
categories: 技术
---


# 源于线上应用事件

某天团队同学收到线上系统报警，web-abc真线有一台机down掉了。

为保留事故现场，做剩余应用做了dump。

然后开始分析

<!--more-->

<br>

# 问题排查

1、 经过对线上出问题周期内的应用日志逐条排查，未发现明显异常。

2、分析当时的dump快照，发现有两个类的实例数和总大小异常，这两个类是跟poi解析excel有关!这个很关键。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230016978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

3、再翻事发时间的真线日志，有所发现，16点11分51秒的时候，确实有流量请求一个上传excel的接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230150609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

4、对比常规接口的日志，事发情况下，只有start没有finish，说明这个请求僵住了，而自这个请求之后的时间，机器才出现不断fullgc的情况

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230212131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)

5、伪造一个excel大文件，在test环境上传，把test服务器搞挂，dump文件分析，与真线类似

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230357752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)


6、通过以上几点证据，判定为本次事件是由于用户上传excel文件出现的极端情况（文件过大，或者一直尝试），具体的情况日志已分析不出。

8、排查代码中POI读取Excel的方式，发现使用的是用户模式，这样会占用大量的内存；POI提供了2中读取Excel的模式，分别是：
- 用户模式：poi下的usermodel有关包，它对用户友好，有统一的接口在ss包下，但是它是把整个文件读取到内存中的，对于大量数据很容易内存溢出，所以只能用来处理相对较小量的数据；
- sax模式：poi下的eventusermodel包下，相对来说实现比较复杂，但是它处理速度快，占用内存少，可以用来处理海量的Excel数据。

POI sax模式只是改善，并不能完全解决这个问题，所以采用阿里开源easyexcel解决优化。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230508784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)
<br>

# 修改优化
1、poi解析excel的方式改为easyexcel方式

2、排查后端文件上传、下载的场景，进行大文件极端场景测试

<br>

# 整改方案
Java领域解析、生成Excel比较有名的框架有Apache poi、jxl等。但他们都存在一个严重的问题就是非常的耗内存。如果并发量大或excel文件过大，则会OOM或者JVM频繁的full gc。POI的SAX模式虽有改善，但相对比较复杂，excel有03和07两种版本，两个版本数据存储方式截然不同，sax解析方式也各不一样。而阿里开源的easyexcel很好地解决了解析excel大量消耗内存的问题，而且使用简单，功能上也可以覆盖我们的业务需求。
<br>
## easyexcel核心原理
**1、文件解压文件读取通过文件形式**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230611873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)


**2、观察者模式按行解析**
easyExcel采用一行一行的解析模式，并将一行的解析结果以观察者的模式通知处理（AnalysisEventListener）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015230636250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzMxODM2Nw==,size_16,color_FFFFFF,t_70#pic_center)


**3、抛弃不重要数据**
Excel解析时候会包含样式，字体，宽度等数据，但这些数据是我们不关心的，如果将这部分数据抛弃可以大大降低内存使用。Excel中数据如下Style占了相当大的空间。

<br>

## easyexcel的使用：
请自行百度

推荐读物：https://alibaba-easyexcel.github.io/quickstart/faq.html