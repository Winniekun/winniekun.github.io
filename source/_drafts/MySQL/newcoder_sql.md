---
title: 牛客网SQL练习题
date: 2020-08-10 18:49:53
tags: mysql基础
categories: mysql
thumbnail:
---



## 第一题

<!--more-->

## 第二题



## 第三题



## 第四题



## 第五题



## 第六题



## 第七题



## 第八题

[找出所有员工当前具体的薪水salary情况](https://www.nowcoder.com/practice/ae51e6d057c94f6d891735a48d1c2397?tpId=82&tags=&title=&diffculty=0&judgeStatus=0&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)

### 描述: 

找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示

### 题解:

```sql
SELECT DISTINCT salary 
FROM salaries
WHERE to_date = '9999-01-01'
ORDER BY salary DESC;
```



## 第九题

 获取所有部门当前manager的当前薪水情况

### 描述:

获取所有部门当前(dept_manager.to_date='9999-01-01')manager的当前(salaries.to_date='9999-01-01')薪水情况，给出dept_no, emp_no以及salary(请注意，同一个人可能有多条薪水情况记录)

### 题解:

```java
SELECT d.dept_no, d.emp_no, s.salary
FROM dept_manager d LEFT JOIN salaries s
ON d.emp_no = s.emp_no
WHERE s.to_date = '9999-01-01'
AND d.to_date = '9999-01-01';
```



## 第十题

### 描述:

 获取所有非manager的员工emp_no

### 题解:

```sql
SELECT e.emp_no 
FROM employees e
WHERE e.emp_no NOT IN (SELECT emp_no FROM dept_manager);
```



## 第十一题

获取所有员工当前的manager

### 描述

获取所有员工当前的(dept_manager.to_date='9999-01-01')manager，如果员工是manager的话不显示(也就是如果当前的manager是自己的话结果不显示)。输出结果第一列给出当前员工的emp_no,第二列给出其manager对应的emp_no。

### 题解

```sql
SELECT e.emp_no, d.emp_no
FROM dept_emp e LEFT JOIN dept_manager d
ON d.dept_no = e.dept_no
WHERE d.to_date = '9999-01-01'
AND d.emp_no != e.emp_no;
```



## 第十二题

获取所有部门中当前员工薪水最高的相关信息

### 描述

获取所有部门中当前(dept_emp.to_date = '9999-01-01')员工当前(salaries.to_date='9999-01-01')薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary

### 题解

```sql
select dept_no,s.emp_no,max(salary) salary 
from dept_emp de left join salaries s on de.emp_no = s.emp_no 
where de.to_date = '9999-01-01' and s.to_date = '9999-01-01' 
group by dept_no 
having s.salary = max(salary)

```



## 第十三题

从titles表获取按照title进行分组

### 描述

从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。

### 题解

```sql
SELECT title, COUNT(title) 
FROM titles
GROUP BY title
HAVING COUNT(title) >= 2;
```



## 第十四题

从titles表获取按照title进行分组

### 描述

从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。
注意对于重复的emp_no进行忽略(即emp_no重复的title不计算，title对应的数目t不增加)。

### 题解

```sql
SELECT title, COUNT(DISTINCT emp_no) 
FROM titles
GROUP BY title 
HAVING COUNT(DISTINCT emp_no) >= 2;
```



## 第十五题

查找employees表

### 描述

查找employees表所有emp_no为奇数，且last_name不为Mary(注意大小写)的员工信息，并按照hire_date逆序排列(题目不能使用mod函数)

### 题解

```sql
SELECT * 
FROM employees
WHERE emp_no%2 = 1 AND last_name != 'Mary'
ORDER BY hire_date DESC;
```



## 第十六题

统计出当前各个title类型对应的员工当前薪水对应的平均工资

### 描述

统计出当前(titles.to_date='9999-01-01')各个title类型对应的员工当前(salaries.to_date='9999-01-01')薪水对应的平均工资。结果给出title以及平均工资avg。

### 题解

```sql
SELECT t.title, AVG(s.salary) avg
FROM titles t LEFT JOIN salaries s ON t.emp_no = s.emp_no 
WHERE t.to_date = '9999-01-01' AND s.to_date='9999-01-01'
GROUP BY t.title ;
```



## 第十七题

获取当前薪水第二多的员工的emp_no以及其对应的薪水salary

### 描述

获取当前（to_date='9999-01-01'）薪水第二多的员工的emp_no以及其对应的薪水salary

### 题解

```sql

```



## 第十八题

### 描述

### 题解



## 第十九题

查找所有员工的last_name和first_name以及对应的dept_name

### 描述

查找所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工

### 题解

```sql
select e.last_name,e.first_name,d.dept_name 
from employees e left join dept_emp de on e.emp_no = de.emp_no
left join departments d on de.dept_no = d.dept_no
```



## 第二十题

### 描述

查找员工编号emp_no为10001其自入职以来的薪水salary涨幅(总共涨了多少)growth(可能有多次涨薪，没有降薪)

### 题解

```sql
SELECT MAX(salary) - MIN(salary) 
FROM salaries 
WHERE emp_no = '10001';
```



## 第二十一题

### 描述

查找所有员工自入职以来的薪水涨幅情况，给出员工编号emp_no以及其对应的薪水涨幅growth，并按照growth进行升序



### 题解

```sql
SELECT end.emp_no, (end.salary - start.salary) AS growth FROM 
-- 获得当前工资表，放入临时表end中
(select emp_no,salary from salaries where to_date = '9999-01-01') end
LEFT JOIN 
-- 获取刚入职工资表, 放入临时表start中
(SELECT s.emp_no, s.salary FROM salaries s LEFT JOIN employees e ON 
    s.emp_no = e.emp_no WHERE  s.from_date = e.hire_date) start 
ON end.emp_no = start.emp_no ORDER BY growth;
```



## 第二十二题

### 描述

统计各个部门的工资记录数，给出部门编码dept_no、部门名称dept_name以及部门在salaries表里面有多少条记录sum

### 题解

```sql

```



## 第二十五题

```sql
select emp_sal.emp_no,mag_sal.manager_no,
emp_sal.emp_salary,mag_sal.manager_salary
from (
    -- 查询员工薪资
    select de.emp_no,de.dept_no,s1.salary as emp_salary
    from dept_emp de
    left join salaries s1 on de.emp_no = s1.emp_no
    where s1.to_date='9999-01-01'
    and de.to_date='9999-01-01'
)as emp_sal
left join(
    -- 查询manager薪资
    select dm.emp_no as manager_no,dm.dept_no,s2.salary as manager_salary
    from dept_manager dm
    left join salaries s2 on dm.emp_no = s2.emp_no
    where s2.to_date='9999-01-01'
    and dm.to_date='9999-01-01'
)as mag_sal
on emp_sal.dept_no=mag_sal.dept_no
where mag_sal.manager_salary<emp_sal.emp_salary;
```



## 第二十六题



## 第二十七题

### 题目描述

给出每个员工每年薪水涨幅超过5000的员工编号emp_no、薪水变更开始日期from_date以及薪水涨幅值salary_growth，并按照salary_growth逆序排列。

```sql
SELECT a.emp_no, b.from_date, b.salary-a.salary AS salary_growth
FROM salaries  a 
LEFT JOIN salaries b ON a.emp_no = b.emp_no
WHERE a.to_date = b.from_date
AND (b.salary-a.salary) > 5000
ORDER BY salary_growth DESC;
```

