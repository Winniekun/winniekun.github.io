---
title: Exception和Error的区别
date: 2020-07-02 19:57:04
tags: 
	- java基础
	- 面试36讲
categories: java
thumbnail:
---

## 解答

<!--more-->

**Exception**和**Error**均继承自**Trowable**, 如下图所示:

![依赖图](https://i.loli.net/2020/07/02/aIOqAKGJe8yZ9cX.png)

在 Java 中只有 **Throwable** 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型, *Exception和Error代表了对不同异常情况的分类*

1. Exception:

   * 程序正常运行的情况下, 可以预料的意外情况, 可以并且应该被捕获, 之后进行处理, 最后仍能程序继续正常运行

   又可以被细分为**可检查异常**和**不可检查异常**

   1. 可检查异常:
      * 编译期间出现的异常, 必须进行捕获处理
   2. 不可检查异常:
      * 运行期间出现的异常, 譬如ArrayIndexOutOfBoundsException, NullPointerException之类

2. Error:

   * 正常情况下, 不大可能出现的异常,  绝大多数Error的出现会导致程序崩溃, 不便于也不需要捕获, 譬如OutOfMemoryError之类, 都是Error的子类



