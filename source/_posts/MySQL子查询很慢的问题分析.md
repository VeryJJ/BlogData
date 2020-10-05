---
title: MySQL子查询很慢的问题分析
tags: [SQL, 数据库]
date: 2018-05-04 15:29:46
categories: 技术
---

---

## 慢查询案例

```sql
DELETE FROM settlement_invoice_attachment g1 WHERE demand_id in (SELECT id FROM settlement_invoice_demand g2 WHERE statement_id = 1802065000000074956)
```

乍眼一看，上述SQL如此简单，且demand_id和statement_id字段都是建了索引，即使是Review也会认为是OK没问题的。

然而，实际情况却是个慢查询，情况如下：

**explain明细**

![image](MySQL子查询很慢的问题分析/img-1.png)

<!-- more -->

settlement_invoice_attachment是全表查

注：rows 2689 是因为用的测试环境，真线环境数据是几十万级别

---

#### 子查询 原理分析(上述SQL子查询为什么这么慢)

#####  `经验之谈`
+ 当看到SQL执行计划中select_type字段出现“DEPENDENT SUBQUERY”的时候，要打起精神了！着重分析下潜在风险！

#####  基础知识：Dependent SubQuery意味着什么？
+ 官方含义为：
    1. SUBQUERY: 子查询中的第一个SELECT；
    1. DEPENDENT SUBQUERY: 子查询中的第一个SELECT， 取决于外面的查询。
  
换句话说，就是`子查询的g2查询执行方式依赖于外层g1的查询结果`
什么意思呢？它以为着两步走：

+ 第一步：【先执行外部SQL查询】MySQL根据"DELETE FROM settlement_invoice_attachment g1 WHERE" 得到一个大结果集t1，其数据量就是全表所有行了，假设是85万行。
+ 第二步：【后执行内部SQL子查询】第一步的大结果集t1中的每一条记录，都将与子查询SQL组成新的查询语句：SELECT id FROM settlement_invoice_demand g2 WHERE statement_id = 1802065000000074956 AND id = %t1.demand_id%。等于说，子查询要执行85万次......即使这两部查询都用到了索引，也是巨慢的。

##### 优化策略

+ 改写SQL为JOIN的方式

```sql
DELETE ah FROM settlement_invoice_attachment ah INNER JOIN settlement_invoice_demand de 
ON ah.demand_id = de.id WHERE de.statement_id = 1802065000000074956;
```
+ 拆成独立SQL多次执行

---

#### 平时怎么识别？
+ 看子查询出现的位置
    - 若子查询出现在WHERE从句中，而且是出现在IN（）中，则需要引起注意，用Explain瞧瞧（并不是子查询放IN（）里就一定是全表扫，本案例用，将DELETE改成SELECT就不是DEPENDENT SUBQUERY）

#### 数据库原理
1. MySQL处理子查询时，会(优化)改写子查询，但优化的不是很友好，一直受业界批评比较多
    - 有时候优化的挺糟糕的，特别是WHERE从句中的IN（）子查询
  
1. MySQL 子查询的弱点
    - mysql 在处理子查询时，会改写子查询。通常情况下，我们希望由内到外，先完成子查询的结果，然后再用子查询来驱动外查询的表，完成查询。

例如：select * from test where tid in(select fk_tid from sub_test where gid=10)
通常我们会感性地认为该 sql 的执行顺序是：

1、sub_test 表中根据 gid 取得 fk_tid(2,3,4,5,6)记录。 
2、然后再到 test 中，带入 tid=2,3,4,5,6，取得查询数据。

但是实际mysql的处理方式为：
select * from test where exists (select * from sub_test where gid=10 and sub_test.fk_tid=test.tid)  
mysql 将会扫描 test 中所有数据，每条数据都将会传到子查询中与 sub_test 关联，子查询不会先被执行，所以如果 test 表很大的话，那么性能上将会出现问题。












