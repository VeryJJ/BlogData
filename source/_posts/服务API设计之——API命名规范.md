---
title: 服务API设计之——API命名规范
tags: [API, API命名]
date: 2018-09-30 17:46:20
categories: 技术
---



# API命名规范

## 命名风格

------

### 面向资源

> 同RESTful命名风格

在大型系统中，常以"业务领域"视角进行模块划分，以达到业务"高内聚低耦合"的效果。

"业务领域"必有"数据对象"沉淀，`从宏观抽象的角度看，"数据对象"可统称为"资源"`，"业务领域"就是业务相近的"资源"的集合。



<!--more-->



`"资源"一定是业务抽象后的对象`：

1. 可以是具体的数据对象：
   - 商品
   - 订单
   - 合同
   - 发票
   - 采购计划
   - etc
2. 可以是抽象的对象概念：
   - 租户
   - 用户
   - 支付
   - 文件
   - 需求
   - etc

`"业务领域"与"业务领域"之间的依赖，可理解为是对"资源"操作(读、写、通知)的依赖。`

`所以，API作为"业务领域"间沟通的手段，其应该(Should)以面向资源角度进行命名。`

注：子资源，需要逐级索引命名，例如：修改-订单-商品：updateOrderItem。

------

### 单一视角

- 参见[单一视角原则]()

------

### 动宾风格

API应该(Should)以`"动宾短语"风格命名`。

例如：

```
xxx.xxx.xxx.OrderService            // 上下文已涵盖Order语义

Response<T> save(...)   

Response<T> updateItem(Long orderId, List<T> items)

```

```
xxx.xxx.xxx.WCService               // 上下文未涵盖Order语义

Response<T> saveOrder(...)   

Response<Boolean> removeOrder(Long orderId)   

Response<T> updateOrderItem(Long orderId, List<T> items) // 逐级索引子资源

```

------

### 统一术语

API命名统一"动词"术语、"名词"术语。优点是能风格一致，经验复用。

详见[政采云API术语参考]()

注：统一术语的节奏，参考研发级术语规范逐步执行：业务内统一、业务领域内统一、平台统一。

##### 错误实践-1："商品"命名不统一

```
业务1：商品 -> item ✔️
业务2：商品 -> items
业务3: 商品 -> product
业务4：商品 -> goods
```

##### 错误实践-2："特性"命名不统一

```
业务1：特性 -> feature ✔️
业务2: 特性 -> character
业务3：特性 -> rule
```

##### 错误实践-3："金额"命名不统一

```
业务1：金额 -> amount ✔️
业务2: 金额 -> money
业务3：金额 -> sum
```

##### 错误实践-4："校验"命名不统一

```
业务1：校验 -> verify
业务2: 校验 -> check ✔️
业务3：校验 -> test
```

##### 错误实践-5："分页"命名不统一

```
业务1：分页 -> page
业务2: 分页 -> paging✔️
业务3：分页 -> list 
```

##### 错误实践-6："创建"命名不统一

```
业务1：创建 -> save✔️
业务2: 创建 -> create
业务3：创建 -> insert 
```

##### 错误实践-7："删除"命名不统一

```
业务1：删除 -> delete
业务2: 删除 -> remove✔️
业务3：删除 -> disable 
业务3：删除 -> cancel 
```

##### 错误实践-8："检索"命名不统一

```
业务1：搜索 -> query✔️
业务2: 搜索 -> search
业务3：搜索 -> list 
```

------

## 常见API命名参考

> 假设：未按资源划分Service(上下文未界定资源域)的情况

> "XXX"指某一种资源，"xxx"指"XXX"下的子资源

### 分页查询

- 正确实践

```
Response<Page<T>> pagingXXX(QueryDTO q)    //用对象包装查询条件
```

- 错误实践    

```
Response<Page<T>> pagingXXX(String name, String code, Long orgId, Long creatorId, Integer pageNo, Integer PageSize) 
```

以上错误实践缺点：  
1、对于调用方来说，无论以什么条件查询，都需要逐个条件传参  
2、API对扩展不友好，一旦想增加查询条件，API就不兼容。

### 列表查询

- 正确实践

```
Response<List<T>> listXXX(...)
```

### 获取单个详情

- 正确实践

```
Response<T> getXXX(Long id) 

类同条件，用重载

Response<T> getXXX(String code) 
```

- 错误实践

```
Response<T> getXXXById(Long id) 

Response<T> getXXXByCode(String code) 

```

说明：

1. API契约应该由"API名 + 入参"共同组成，而不是只靠"API名"说明一切。
2. API方法支持获取单个详情的方式，可以通过入参字段名自解释。无需再用"By***"来额外标注。
3. 不带"By***"声明的方法语义上更具有扩展性。

### 创建

- 正确实践

```
Response<T> saveXXX(...)      //参照《阿里巴巴Java编码规范》
```

### 删除

- 正确实践

```
Response<T> removeXXX(...)      //参照《阿里巴巴Java编码规范》
```

### 更新

- 正确实践

```
Response<T> updateXXX(...)      //参照《阿里巴巴Java编码规范》

Response<T> updateXXXxxx(...)   //更新主资源下的子资源
```

### 提审

- 正确实践

```
Response<T> submitXXX(...)  
```

### 审核

- 正确实践

```
Response<T> auditXXX(...)
```

### 退回（退回到流程中的某一步）

- 正确实践

```
Response<T> returnXXX(...)
```

### 撤销（退回到流程的第一步）

- 正确实践

```
Response<T> cancelXXX(...)
```



