---
title: RESTful API命名实践
tags: [RESTful, Web]
date: 2018-05-18 10:29:30
categories: 技术
---


# 引言

在互联网高度普及的今天，作为一名Web开发者，如果你还没听说过“REST”这个技术名词，出门都不好意思跟人打招呼。尽管如此，对于REST这个泊来品的理解，大多数人仍然停留在“盲人摸象”的阶段。



有人认为，在Web Controller层写的API就是REST API。而且，从开发角度对于URI的命名、HTTP  Mehthod的选择没有建立起规范的意识。这样是不优雅的！（没有对错之分）


作为带着问题学习总结的我，未打算通过本篇文档全面的阐述清楚REST，而是尽量的总结一些理论和思考，一起探讨！


<!-- more -->

# REST 的诞生

## Web 技术发展

Web开发技术的发展可以粗略划分成以下几个阶段：

1. 静态内容阶段：在这个最初的阶段，使用Web的主要是一些研究机构。Web由大量的静态HTML文档组成，其中大多是一些学术论文。Web服务器可以被看作是支持超文本的共享文件服务器。
    - 可以想象下当时的HTTP请求只有“GET”，且MIME为“HTML或TEXT”

1. CGI程序阶段：在这个阶段，Web服务器增加了一些编程API。通过这些API编写的应用程序，可以向客户端提供一些动态变化的内容。Web服务器与应用程序之间的通信，通过CGI（Common Gateway Interface）协议完成，应用程序被称作CGI程序。

1. 脚本语言阶段：在这个阶段，服务器端出现了ASP、PHP、JSP、ColdFusion等支持session的脚本语言技术，浏览器端出现了Java Applet、JavaScript等技术。使用这些技术，可以提供更加丰富的动态内容。

1. 瘦客户端应用阶段：在这个阶段，在服务器端出现了独立于Web服务器的应用服务器。同时出现了Web MVC开发模式，各种Web MVC开发框架逐渐流行，并且占据了统治地位。基于这些框架开发的Web应用，通常都是瘦客户端应用，因为它们是在服务器端生成全部的动态内容。

1. RIA应用阶段：在这个阶段，出现了多种RIA（Rich Internet Application）技术，大幅改善了Web应用的用户体验。应用最为广泛的RIA技术是DHTML+Ajax。Ajax技术支持在不刷新页面的情况下动态更新页面中的局部内容。同时诞生了大量的Web前端DHTML开发库，例如Prototype、Dojo、ExtJS、jQuery/jQuery UI等等，很多开发库都支持单页面应用（Single Page Application）的开发。其他的RIA技术还有Adobe公司的Flex、微软公司的Silverlight、Sun公司的JavaFX（现在为Oracle公司所有）等等。

1. 移动Web应用阶段：在这个阶段，出现了大量面向移动设备的Web应用开发技术。除了Android、iOS、Windows Phone等操作系统平台原生的开发技术之外，基于HTML5的开发技术也变得非常流行。


## REST 的诞生

从上述Web开发技术的发展过程看，Web从最初其设计者所构思的主要支持静态文档的阶段，逐渐变得越来越动态化。Web应用的交互模式，变得越来越复杂：从静态文档发展到以内容为主的门户网站、电子商务网站、搜索引擎、社交网站，再到以娱乐为主的大型多人在线游戏、手机游戏。


Web发展到了1995年，在CGI、ASP等技术出现之后，沿用了多年、主要面向静态文档的HTTP/1.0协议已经无法满足Web应用的开发需求，因此需要设计新版本的HTTP协议。在HTTP/1.0协议专家组之中，有一位年轻人脱颖而出，显示出了不凡的洞察力，后来他成为了HTTP/1.1协议专家组的负责人。这位年轻人就是Apache HTTP服务器的核心开发者Roy Fielding，他还是Apache软件基金会的合作创始人。

所以，REST 并不是在互联网诞生之初就有的，它是在HTTP/1.1协议中才出现的，由Roy Thomas Fielding这位大神对Web技术做了深入的总结和分析，提出的一套网络软件的架构风格理论框架，当时Fielding为这种架构风格取了一个轻松愉快的名字：“REST” ———— Representational State Transfer（表述性状态转移）

