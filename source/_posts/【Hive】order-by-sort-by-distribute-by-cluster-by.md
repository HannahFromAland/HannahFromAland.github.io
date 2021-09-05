---
title: '【Hive】order by, sort by, distribute by, cluster by'
date: 2021-03-17 09:28:14
tags:
- Hive
- BigData
- SQL
category: 
- Big Data Processing
---

对Hive中四种排序方式进行整理和区分。

<!-- more -->

### Order by

与传统SQL中的Order by效果一致，即对查询结果进行全局排序。由于Hive通过MapReduce实现数据查询，因此全局排序要求所有数据必须集中在**一个Reducer**上进行处理。

- 除此之外，如果指定了`hive.mapred.mode=strict`（默认是`nonstrict`）时，就必须指定limit来限制输出条数。

示例如下，表中列为`id`  ，`group`， `grade`，共20条数据

直接进行order by操作：

```sql
select user_id,`group`,grade
    > from test_log
    > order by grade;
```

### ![](https://i.loli.net/2021/03/17/Qy13xmCwWYUViGA.png)

- 数据结果按照grade正序排列。

### Sort by

Sort by的数据是局部有序的，其排序作用发挥在数据分别进入各个Reducer之前，因此只会对每个Reducer上的数据进行局部排序。

- 使用sort by时，可以通过指定执行的reducer个数（`set mapred.reduce.tasks=n`），对输出的数据再执行归并排序，即可得到全部结果。
- sort by排序顺序将取决于列类型。如果该列是数字类型，那么排序顺序也是按数字顺序排列的。如果该列是字符串类型，那么排序顺序将是字典顺序。
- sort by不保证相同参数被分配到同一reducer（即若某个参数的数据量较大，可能会被分配到不同的reducer，且只能保证各个reducer的结果是有序的）

```sql
set mapreduce.job.reduces=3;
set hive.cli.print.header=true;
-- 将reducer个数设为3，观察各个分区上的排序结果
select *
    > from test_log
    > sort by grade;
```

![](https://i.loli.net/2021/03/17/nFIL5ZMCdYpyl8O.png)

- 在三个分区中的数据记录均按照升序排列，但对于相同参数可能存在重合的排序区间

### Distribute by

Distribute by控制map端数据如何拆分和分发给reduce端。Distribute by指定的参数相同的数据将被分配到同一个Reducer上。

- 分发Reducer采用的是Hash算法，因此相同参数的记录只会被分发到一个reducer，但同一reducer上可能会出现多个参数的记录。

- 在各个reducer中的数据不保证有序排列

```sql
select *
    > from test_log
    > distribute by grade;
```

![](https://i.loli.net/2021/03/17/9SoRf1MQYuWhJyc.png)

- grade为9的数据全部进入reducer1，5和8的数据均在reducer2，而1、4、7分布在reducer3上，但是各个reducer内并没有按照grade大小进行排序

- 将distribute by和sort by连用，即可将指定字段的记录分发到不同的reduce中，且在每个reduce内部会根据指定字段值来排序

```sql
select *
    > from test_log
    > distribute by grade
    > sort by grade;
```

![](https://i.loli.net/2021/03/17/kvUAbh4S7MHNKBo.png)

### Cluster by

CLUSTER BY与DISTRIBUTE BY和SORT BY连用的执行结果相同。即当distribute by的字段与sort by字段相同时，使用cluster by与其等价。

Notice：cluster by 不能指定升降序排列，默认升序。

```sql
hive> select *
    > from test_log
    > cluster by grade;
```

![](https://i.loli.net/2021/03/17/aNI3vc4FeKVAjpE.png)



#### Reference

[Hive的Order by、Sort by、Distribute by和Cluster by的区别](https://blog.csdn.net/u012369535/article/details/89639434?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

[Hive : SORT BY vs ORDER BY vs DISTRIBUTE BY vs CLUSTER BY](https://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247487481&idx=1&sn=1f707a3cada10e06f6d4c4f4293e752b&source=41#wechat_redirect)

鸣谢：CYS Macbook