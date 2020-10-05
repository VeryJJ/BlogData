---
title: 我眼中的Java工作流
tags: [Java, bpmn, 工作流]
categories: 技术
---

# 工作流

工作流是一组功能套件（开源版本/自研都可），其提供了业务流程定义的能力，并提供了将业务流程应用到系统中去的能力。

基于工作流，可以将系统中复杂业务流程场景的设计实现简单化，并极大的提高了系统功能的灵活性和创造性。


### 工作流优点：

- 基于配置，修改灵活
- 提供了产品、业务、研发间对于业务流程的统一建模语言
- 内置工作流引擎，规则引擎，通过配置组合的方式即可完成复杂业务场景的定义和实现
- 极大的降低开发量



### 工作流套件
- 【 BPMN 2.0 】

【很重要】业界主流工作流都是遵守BPMN2.0标准的。

**BPMN 2.0 是什么？**

BPMN - Business Process Modeling Notation ： 业务流程建模标记语言

BPMN 是业界业务流程建模语言的标准。工作流通过BPMN语言来定义业务流程。

通过统一的业务流程建模语言，使得产品、业务、研发对于业务流程的交流更高效、精确！

BPMN 2.0 更详细介绍：[BPMN 2.0](https://blog.csdn.net/a123demi/article/details/50674124)


- 【 Flow Designer 】

工作流解决方案提供的工作流设计器，常见的有：独立部署的设计器UI；设计器Server；直接XML编写；

无论哪种方式的Flow Designer，最终会生成一个XML配置文件，上面记录着关于工作流的所有配置内容。


- 【 Intergration Solution 】

上述两者解决了工作流定义的需求，接下来就是工作流集成到系统中去的需求。比较常见的是将工作流组件包集成到系统工程中，进行二次开发。

- 【 Advanced 】

    - 可扩展
    - 监控


从open-open上看，java平台的共有50个工作流开源框架。建议选择Activiti。
原因：
1、Activiti是当今最流行的工作流开源框架；
2、它在jBPM4的基础发展过来，而jBPM只要搞过工作流的人基本都会；
3、使用java语言，我们公司会java的人最多；
4、包含了引擎核心PVM流程虚拟机，不需要单独引入一个规则引擎框架（如Drools）；
5、社区活跃，容易解决问题，容易功能扩展；
6、支持oracle、mysql、sql server；
7、公司吉林的系统用的就是它，有先例；
8、是开源的，并且是免费的；
9、与spring结合很好；
http://www.activiti.org/


# 常见Java工作流

| | Activiti | Drools jBPM | jBPM | Shark | OSWorkflow |
|---|---|---|---|---|---|
| 出品方 | 开源：www.activiti.org | JBoss | JBoss | Enhydra | opensymphony
| 开源协议 | | | Apache |
| 使用难度 | 轻量级 | | | 重量级，功能强大 | 轻量级
| 建模语言 | BPMN | BPMN | BPMN | XPDL | 
| 自带Designer | 
| 持久化 | Mybatis | | hibernate |
| | | | |
| | | | |
| 缺陷 | | | | | 对于分支、聚合、子流程的支持度很低
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