+ Roy Thomas Fielding 关于REST的论文
    - [https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf](https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf)

---

# REST 详解

## REST 架构风格

    问题：REST 究竟是什么？是一种新的技术、一种新的架构、还是一种新的规范？


首先，REST是Web自身的`架构风格`，也是世界上最成功的分布式应用架构风格。它是为运行在互联网环境的分布式超媒体系统量身定制的。

+ REST是一种架构风格！
+ REST是一种架构风格！
+ REST是一种架构风格！

所以，就会存在实际开发工作中即使没有正确的理解和应用REST，但也能顺利的完成开发工作。也正因为如此，给开发工作中推广正确实践和统一风格带来不小的困难。因为大多数程序员总是在寻找最快解决问题，最快完成需求的方式，怎么简单怎么来。


## 解读 REST

    REST ———— Representational State Transfer (表现层状态转化)


从“Representational State Transfer”这个定义去理解REST架构风格原则。


### 1. 资源（Resources）

REST 的名称“表现层状态转化”中，省略了主语。“表现层”其实指的是“资源（Resources）”的“表现层”


资源是一种看待服务器的方式，此处指的“资源”是一个抽象的概念，它不仅仅指服务器端真实存在的文件、数据库表，而是指任何可被名词表述的东西。所以在定义“资源”时可以要多抽象就多抽象。


对于客户端，可以将服务器端看作是由很多离散的资源组成。服务端可以用URI（统一资源定位符）指向资源，每种资源都对应一个特定的URI。要向获取这个资源，访问它的URI就可以了，因此URI就成了每一个资源的地址或独一无二的识别符。


所谓“上网”，就是与互联网上一系列的“资源”互动，调用它的URI。

### 2. 表现层（Representation）

"资源"是一种信息实体，它可以有多在的表现形式。我们把“资源”具体呈现出来的形式，叫做它的“表现层（Representation）”


比如，文本信息可以用txt格式表现，也可以用HTML格式 、XML格式、JSON格式表现，甚至可以用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。


URI只代表资源的实体，不代表它的表现形式。资源的具体表现形式，应该在HTTP请求的的头部信息中用Accept和Content-Type字段指明，这两个字段才是对“表现层”的描述。  

+ 详见HTTP MIME明细


### 3. 状态转化（State Transfer）

HTTP协议是一个无状态的协议，这意味着所有资源的状态都保存在服务器端。因为客户端想要操作服务器，必须通过某种手段，让服务器端资源发生“状态转化”。而这种转化是建立在表现层之上的，所以就是“表现层状态转化”。


客户端用到的手段，只能是HTTP协议。具体对应HTTP协议中的HTTP Method：GET、POST、PUT、PATCH、DELETE、HEAD、OPTIONS。每一种HTTP Method代表资源状态转化的一种`约定的`方式。



**HTTP 动词**

对于资源的具体操作类型，有HTTP动词表示。

常用的HTTP动词如下：

```
- GET : 从服务器取出资源（一个或多个）

- POST : 在服务器新建一个资源，并返回创建后的完整资源到客户端

- PUT : 在服务器以覆盖形式，全量更新资源，并返回更新后的完整资源到客户端

- PATCH : 在服务器端更新资源，但只更新指定的内容

- DELETE : 在服务器端删除资源
```

其中，GET、PUT、PATCH、DELETE都应该是幂等的。

另外，HEAD、OPTIONS对于团队开发来说基本不用。

```
- HEAD : 获取资源的元数据

- OPTIONS : 获取信息，关于资源的哪些属性是客户端可以改变的
```

### 4. 综述

综合上面的解读，总结一下什么是REST架构风格：

(1) 服务器端的任何信息和数据都要被抽象资源化；  
(2) 资源用URI进行表述，每一个URI代表一种资源；  
(3) 客户端与服务器之间，基于某种表现层形式，互相传递资源；  
(4) 客户端与服务器之间，基于HTTP Method对服务器端资源的操作，实现“表现层状态转化”；  


## REST 与 RESTful

定义：  

+ 如果一个架构符合REST原则，就称它为RESTful架构
+ 如果HTTP API的设计符合REST原则，那么可称它为RESTful API

