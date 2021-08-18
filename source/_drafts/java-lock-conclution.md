---
title: Java锁机制
mathjax: false
tags: 锁
categories: java
---

## 锁

锁这一抽象概念是在Java中是如何实现的

### 内置锁（基于互斥锁实现）

在Java中，每个对象都有一把锁，放置于对象头中，用于记录当前对象被哪个线程所持有。相对于实例数据，对象头属于额外开销，所以被设计的极小来提高效率。对象头中的markword更加体现了这一点，且是非结构化的，这样在不同的锁状态下，能够复用相同的bit位，markword中就有存储锁的信息的部分。

![markword](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210525175926275.png)



我们知道在Java中Synchronized能够实现的线程的同步，synchronized被编译之后会生成monirotenter和monitorexit两个字节码指令，依赖这两个字节码指令实现线程同步。

Monirtor：可以理解为只能容纳一名客人的房间，而线程可以等比于客人。

![monitor](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210525181050205.png)

sychronized整体的执行流程：

性能问题：因为synchronized被编译之后，使用的是monitorenter和monitroexit两个字节码指令，而这两个字节码指令实质上是依赖于操作系统中的互斥锁（mutex lock）实现，同时Java线程可以理解为对操作系统中线程的映射，所以每当挂起/唤醒一个线程，都需要涉及到操作系统的内核态内容的切换。属于重量级操作，甚至是有可能切换的时间远远超过于线程本身的运行时间。但是从Java6开始，引入了偏向锁、轻量级锁、重量级锁的锁优化过程，进而优化了其性能。

![锁梗概](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210525181927956.png)

#### 无锁

cas

#### 偏向锁



#### 轻量级锁

#### 重量级锁

### 无锁模式

cas、aqs、juc



## TODO

* [ ] 对象的结构

   

```java
初始化;
for (int len = 2; len <= N; len++) { // 先遍历区间大小
		for (int i = 0; i + len - 1 < N; i++) { // 再遍历起始位置（初始位置）
      	int j = i + len - 1; // 右端点位置
      	// 进行状态转移
        // 具体的状态转移还是要根据题意来
        // 下面的属于一种情况
        // 1. 在[i:j]区间找到最优的结果
        for (int k = i; k <= j; k++) {
          	dp[i][j] = best();
        }
    }
}

return dp[0:N];
```

