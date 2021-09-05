---
title: 【SQL】Salary Table
date: 2021-09-04 19:44:54
tags:
- SQL
- Hive
category:  Data Analysis
---

# Salary Table

记录截至目前遇到的最难的SQL题！:smirk_cat:

![](https://i.loli.net/2021/09/04/tzbJw1A48olkn9d.png)

![](https://i.loli.net/2021/09/04/H1KLqyaV98X32eJ.jpg)



- 大致思路：按照`position` 进行分组后按工资正序进行rank（为了hire尽可能多的员工，优先从工资最少的开始），并返回截止目前该分类内的`salary`总和

- 中间表构造：

  ```sql
  select position, salary, row_number()over(partition by position order by salary) as rnk
  from candidates 
  order by position DESC
  ```

  

- **截止目前该分类内的`salary`总和**：

  - 方法一：实现连接

```sql
select
 a.position,
 a.rnk,
 sum(j.salary) 
 from(select id,position, salary, row_number()over(partition by position order by salary) as rnk  
 from candidates  order by position DESC) as a  
 join (select id,position, salary, row_number()over(partition by position order by salary) as rnk  from candidates  order by position DESC)  as j on 
 a.position = j.position and a.rnk >= j.rnk group by a.position,a.rnk;
```

也可以通过将sum子查询放在select中进行简化：

```sql
select
 a.position,
 a.rnk,
(select sum(j.salary) 
 from (select 
       id
       ,position
       ,salary
       ,row_number()over(partition by position order by salary) as rnk  
       from candidates  
       order by position DESC) as j 
 	where  a.position = j.position and a.rnk >= j.rnk ) as cumulative_salary
 	-- select中直接进行两表连接和聚合查询
 from(select id,position, salary, row_number()over(partition by position order by salary) as rnk  
 from candidates  order by position DESC) as a  

```


  - 方法二： 使用` rows between unbounded preceding and current row`实现到当前行的聚合累加

      - `unbounded preceding` 表示最开始的行记录
      - `current`为截止当前行

```sql
select id,position, salary, row_number()over(partition by position order by salary) as rnk,
sum(salary) over(partition by position order by salary rows between unbounded preceding and current row) as cumulative_salary
from candidates 
order by position DESC;
```

> Notes: 发现`order by`中可以对所有参数进行单独的`ASC/DESC`指令

最终解：

```sql
select junior,senior
from(select b.position, b.rnk as junior, b.cumulative_budget as j_budget,sen.rnk as senior,sen.cumulative_budget as s_budget
from(select position, row_number()over(partition by position order by salary) as rnk ,
sum(salary) over(partition by position order by salary rows between unbounded preceding and current row) as cumulative_budget
from candidates
where position = 'junior' and salary <=50000
order by salary ) b 
     -- 表内为所有小于50000的junior按工资数额正序排列，
     -- 并返回各雇佣数目的累计所需budget
full join 
     -- mysql居然不支持full join...
     -- 可以将left outer join 和right outer join的结果再进行union
(select position,rnk,cumulative_budget
 from (select position, salary,row_number()over(partition by position order by salary) as rnk,
 	sum(salary) over(partition by position order by salary rows between unbounded preceding and current row) as cumulative_budget
 from candidates
 order by position DESC, cumulative_budget DESC) a
where position = 'senior' and cumulative_budget <= 50000
limit 1) sen -- 表内为实现预算内最大化的senior雇佣花费和雇佣senior的人数
on b.cumulative_budget + sen.cumulative_budget <= 50000
     -- 使用join实现雇佣senior的remains最大化的junior雇佣数量
order by b.rnk 
limit 1)final

-- 另：union后的结果作为新的母表进行嵌套查询的语法为
select 
from(
	(select A from A)
	union
	(select B from B)) as alias -- 即内部需要括号但不需要alias
```

（写了一早上才写完.... :cry:

Reference：

[MySQL全连接(Full Join）实现](https://blog.csdn.net/wushanyun1989/article/details/9265019)

[SQL统计字段值的累加和](https://blog.csdn.net/oppo62258801/article/details/103060420)
