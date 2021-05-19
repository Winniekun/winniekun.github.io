---
title: Mysql-Explain
date: 2020-03-29 13:21:29
tags: 文档
categories: MySQL
thumbnail: https://i.loli.net/2020/03/11/7mPyuYUE3w8bKf6.png
---

## MySQL Explain

<!--more-->

| 名称          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| Id            | `id`是用来顺序标识整个查询中SELELCT 语句的，在嵌套查询中id越大的语句越先执行 |
| select_type   |                                                              |
| table         | 对应行正在访问哪一个表，表名或者别名                         |
| partitions    | 分区位置                                                     |
| type          | type显示的是访问类型，是较为重要的一个指标，表示MySQL在表中找到所需行的方式 |
| possible_keys | 显示查询使用了哪些索引，表示该索引可以进行高效地查找，但是列出来的索引对于后续优化过程可能是没有用的 |
| key           | key列显示MySQL实际决定使用的键（索引）                       |
| key_len       | key_len列显示MySQL决定使用的键长度                           |
| ref           | ref列显示使用哪个列或常数与key一起从表中选择行。             |
| rows          | rows列显示MySQL认为它执行查询时必须检查的行数。注意这是一个预估值。 |
| filtered      |                                                              |
| extra         | Extra是EXPLAIN输出中另外一个很重要的列，该列显示MySQL在查询过程中的一些详细信息，MySQL查询优化器执行查询的过程中对查询计划的重要补充信息。 |

