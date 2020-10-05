---
title: MySQL索引合并
tags: [MySQL,数据库,索引]
date: 2018-05-06 17:55:18
categories: 技术
---

## 历史背景

MySQL 5.0版本之前，一个表一次只能选择并使用一个索引。

MySQL 5.1版本开始，引入了Index Merge Optimization技术，使得MySQL支持一个表一次查询同时使用多个索引。

官方文档：[MySQL Index Merge Optimization](https://dev.mysql.com/doc/refman/5.5/en/index-merge-optimization.html) 

Index Merge Optimization支持三种合并算法

+ The Index Merge Intersection Access Algorithm
    - 对应SQL 中的 AND 场景
+ The Index Merge Union Access Algorithm
    - 对应SQL中的 OR 场景（where条件是等值判断）
+ The Index Merge Sort-Union Access Algorithm
    - 对应SQL中的 OR 场景（where条件是范围查询）
    
<!--more-->


> 注：索引合并(Index Merge)的使用取决于optimizer\_switch系统变量的index\_merge，index\_merge\_intersection，index\_merge\_union和index\_merge\_sort\_union标志的值。默认情况下，所有这些标志都打开。 要仅启用特定算法，请将index\_merge设置为关闭，并仅启用其他应允许的其他算法。

##关于"Index Merge Intersection Access Algorithm"的疑问

> 针对 MySQL Index Merge Optimization Intersection Algorithm

AND 场景的 index merge optimization为什么会比使用单个索引来的高效？


设想：

+ 使用单个索引的场景
    1. 选中选择性高的索引先获得一份数据
    1. 在再mysql服务器端用using where的方式，按第二条件进行过滤，得到最终满足所有条件的数据行。
    
+ 同时使用表内多个索引的场景
    1. 按每个索引，在索引树里拿只满足本索引条件的行数据
    1. 将两份行数据，放一块进行交集运算。
    1. 从索引的次数、磁盘IO、内存交接运算来看，事情没变少、反而变多了。

---

## 自我初版解释

#### 合理的解释

样例SQL
```
select * from table_sample where column_1 = A AND column_2 = B;
```

1. 前提条件，SQL中不能有范围查询，如果存在范围查询，数据库优化器默认使用单索引方式，不用index merge optimization
1. SQL的`WHERE从句中的所有条件字段都有对应的索引`，否则问题就来了，肯定会在内存中有次using_where的。
1. 单表多Index并行检索时，拿到的是数据行地址，以上述SQL为例，即拿到了两份行数据地址：Index Column_1的行数据地址集，Index Column_2的行数据地址集
1. 再在内存中完成两份行数据地址集的交集运算（只需要比地址）
1. 此时，再决定是否回表拿更多的数据。
    - 如果字段中有primary key，就不用回表啦！

+ 如上的执行步骤，就会比较合理。有效率上的优势。

#### 【更进一步】 explain 显示type 为 index_merge时，到底要不要引起关注？

`【需要引起注意】`

拿着SQL琢磨下，是否还有优化的空间，例如：采用组合索引；强制走单索引（需要对比测试看效果，还要看业务数据场景和增长趋势）；

---

注：

1. 当索引本身信息可以覆盖select的字段时（或是select count(*)）,效率会很高，因为内存索引里已经能提供返回的数据了，不用回表。
1. 当索引本身信息不能覆盖select的字段时，就要回表查行数据了，性能差别很大。







