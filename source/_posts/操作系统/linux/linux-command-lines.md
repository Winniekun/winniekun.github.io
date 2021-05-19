---
title: 程序员的魔法---命令行
date: 2020-04-10 09:13:07
tags: 命令行
categories: Linux
thumbnail: https://i.loli.net/2020/04/10/PAWsu47k2IgKEVY.png
---

## 前言

不知不觉已经使用`Ubuntu`四年了，算是一只老白了。命令行，感觉对于程序员来说，简直和魔法一样，输入一行命令，然后就能得到你想要的结果，极其的amazing。当然，也和魔法一样难以使用，因为要记住很多很多的内容。

<!--more-->

## 常用操作

自言自语 echo

在哪儿 pwd(Print Working Directory)

换个地方 cd (change directory)

瞅瞅 ls

* ls
* ls -l
  * 显示当前目录下文件信息
* ls -la
  * 列出所有文件，包括以.开头的隐藏文件
* ls -lh
  * 做单位换算，将byte换算成K，M，G等.

寻求帮助 man（manual）、 xxx -h  xxx --help



## 文件内容

* 打印文件内容 cat(concatenate and print files)
  
  * cat a.txt
* 头和尾 head、tail
  * head a.txt
  * tail a.txt
* 交互浏览 less
  
  * less a.txt
* 内容查找 / 、grep
  
  *  less  | grep xxx (grep -n 显示行数)
* 单词统计
  
  * wc
  
  

## 管道和重定向

* **重定向**: 改变输入输出设备
  * 标准输入/ 输出: 控制台/键盘  / 屏幕
  * echo hello > hello.txt
    * hello 输入到hello.txt
  * echo world >> hello.txt
    * world 追加输入到hello.txt的下一行
    * 再追加输入下一行仍然使用>>
  * cat < hello.txt
    * 将hello.txt文件输出到控制台
* **管道**: 建一个命令的标准输出作为下一个命令的标准输出
  * man less | grep sim 
    * 查看less的用户手册, 然后匹配出包含sim的行
  * man less | grep sim - n | grep That > that.txt
    * 查看less的用户手册, 然后匹配出包含sim的行, 之后将又包含that的行存入that.txt

## 文件权限(chmod)

`Change Mode`改变文件权限,  Owner - Group - Others

* chmod +x foo
  * 增加可执行权限/ +w +r
* chmod  -x foo
  * 移除可执行权限/ -w -r
* chmod 740 foo
  * 把foo的权限设置成740
    * Owner: 7 = 1 + 2 + 4 = 可执行 + 可写 +可读
    * Group: 4  = 4 = 可读
    * Others: 0 = 没有任何权限
  * 常见:
    * -rw-r--r--r 
      * 默认(644)
    * -rwxr-xr-x
      * 755
    * -rwxrwxrwx
      * 777

## 文件基本操作

* mv(移动文件  重命名 剪切+粘贴)
  * mv 需要移动文件 /指定目录  (移动文件)
  * mv  hwllo.txt hello.txt (重命名)
* cp(复制文件 复制文件夹)
  * cp a.txt  a_copy.txt
  * cp -r dir1  dir2
* rm(删除文件 删除文件夹)
  * rm a.txt (删除一个文件)
  * rm a.txt b.txt c.txt(删除多个文件 也可使用 rm *.txt)
  * rm -r dir (删除文件夹)

**yes**: 不停输出



## 进程管理

1. top: 显示或者更新排序过的进程信息
   * 默认按照CPU占用率排序(导致风扇很响的那个玩意儿)
2. ps: 显示进程状态(Process Status)
   * 默认显示当前用户**有控制终端**的进程
   * ps  aux : 显示所有进程, 包括其他用户
   * ps aux | grep idea | wc -l : 查看idea起了多少进程
3. kill/killall: 杀进程
   * kill: 终止或者给进程信号
     * kill -singal_number/-singal_name PID
     * kill PID
     * kill -9/-KILL PID 
   * killall: 按照名字终止进程









