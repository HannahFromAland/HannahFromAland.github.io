---
title: 【SQL】 Predicate pushdown
date: 2021-03-23 09:03:11
tags:
- SQL
- Hive
category:  Data Analysis
---

## 什么是谓词下推（ Predicate pushdown）

- 对SQL语句来说，就是将外层查询块中的where子句过滤条件移入所包含的较低层查询块（即下推至离数据源更近的地方）；
- 对于Hive或Spark QL来说，在不影响结果的情况下提前执行过滤语句，可以减少Map端输出的数据量，降低集群的传输资源同时减少Reduce端的工作量，最终提升任务的性能

<!-- more -->

### SQL中join条件谓词下推

#### 几个概念

- Preserved Row table（保留表）

在`outer join`中需要返回所有数据的表叫做保留表，也就是说在`left outer join`中，左表需要返回所有数据，则左表是保留表；`right outer join中`右表则是保留表；在`full outer join`中左表和右表都要返回所有数据，则左右表都是保留表。

- Null Supplying table（空表）

在`outer join`中对于没有匹配到的行需要用`null`来填充的表称为`Null Supplying table`。在`left outer join`中，左表的数据全返回，对于左表在右表中无法匹配的数据的相应列用`null`表示，则此时右表是`Null Supplying table`，相应的如果是`right outer join`的话，左表是`Null Supplying table`。但是在`full outer join`中左表和右表都是`Null Supplying table`，因为左表和右表都会用`null`来填充无法匹配的数据。

- During Join predicate（Join中的谓词）

`Join`中的谓词是指 `Join On`语句中的谓词。如：`R1 join R2 on R1.x = 5 the predicate R1.x = 5`是`Join`中的谓词

- After Join predicate（Join之后的谓词）

`where`语句中的谓词称之为`Join`之后的谓词

#### InnerJoin

`inner join`中及`join` 后的谓词均可以下推，二者是等价的（即内连接要求左右表均要满足条件，无空表）

#### OuterJoin（包含left和right两种情形）

以`left outer join`为例，执行时左表为保留表，因此`Join`中左表的谓词不能进行下推，`Join`中右表谓词可以下推；`Join`后谓词均可以下推

#### FullJoin

`full join`中左右表均属于`Null Supplying tables`，因此在`Join`中的谓词均不能下推；同`outer join`，`Join`后的谓词均可以下推

### 谓词下推的意义

- 大多数表存储行为都是列存，列之间独立存储，扫描过滤只需要扫描join列数据（而不是所有列），如果某一列被过滤掉了，其他对应的同一行的列就不需要扫描了，减少了IO扫描次数
- 减少了数据从存储层通过socket发送到计算层的开销（正常情况下执行会将所有数据从存储进程加载到计算进程，再进行过滤计算；谓词下推后存储进程将直接过滤无效数据，减少后续一系列开销，提升性能



**Reference**

[predicate pushdown](https://csy99.github.io/Blog/2020/05/10/predicate-pushdown/)

[sql优化中join 条件谓词下推](https://zhuanlan.zhihu.com/p/129026235)

[从一个sql引发的hive谓词下推的全面复盘及源码分析(上)](https://zhuanlan.zhihu.com/p/78266517)