## 一些小技巧&概念

### concat

用于拼接字符串

```sql
select
    concat(last_name, '\'', first_name)
from
    employees
-- Facello'Georgi
-- Simmel'Bezalel
```

### group_concat

![image-20210228110606183](https://i.loli.net/2021/02/28/pcq9VtlK1INmrR2.png)



1. 连接同一列字段：group_concat( [distinct] <要连接的字段> [order by 排序字段 asc/desc ] [separator '分隔符'] ) 。分隔符可以选择省略，省略时默认为逗号，这里还是写出来了。另外还有一点需要注意，group_concat函数中的各个参数之间用空格隔开，不能用逗号隔开，不然会出错。

2. 按照dept_no进行汇总，所以要对dept_no进行分组。

```sql
select 
    dept_no, group_concat(emp_no) as employees
from
    dept_emp
group by dept_no;

-- 更加明朗
select dept_no,group_concat(emp_no separator ',') as employees
from dept_emp
group by dept_no;
```



### replace

```sql
-- replace(column, 替换的内容，被替换的内容)
-- 查找字符串'10,A,B' 中逗号','出现的次数cnt。
select 
    (length("10,A,B") - length(replace("10,A,B", ",", ""))) as cnt
```

### substring

```sql
--subtring(对象字符串,截取的起始位置,截取的字符数)
select
    first_name
from 
    employees
order by substring(first_name, (length(first_name)-1), 2) asc;
```

### limit

```sql
limit m, n
-- m : 偏移量也就已经过去了几页
-- n : 数量 也就是每一个的大小
-- 每页 5 条 要求返回第三页的数量
-- limit (curPage-1)*pageSize,pageSize
limit 2 * 5 : 5
```



### **exists**



## 插入数据

1. insert ignore into

   数据已存在，则忽略该sql

2. 

## 索引

1. 添加索引

   ```sql
   alter table actor
       add unique index uniq_idx_firstname (first_name), -- 唯一索引
       add index idx_lastname (last_name) -- 普通索引
   ```

2. 强制走索引

   ```sql
   select 
       *
   from 
       salaries force index (idx_emp_no) 
   where 
       emp_no = 10005
       
   强制索引：FORCE INDEX(<索引名>);
   SELECT * FROM <表名> FORCE INDEX (<索引名>)
   ```

   

## 视图

1. 创建视图

   ```sql
   CREATE VIEW actor_name_view (first_name_v, last_name_v)
   as 
   select first_name, last_name
   from actor;
   
   CREATE VIEW <视图名称> (<视图列名1>,<视图列名2>…)
   AS
   <select 语句>；
   ```



## 修改表结构

1. 添加字段

   ```sql
   alter table actor
       add column create_date datetime not null default '2020-10-01 00:00:00' after last_update
   ```

2. 添加索引

   ```sql
   alter table actor
   		add index idx_emp_no (emp_no)
   ```

   

3. 添加外键

   ```sql
   alter table audit
       add foreign key (emp_no) references employees_test(id);
   ```

   

## 触发器

1. 构建触发器

   ```sql
   CREATE TRIGGER audit_log AFTER insert
   ON employees_test FOR EACH ROW
   BEGIN
       INSERT INTO audit VALUES(NEW.ID, NEW.NAME);
   END;
   
   CREATE TRIGGER <触发器名称> <触发时机> <触发事件>
   ON <表名> FOR EACH ROW <触发后执行的语句>;
   ```

   

## 滑动窗口

### 窗口函数和聚合窗口的区别

1. 聚合函数是将多条记录聚合为一条。窗口函数是每条记录都会执行
2. 聚合函数也可以用于窗口函数

### 窗口函数的基础用法

```sql
函数名 OVER 子句
```

1. PARTITION BY
   1. 窗口按照哪些字段进行分组
   2. 按照哪些字段进行排序，窗口函数将按照排序后的记录顺序进行编号
2. ORDER BY 
   1. 按照哪些字段进行排序，窗口函数按照排序后的顺序进行编号
3. FRAME 



### 涉及题目

1. 本题关键在于把sum聚合函数作为窗口函数使用，所有聚合函数都能用做窗口函数，其语法和专用窗口函数完全相同。

   sum(<汇总列>) over(<排序列>) as 别名；

   ```sql
   select 
       emp_no, 
       salary,
       sum(salary) over(order by emp_no) as running_total
   from 
       salaries
   where to_date = '9999-01-01'
   ```

   

   ​		
   
2. [lc 178](https://leetcode-cn.com/problems/rank-scores/solution/zi-jie-ti-ku-178-zhong-deng-sqlfen-shu-pai-ming-1s/)

   值得注意的三个窗口函数。现在给定五个成绩：99，99，85，80，75。
   DENSE_RANK()。如果使用 DENSE_RANK() 进行排名会得到：1，1，2，3，4。
   RANK()。如果使用 RANK() 进行排名会得到：1，1，3，4，5。
   ROW_NUMBER()。如果使用 ROW_NUMBER() 进行排名会得到：1，2，3，4，5。

   ```sql
   select Score, dense_rank() over(order by Score desc) as `Rank`
   from Score;
   ```

   