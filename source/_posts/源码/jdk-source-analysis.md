---
title: JDK源码分析计划
date: 2020-05-13 13:40:58
tags: JDK源码
categories: 源码
thumbnail: https://i.loli.net/2020/03/10/R2ExlKrgYVPeM6U.png
---



## 前言

<!--more-->

自己也断断续续的读了一些`JDK`的源码了，并且也写了总结，但是尴尬的是，该忘得还是忘了，所以打算系统的理理，然后没看完一个除了总结，再做个思维导图。阅读JDK的目的有三：

1. 学好数据结构以及相关算法
   1. 毕竟是内功，该硬啃还是要硬啃
   2. 不希望自己到时候追悔莫及
2. 能让自己的Java的理解和熟练度更上一层楼
   1. 莫问Python嘞，问就是爱过
3. 面试
   1. 这也是没得办法，感觉目前面试不问个源码问题，都不像面试

顺带把`LeetCode`相关的算法刷一遍:smile:

### 总体进程

1. 常用数据结构手撸一遍
   1. 算法第四版
   2. 数据结构与算法分析

2. 阅读JDK源码，记录
   1. 参考其他大佬的总结，查缺补漏

3. 相关LeetCode题目刷了

## JDK 集合类

![所有集合](https://i.loli.net/2020/06/19/fwuW7NYg64poKA9.png)

太复杂, 然后去除JUC上的一些实现类, 再看一下

![collection-map-不包含JUC中的类](https://i.loli.net/2020/05/13/d49JiEoCbmezAQR.png)



### **Collection**

#### **List**

List中的元素是有序的, 可重复的, 主要的实现方式是**动态数组**和**链表**

![List](https://i.loli.net/2020/06/19/GbwoBkvLu862IKm.png)

Java中提供的List的实现主要有`ArrayList、LinkedList、CopyOnWriteArrayList`，另外还有两个古老的类`Vector`和`Stack`。

1. `java.util.LinkedList` :heavy_check_mark:
5. `java.util.Vector` :heavy_check_mark:
6. `java.util.Stack` :heavy_check_mark:
7. `java.util.ArrayList`:heavy_check_mark:
5. `java.util.concurrent.CopyOnWriteArrayList` :heavy_check_mark:

#### **Queue**

队列是遵循着一定原则的入队出队操作的集合, 先进先出, 一般来说在堆尾添加元素(入队), 在队头删除元素(出队), 但是也不一定, 不如优先队列的原则就不同   

![Queue](https://i.loli.net/2020/06/19/3VarMDGLi4tB1dc.png)

Java中提供的Queue的实现有: `PriorityQueue、ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue、LinkedTransferQueue、DelayQueue、ConcurrentLinkedQueue。`

1. `java.util.PriorityQueue`
2. `java.util.concurrent.ArrayBlockingQueue`
3. `java.util.concurrent.LinkedBlockingQueue`
4. `java.util.concurrent.SynchronousQueue`
5. `java.util.concurrent.PriorityBlockingQueue`
6. `java.util.concurrent.LinkedTransferQueue`
7. `java.util.concurrent.DelayQueue`
8. `java.util.concurrent.ConcurrentLinkedQueue`

##### **Deque**

`Deque`是一种特殊的队列，它的两端都可以进出元素，故而得名双端队列（Double Ended Queue）。

![Deque](https://i.loli.net/2020/06/19/yGEa4p1QmUlKcBo.png)

Java中提供的`Deque`的实现主要有`ArrayDeque、LinkedBlockingDeque、ConcurrentLinkedDeque、LinkedList。`

1. `java.util.Deque`
2. `java.util.ArrayDeque`
3. `java.util.concurrent.ConcurrentDeque`

#### **Set**

![Set](https://i.loli.net/2020/06/19/kMiKCyLu9czoAxJ.png)

Java中提供的Set的实现主要有`HashSet、LinkedHashSet、TreeSet、CopyOnWriteArraySet、ConcurrentSkipListSet`。

1. `java.util.HashSet`
2. `java.util.TreeSet`
3. `java.util.LinkedHashSet`
4. `java.util.concurrent.CopyOnWriteArraySet`
5. `java.util.concurrent.ConcurrentSkipListSet`





### **Map**

![Map](https://i.loli.net/2020/06/19/6v8fql7k3OcixPE.png)

java中提供的Map的实现主要有`HashMap、LinkedHashMap、WeakHashMap、TreeMap、ConcurrentHashMap、ConcurrentSkipListMap`，另外还有两个比较古老的Map实现`HashTable、Properties`。

1. `java.util.TreeMap` :heavy_check_mark:
2. `java.util.WeakHashMap`
3. `java.util.LinkedHashMap`
4. `java.util.HashMap` :heavy_check_mark:
5. `java.util.concurrent.ConcurrentHashMap`
6. `java.util.concurrent.ConcurrentSkipListMap`
7. `java.util.Hashtable`
8. `java.util.Properties`





## 集合类概述

逛博客的时候，发现一张神图

![java-collection-cheat-sheet](https://i.loli.net/2020/05/13/zyYEUgGftpMTcv8.png)

> [Java Collections Framework Cheat Sheet](https://pierrchen.blogspot.com/2014/03/java-collections-framework-cheat-sheet.html#more)



## 阅读套路

大体上的阅读框架写好了，接下来就是细致的硬啃了，可以先从顶层的接口开始，然后是Abstact类，这样方便理解不同的具体的集合类必备的通用方法等，然后再看这些具体类对这些通用方法的具体实现。顺序还是老样子：`增`、`删`、`改`、`查`，然后是其他的细节部分。





## References

* [【死磕 Java 集合】— 总结篇](http://cmsblogs.com/?p=4781)