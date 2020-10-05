---
title: '[翻译] REST API必须是超文本驱动的'
tags: [REST]
date: 2018-05-20 14:53:12
categories: 技术
---

原文地址：[Roy T. Fielding: REST APIs must be hypertext-driven
](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)

> I am getting frustrated by the number of people calling any HTTP-based interface a REST API. Today’s example is the SocialSite REST API. That is RPC. It screams RPC. There is so much coupling on display that it should be given an X rating.

我是越来越失望了，许多人把任何基于HTTP的接口叫做REST API，眼前的例子就是SocialSite REST API。那是RPC，实实在在的RPC。它与显示如此耦合，再差也莫过于此

<!-- more -->

> What needs to be done to make the REST architectural style clear on the notion that hypertext is a constraint? In other words, if the engine of application state (and hence the API) is not being driven by hypertext, then it cannot be RESTful and cannot be a REST API. Period. Is there some broken manual somewhere that needs to be fixed?

基于超文本概念，如何才能确保清晰的REST架构风格呢？这样来说吧，如果应用程序状态引擎（即API）不是由超文本驱动的，那就不是RESTful也不是REST的API。就这么简单。某些REST方面的破手册是否该修正一下呢？

> API designers, please note the following rules before calling your creation a REST API:

API的设计者们，把你们的那些东西叫做REST API前请注意以下的规则：

> A REST API should not be dependent on any single communication protocol, though its successful mapping to a given protocol may be dependent on the availability of metadata, choice of methods, etc. In general, any protocol element that uses a URI for identification must allow any URI scheme to be used for the sake of that identification. [Failure here implies that identification is not separated from interaction.]

REST API不应依赖于任何特定的通讯协议，在采用某个具体协议时可能受限于元数据的有效性、方法的选择等。通常，协议元素使用URI作标识时，对该标识必须允许运用任何URI方案。[ 不符合这一点意味着标识与交互没有分离 ]

> A REST API should not contain any changes to the communication protocols aside from filling-out or fixing the details of underspecified bits of standard protocols, such as HTTP’s PATCH method or Link header field. Workarounds for broken implementations (such as those browsers stupid enough to believe that HTML defines HTTP’s method set) should be defined separately, or at least in appendices, with an expectation that the workaround will eventually be obsolete. [Failure here implies that the resource interfaces are object-specific, not generic.]

REST API不应修改通讯协议中预留出来作为补充或修正标准协议用途的资源，例如HTTP的PATCH方法和Link head域。违背了这一原则的方案应当单独定义，或者至少在附录中标注出来这样的方案最终会废弃掉。[ 不符合这一点意味着资源接口是对象相关的，不通用 ]

> A REST API should spend almost all of its descriptive effort in defining the media type(s) used for representing resources and driving application state, or in defining extended relation names and/or hypertext-enabled mark-up for existing standard media types. Any effort spent describing what methods to use on what URIs of interest should be entirely defined within the scope of the processing rules for a media type (and, in most cases, already defined by existing media types). [Failure here implies that out-of-band information is driving interaction instead of hypertext.]

REST API应当将绝大部分精力放在媒体类型的定义上，或者是扩展关系名称的定义、已有超文本标记中的标准媒体类型等方面，以实现资源的表述、操作应用程序状态。任何类似于对某某URI应当使用什么样的方法等工作，都应当完全定义在特定媒体类型的处理规则范围中（绝大部分情况下已有媒体类型都已经定义好了这些规则）。[ 不符合这一点意味着交互是由其它信息驱动，而不是超文本 ]

> A REST API must not define fixed resource names or hierarchies (an obvious coupling of client and server). Servers must have the freedom to control their own namespace. Instead, allow servers to instruct clients on how to construct appropriate URIs, such as is done in HTML forms and URI templates, by defining those instructions within media types and link relations. [Failure here implies that clients are assuming a resource structure due to out-of band information, such as a domain-specific standard, which is the data-oriented equivalent to RPC's functional coupling].

REST API决不能定义固定的资源名称或者层次关系（这是明显的客户端、服务器端耦合），服务器必须可以自由控制自己的名称空间。应当像HTML forms和URI模板一样，通过媒体类型和链接关系指示客户端如何构造正确的URI。[ 不符合这一点意味着客户端在通过其它信息（例如领域相关标准）猜测资源结构，这是数据导向，类似于RPC的函数耦合 ]

> A REST API should never have “typed” resources that are significant to the client. Specification authors may use resource types for describing server implementation behind the interface, but those types must be irrelevant and invisible to the client. The only types that are significant to a client are the current representation’s media type and standardized relation names. [ditto]

REST API决不能使用对客户端有重要意义的类型化资源。规范的作者可能使用资源类型描述接口背后的服务器端实现，但这些类型必须与客户端无关，对客户端不可见。对客户端唯一有意义的类型是当前的表述性媒体类型和标准的关系名称。[ 同上 ]

> A REST API should be entered with no prior knowledge beyond the initial URI (bookmark) and set of standardized media types that are appropriate for the intended audience (i.e., expected to be understood by any client that might use the API). From that point on, all application state transitions must be driven by client selection of server-provided choices that are present in the received representations or implied by the user’s manipulation of those representations. The transitions may be determined (or limited by) the client’s knowledge of media types and resource communication mechanisms, both of which may be improved on-the-fly (e.g., code-on-demand). [Failure here implies that out-of-band information is driving interaction instead of hypertext.]

使用REST API应该只需要知道初始URI（书签）和一系列针对目标用户的标准媒体类型（任何客户端都了解用来操作该媒体类型的API）。这样所有的应用程序状态转换都通过这样的方式进行：服务器在返回的表述性消息中提供选项，由客户端进行选择，或者是伴随着用户对表述性内容的操作而进行。状态转换由客户端对媒体类型的了解程度和资源通讯机制决定，或者受限于这些因素，这些问题都可以根据实际情况得以改善的（例如使用javascript这种code-on-demand技术）。[ 不符合这一点意味着交互是由其它信息驱动，而不是超文本 ]

> There are probably other rules that I am forgetting, but the above are the rules related to the hypertext constraint that are most often violated within so-called REST APIs. Please try to adhere to them or choose some other buzzword for your API.

也许还有其它一些规则我一时想不起来了，但在那些所谓的REST API中通常都违背了上面这些超文本约束相关的规则，请纠正这些错误或者改用其它称谓吧