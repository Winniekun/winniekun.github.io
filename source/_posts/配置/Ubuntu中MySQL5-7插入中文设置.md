---
title: Ubuntu中MySQL5.7插入中文设置
date: 2020-03-11 17:44:55
tags: 配置
categories: mysql
thumbnail:
---

## 前言
其实关MySQL中插入中文,自己前年是设置过的,之后就一直没有管过,后来系统重装之后,重新安装配置各种环境之后,今天开始用MySQL的时候,才发现这个问题还没有解决,然后一时也想不起来怎么弄了,就重新网上搜索教程,最后想了下还是写一下配置过程吧,以后备用

<!--more-->

## 第一步
```
cp /etc/mysql/mysql.conf.d/mysqld.cnf  /etc/mysql/my.cnf
```
## 第二步
修改/etc/mysql/my.cnf文件，找到[client]标记后在下方添加一行代码：
```
default-character-set=utf8
```
如果找不到[client]标记，就直接在文件末尾插入：
```
[client]
default-character-set=utf8
```
## 第三步
找到[mysqld]标记，在该标记下方添加两行
```
character-set-server=utf8
collation-server=utf8_general_ci
```
## 最后
### 执行重启mysql命令：
```
sudo service mysql restart
```
### 查看修改结果
进入MySQL，之后输入如下命令
```
show variables like 'character%';
```