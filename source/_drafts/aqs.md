---
title: 浅析AQS
mathjax: false
tags: 
- juc
- 并发
categories: java
---

## 序

因为接触到AQS的使用不是特别多，如果说有什么使用心得，那就是没有。但是AQS是JUC里面绝大多数实现的依赖支撑，所以为了迎接以后可能接触到的业务场景，避免一些错误使用或者理解。啃AQS源码还是很有必要的。

emmmm～～～

let‘s go

## 疑惑

我们知道，AQS的核心是提供了一套同步机制，所以在阅读源码之前，可以先尝试问下自己，如何设计出一种框架，来实现同步机制，需要哪些要求。

1. 通用性，下层实现透明的同步机制，同时与业务解耦
   1. 可以类比为经验知识库的接口调用封装。
2. 利用CAS，原子地修改共享标记位
   1. 将某个值作为对竞争资源的标识位，在多个线程想要获取共享资源的时候，先读取这个标记位
   2. 如果标记显示是共享资源空闲，则赋予该线程操作共享资源的权限，同时将这个标记为设置为不可用。
   3. 那么其他线程该如何处理？
      1. 如果获取不到共享资源
         1. 仅仅是想快速尝试获取共享资源，如果获取不到，继续执行其他任务
            - 直接返回true、false
         2. 有的业务线程依赖当前的共享资源，也就是必须要获取共享资源，则需要进行等待。
            - 业务线程轮询，消耗CPU，系统利用率不高
            - 直接返回true、false，交给业务线程自己处理
              - 既然是框架了，主要就是为了简化上层操作、屏蔽上层业务的复杂度
              - 这样反而增加了上层业务开发的难度
            - 设计队列，让需要获取共享资源的线程进入队列，然后共享资源可获取时，让队列中的线程依次去获取共享资源
3. 等待队列的设计
   1. 如何维护这个队列
   2. 线程的阻塞和唤醒如何实现



## 属性

```java
// 头结点，你直接把它当做 当前持有锁的线程 可能是最好理解的
private transient volatile Node head;

// 阻塞的尾节点，每个新的节点进来，都插入到最后，也就形成了一个链表
private transient volatile Node tail;

// 这个是最重要的，代表当前锁的状态，0代表没有被占用，大于 0 代表有线程持有当前锁
// 这个值可以大于 1，是因为锁可以重入，每次重入都加上 1
private volatile int state;

// 代表当前持有独占锁的线程，举个最重要的使用例子，因为锁可以重入
// reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁
// if (currentThread == getExclusiveOwnerThread()) {state++}
private transient Thread exclusiveOwnerThread; //继承自AbstractOwnableSynchronizer
```



### Node

![Node内部内容](https://i.loli.net/2021/04/16/m95lIf2h6EZOBSi.png)

**waitstatus**

1. CANCELLED = 1

   - ```java
     线程取消了争抢这个锁
     ```

2. SIGNAL = -1;

   1. ```java
      其表示当前node的后继节点对应的线程需要被唤醒
      ```

3. CONDITION = -2

```java
// 取值为上面的1、-1、-2、-3，或者0(以后会讲到)
    // 这么理解，暂时只需要知道如果这个值 大于0 代表此线程取消了等待，
    //    ps: 半天抢不到锁，不抢了，ReentrantLock是可以指定timeouot的。。。
```

Node 的数据结构其实也挺简单的，就是 thread + waitStatus + pre + next 四个属性而已



所以对于整个AQS，其内部的数据结构可以理解为这样：

![AQS数据结构](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210607142755556.png)

对整个AQS的粗略理解如下：

![AQS](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210607170352289.png)







## 总结

总结下我目前所理解的AQS（独占的非公平的可重入锁）

![AQS流程](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210607192254013.png)

AQS，一个用来构建锁和同步器的框架，许多的经典的同步器都是基于AQS构建出来的，譬如CountDownLatch、Semaphored。其内部使用同步队列帮我们解决了实现同步器时的一些细节问题，譬如何时阻塞线程，如何按照顺序唤醒线程。

核心思想：AQS通过一个被volatile修饰的int类型的成员变量表示同步状态（共享资源），然后使用cas对这个同步状态进行修改，从而保证线程安全。如果被请求的共享资源空闲，则将当前的请求线程设置为有效的工作线程，然后将对应的共享资源设置为锁定状态。如果共享资源被占用，则通过CLH同步队列将暂时用不到的线程封装成一个节点加入到队列中，同时在适当的时候对其进行阻塞和唤醒。

基于AQS实现的应用（目前接触到的）：

1. lock

   区别于synchronozed的锁，同时还支持了更加的灵活丰富操作，譬如可定时的尝试获取锁（避免了死锁的发生）、可中断的获取锁。

2. CountDownLatch

   一个或一组操作等其他操作执行结束之后，继续执行。

3. Semaphore

   用于多个共享资源的互斥，或者用于控制并发数量





## 后续任务

![AQS后续计划](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210531144704740.png)



## references

- [寒食君---必知必会的Java AQS机制](https://www.bilibili.com/video/BV12K411G7Fg)