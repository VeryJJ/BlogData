---
title: 服务API设计之——API参数规范
tags: [API, API规范]
date: 2018-09-30 17:32:03
categories: 技术
---

### 【强制】字段名称用小驼峰风格 






### 【强制】Service API返回值必须使用Response包装

- Service API返回值强制要求进行通用包装，例如：Response。
- Response的作用：
  1. 统一方法表示API调用是否成功
  2. API调用失败时，统一格式反馈错误Code，错误Message
  3. 统一的Response易于调用方经验复用，框架集成

- 作为API调用方，其编码诉求很简单：
  1. API调用是否成功；
  2. 调用不成功时，提示文案是什么；
- 调用方几不想：
  1. 不想关心API内部有多牛逼
  2. 不想关心API可能会抛的各种Exception，以及因此不得不做各种异常处理

- 关于当前不统一的Response
  - 【新业务】【强制】使用架构组定义的统一Response：[ZCY Response](http://confluence.cai-inc.com/pages/viewpage.action?pageId=6623775)
  - 目前业务方有自定义Result/Response，风格和作用大同小异。有更好的设计可以自荐给架构组集成，杜绝各自开辟重复的新定义。

<!--more-->



### 【强制】杜绝完全不规范的缩写，避免望文不知义。（国际通用缩写除外）

- 错误实践
  - AbstractClass“缩写”命名成 AbsClass;
  - condition“缩写”命名成 condi；
  - 此类随意缩写严重降低了代码的可阅读性。



### 【强制】禁止使用 Map 作为参数类型

Map<K,V>机制非常灵活，但这样的灵活却是负作用巨大。

1. Map的数据说明是晦涩的，调用方、实现方之间需要具有隐式的契约解释支持哪些Key，每个Key的Value是什么类型。增加了双方的使用复杂度。
2. Map<K,V>不可被校验。加之第1条的使用复杂度，导致使用上非常容易出错。
3. 用Map类型字段做预留扩展性的设计都是不优雅的设计。

注：参数中的调用方自定义数据部分允许使用Map。API提供方不关系、不解析、只透传。





### 【强制】业务对象/查询条件用DTO封装，禁止以入参方式平铺字段。

- 正确实践

分页查询，将查询条件以DTO方式包装。

Dubbo序列化特点：

- Dubbo API的POJO类中，UID不一致：没关系。
- Dubbo API的POJO类中，字段数量不一致：没关系，只要字段名和类型一致，数据能反序列化成功。
- 发送方比接收方的字段多：没关系。
- 发送方比接收方的字段少：没关系。

```
Response<Page<T>> pagingXXX(QueryDTO q) 
```

- 错误实践    

```
Response<Page<T>> pagingXXX(String name, String code, Long orgId, Long creatorId, Integer pageNo, Integer PageSize) 
```

以上错误实践缺点：  
1、对于调用方来说，无论以什么条件查询，都需要逐个条件传参。
2、API对扩展不友好，一旦想增加查询条件，API就不兼容。





### 【推荐】DTO字段设置JSR303 Annotation进行基础校验

- 正确实践

```
public interface ZcyPayFacade {
    Result<Boolean> validTradePay(@NotNull @Valid TradePayPO tradePayPO);
}
```

```
public class TradePayPO implements Serializable {

    @NotBlank
    @Length(max = 15)
    /** 业务交易编号(订单编号) */
    private String businessTradeNo;

    /**
     * 业务渠道：1-订阅，2-CA
     * @see BusinessTypeEnum
     *
     * */
    @NotNull
    @Range(min = 1, max = 2)
    private Integer businessType;

    ......
    
    /** 商户名称(商家) */
    @NotBlank
    @Length(max = 50)
    private String merchantName;

    /** 订单标题（即商品名称），粗略描述用户的支付目的。如“喜士多（浦东店）消费”*/
    @NotBlank
    @Length(max = 256)
    private String orderSubject;

    /** 订单描述（即商品描述），可以对交易或商品进行一个详细地描述，比如填写"购买商品2件共15.00元"*/
    @NotBlank
    @Length(max = 128)
    private String orderBody;

    ......
}

```





### 【推荐】在客户端完成基础字段校验

- 方式1：【推荐】自定义Dubbo Filter实现通用拦截、校验。
- 方式2：【推荐】通过Builder模式构建入参对象。
- 方式3：【不推荐】Dubbo 客户端参数校验，要求consumer方设置validation="true"，[Dubbo 客户端参数校验](http://dubbo.apache.org/zh-cn/docs/user/demos/parameter-validation.html)。缺点：以抛异常方式处理校验失败，需要业务方额外处理Exception。而且，IDE并不会提示consumer方需要处理ConstraintViolationException。
- 方式4：Dubbo方式，local-stub特性。实现较复杂，校验代码通用性低。[Dubbo local-stub](http://dubbo.apache.org/zh-cn/docs/user/demos/local-stub.html)





------

### 注：此规范与《阿里巴巴Java编码规范》互补，同时有效。