所以，回到开篇讲的大多数人对于REST还是处于“盲人摸象”的阶段，回想下自己和身边的同事，在工作中经常交流到的REST API或RESTful API，其实只能算个HTTP API吧？


---

# REST 风格优点

架构风格不是非此即彼的是非题，在实际开发中可以自主的选择是否应用REST风格。那么，如果应用REST风格会带来哪些优势呢？

1. 从面向操作编程，转变为面向资源编程。更面向对象，架构更清晰、松耦合。
    - 我们应该确定的认为系统由“资源+对资源的操作”组成，而不是由“操作”组成
    - 面向操作编程会导致API膨胀，功能重复度高。
1. 统一URI命名风格，URI具备很强的可读性，具备自解释的能力。服务器资源层次目录清晰。
1. 状态无关。确保系统横向扩展的能力。
1. 超文本驱动。确保系统演化的能力。


---


# REST实践体会

## 1. URI命名难度变大

在没有要求URI必须用资源名词来组成URI时，URI的命名从来不是什么难事，常见的命名风格有：

+ 动词+名词
    - /deposit/getUsers: 获取某个项目保证金用户列表
    - /orders/submitAudit: 订单提交审核 
    - /cart/add: 商品加购物车
+ URI全局唯一即可
    - /finance/budget/getPurchaseplanNextAuditOrgList：我有点小无语...  

> 为什么会这样：

我们平时搞系统是这样的：

1. 有新建用户功能
1. 新建用户需要一个URL
1. 往这个URL发送的数据要定义好
1. 开始写后端和前端

这是以操作为第一位的设计方法，首先确认了一个操作，然后围绕这个操作把周边需要的东西建设好，这种方式当然可以架构出一个系统，甚至是一个好系统，但是偶尔会有些问题：

1. 操作之间是会有关联，你的设计容易变成“第2个操作要求第1个操作进行过”，这种关系多起来你的系统就乱了
1. 你的URL设计会缺乏一致性
1. 操作通常被认为是有副作用（Side Effect）的，所以很少有人基于操作去设计缓存之类的东西  
  
  
> 该怎么应对？

确实，REST是高度抽象的理论和风格，在实际开发中会面对各种复杂的功能和场景，导致很难完全的应用REST风格。当我们在争论REST风格到底如何设计才是正宗时，发现心中的困惑不仅没有降低，反而增加了。

我的想法：仍以真正的系统需求为出发点，使用REST风格让系统的架构更清晰，让系统的开发协作更高效。部分不适合REST的场景应该灵活变通。


回到URI的命名：

1. 坚持URI仍以资源为导向，清晰的表述服务器端资源目录
1. 保障URI资源层次清晰的情况下，只允许在URI最末一级添加动词，例如：/market/orders/1/audit
1. 如果某些动作是HTTP动词表示不了的，考虑把动作抽象成一种资源

比如：网上汇款，从账户1向账户2汇款100元，错误的URI

```
POST /accounts/1/transfer/500/to/2
```

正确的写法是把动词transfer改成名词transaction

```
POST /transaction?from=1&to=2&amount=100
```


## 2. 用不用HTTP PATCH

PATCH 作为HTTP的Method之一，其实它是2010年3月份才正式成为HTTP Method的，详见：[RFC 5789](http://tools.ietf.org/html/rfc5789)

也正因为PATCH出现的晚, 所以并不是所有Web容器都支持，反而目前实现了PATCH方法的Web容器很少

几个常见Web容器实现PATCH方法的情况，供参考：

1. Apache HttpComponents HttpClient version 4.2 or later 支持了 PATCH
2. 目前 JDK7 的 HttpURLConnection 未实现 PATCH
3. TOMCAT 7 也不行
4. PlayFramework 2 也不支持
5. Spring 3.2 开始支持 PATCH 方法，但要选对部署的容器
6. JBoss Netty 支持 PATCH，可见： [http://docs.jboss.org/netty/3.2/api/org/jboss/netty/handler/codec/http/class-use/HttpMethod.html](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/handler/codec/http/class-use/HttpMethod.html)


---



