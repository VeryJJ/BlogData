---
title: 你不得不知的Spring事务失效 8 大原因
tags: [Spring,Java,事务]
date: 2020-12-15 23:25:30
categories: 工具
---


#### 1、数据库引擎不支持事务
<br>

这里以 MySQL 为例，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB。
<br>
根据 MySQL 的官方文档：

> https://dev.mysql.com/doc/refman/5.5/en/storage-engine-setting.html

从 MySQL 5.5.5 开始的默认存储引擎是：InnoDB，之前默认的都是：MyISAM，所以这点要值得注意，底层引擎不支持事务再怎么搞都是白搭。

<!--more-->

<br>

#### 2、没有被 Spring 管理
<br>

如下面例子所示：
```java
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```
如果此时把 @Service 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。
<br>

#### 3、方法不是 public 的
<br>

以下来自 Spring 官方文档：

> When using proxies, you should apply the @Transactional annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the @Transactional annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ (see below) if you need to annotate non-public methods.

大概意思就是 @Transactional 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 AspectJ 代理模式。

<br>

#### 4、类内部自身调用问题
<br>


来看两个示例：
```java
@Service
public class OrderServiceImpl implements OrderService {

    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```
update方法上面没有加 @Transactional 注解，调用有 @Transactional 注解的 updateOrder 方法，updateOrder 方法上的事务管用吗？
<br>
再来看下面这个例子：
```java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateOrder(Order order) {
        // update order
    }
}
```
这次在 update 方法上加了 @Transactional，updateOrder 加了 REQUIRES_NEW 新开启一个事务，那么新开的事务管用么？
<br>
这两个例子的答案是：不管用！
<br>
因为它们发生了自身调用，就调该类自己的方法，而没有经过 Spring 的代理类，默认只有在外部调用事务才会生效，这也是老生常谈的经典问题了。

这个的解决方案之一就是在的类中注入自己，用注入的对象再调用另外一个方法，这个不太优雅

另外一个可行的方案如下：
举个简单的例子：
```java
@Service
public class ServiceA {

  @Transactional
  public void doSomething(){
    
    向数据库中添加数据;
    
    调用其他系统;
  }
}
```
这里就用伪代码来做示例了，当我们执行了“向数据库中添加数据”，我们去数据库中查询，发现并没有我们添加的数据，但是当我们的service这个方法执行完成之后，数据库中就有这条数据了，这是由于数据库的隔离性造成的。
<br>
我们将代码修改一下：
```java
@Service
public class ServiceA {

  @Autowired
  private ServiceB serviceB;
  @Transactional
  public void doSomething(){
    
    serviceB.insert();
    
    调用其他系统;
  }
}

@Service
public class ServiceB {

  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void insert(){
    向数据库中添加数据;
  }
}
```

我们将要事务分离出来的方法写在另一个service中，再次测试，发现执行完插入语句之后，数据库中就已经能查到数据了，说明事务分离了，完成了我们的需求。

当然 Spring 其实也考虑这个，在 Spring 的配置中，我们只需要添加标签：
```xml
<aop:aspectj-autoproxy expose-proxy="true"/>
```

或者：
```xml
<aop:config expose-proxy="true">
```

并且在代码的调用中要求使用代理对象去调用即可：

```java
((ServiceA ) AopContext.currentProxy()).insert();
```

<br>

#### 5、数据源没有配置事务管理器
<br>

```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```
如上面所示，当前数据源若没有配置事务管理器，那也是白搭！

<br>

#### 6、不支持事务
<br>


来看下面这个例子：

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // update order
    }
}
```

Propagation.NOT_SUPPORTED： 表示不以事务运行，当前若存在事务则挂起，详细的可以参考《事务隔离级别和传播机制》这篇文章。

都主动不支持以事务方式运行了，那事务生效也是白搭！

<br>

#### 7、异常被吃了
<br>


这个也是出现比较多的场景：

```java
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {

        }
    }
}
```

把异常吃了，然后又不抛出来，事务怎么回滚吧！

<br>

#### 8、异常类型错误
<br>


上面的例子再抛出一个异常：

```java
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }
}
```

这样事务也是不生效的，因为默认回滚的是：RuntimeException和error
```java
public boolean rollbackOn(Throwable ex) {
        return (ex instanceof RuntimeException || ex instanceof Error);
    }
```

如果你想触发其他异常的回滚，需要在注解上配置一下，如：

```java
@Transactional(rollbackFor = Exception.class)
```

这个配置仅限于 Throwable 异常类及其子类。