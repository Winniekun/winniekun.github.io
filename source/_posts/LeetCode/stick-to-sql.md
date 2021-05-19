---
title: 死磕SQL
date: 2021-04-20 19:56:11
tags: 
   - SQL
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png
---



## 前言

LeetCode死磕系列十： SQL

<!--more-->

作为一个后端开发程序员，SQL功底还是很重要的！！！

所以还是要练习。

> 还记得去年被字节面试的时候，连SQL都写不出来。。。

## LeetCode SQL题目整理

刷了LC上的top70之后，感觉考察最多的还是多表的联合查找，毕竟实际中也不会仅仅是单表的查询。

**主要题型：**

1. TOP N 问题
2. 排名问题
   1. 滑动窗口
3. 自链接问题

## 题解

### TOP N问题

#### [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

**思路：**

1. 使用limit

   注意判空、去重

   ```sql
   select 
   ifnull ((select 
       distinct Salary 
   from
       Employee
   order by Salary desc
   limit 1, 1) , null) as SecondHighestSalary;
   ```

   

2. 先使用max查询最高的薪水 max，之后再通过自查询，查询小于max的max

#### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

```sql
-- 思路： 寻找最大薪资的数据
-- join
-- 1. 直接join group by  选取最大的即可 但是 无法保证最大的薪资不重复 失败
-- 2. 先通过员工信息 寻找最大的薪资+id 的数据A，然后再join 两张表，将A中对应的数据返回

-- 1. 
select 
    d.Name as Department,
    e.Name as Employee,
    e.Salary as Salary
from 
    Employee e inner join Department d on e.DepartmentId = d.Id
where 
    (e.DepartmentId, e.Salary) in
    (select 
        DepartmentId,
        max(Salary)
    from 
        Employee
    group by 
        DepartmentId);
```



#### [177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

LIMIT PAGE, OFFSET

跳过 PAGE * OFFSET条数据，接下来的offset条数据， 因为题目中未说明薪资是否不重复，所以还需要使用distinct去重

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  set N:= N-1;
  RETURN (  
      # Write your MySQL query statement below.
    select 
        distinct Salary
    from 
        Employee
    order by 
        Salary desc
    limit N,1
  );
END
```

#### [185. 部门工资前三高的所有员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

```sql
select d.Name as Department, 
e2.Name as Employee, 
e2.Salary from 
Department d inner join 
(
    select e.*, 
    dense_rank() over(partition by DepartmentID 
    Order by Salary DESC) as 'rank'
    from Employee e 
) e2 on d.Id= e2.DepartmentID
where e2.rank<=3
order by Department AND Salary
```



### 排名问题

#### [178.分数排名](https://leetcode-cn.com/problems/rank-scores/)

```sql
select 
	Score, dense_rank() over(order by Score desc) as `Rank`
from Scores;
```



### 自链接问题

#### [603. 连续空余座位](https://leetcode-cn.com/problems/consecutive-available-seats/)

```sql
-- 几个朋友来到电影院的售票处，准备预约连续空余座位。
-- 你能利用表 cinema ，帮他们写一个查询语句，获取所有空余座位，并将它们按照 seat_id 排序后返回吗？

select distinct a.seat_id
from cinema a join cinema b
  on abs(a.seat_id - b.seat_id) = 1
  and a.free = true and b.free = true
order by a.seat_id
```

