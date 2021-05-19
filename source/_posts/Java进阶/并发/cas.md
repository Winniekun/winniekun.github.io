---
title: cas
date: 2020-05-28 11:45:05
tags: 并发
categories: java
thumbnail:
---

## CAS概念

<!--more-->

CAS ，Compare And Swap ，即比较并交换。Doug Lea 大神在实现同步组件时，大量使用CAS 技术，鬼斧神工地实现了Java 多线程的并发操作。整个 AQS 同步组件、Atomic 原子类操作等等都是基 CAS 实现的，甚至 ConcurrentHashMap 在 JDK 1.8 的版本中，也调整为 CAS + `synchronized` 。可以说，CAS 是整个 J.U.C 的基石。

![CAS](https://i.loli.net/2020/05/28/aMqRzm1jXhBHEPf.png)

在CAS中，有这样三个值：

- V：要更新的变量(var)
- E：预期值(expected)/原值(old value)
- N：新值(new)

比较并交换的过程如下：

判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。

所以这里的**预期值E本质上指的是“旧值”**。

我们以一个简单的例子来解释这个过程：

1. 如果有一个多个线程共享的变量`i`原本等于5，我现在在线程A中，想把它设置为新的值6;
2. 我们使用CAS来做这个事情；
3. 首先我们用i去与5对比，发现它等于5，说明没有被其它线程改过，那我就把它设置为新的值6，此次CAS成功，`i`的值被设置成了6；
4. 如果不等于5，说明`i`被其它线程改过了（比如现在`i`的值为2），那么我就什么也不做，此次CAS失败，`i`的值仍然为2。

在这个例子中，`i`就是`V`，`5`就是`E`，`6`就是`N`。

那有没有可能我在判断了`i`为5之后，正准备更新它的新值的时候，被其它线程更改了`i`的值呢？

不会的。因为CAS是一种**原子操作**，它是一种系统原语，是一条CPU的原子指令，从CPU层面保证它的原子性

> **当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。**

### CAS 的实现

#### Java实现CAS的原理 - Unsafe类

在Java中, 如果有一个方法是native的, 那么Java不负责实现它, 而是交由底层的JVM使用C或C++去实现. 在Java中有一个`Unsafe`类, 位于`sun.misc`包中, 里面是native方法, 其中有几个关于CAS的

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6); 
```

J.U.C 下的 Atomic 类，都是通过 CAS 来实现的, 以AtomicInteger为例子, 阐述CAS的实现

#### AtomicInteger

```java
// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
 
private volatile int value;
```

- Unsafe 是 CAS 的核心类，Java 无法直接访问底层操作系统，而是通过本地 `native` 方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe ，**它提供了硬件级别的原子操作**。
- `valueOffset` 为变量值在内存中的偏移地址，Unsafe 就是通过**偏移地址**来得到数据的原值的。
- `value` 当前值，使用 `volatile` 修饰，保证多线程环境下看见的是同一个。



```java
// AtomicInterger类
public final int addAndGet(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
}
// Unsafe(核心)  CAS的具体实现
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        // 放入do内, 时刻获取最新的期望值(期望值在多线程中会被其他的线程修改)
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
// Unsafe
// var1: 对象
// var2: 对象地址
// var4: 预期值
// var5: 新值
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

`CAS`是`无锁`的基础，它允许更新失败。所以经常会与while循环搭配，在失败后不断去重试。 `getAndInt()`方法, 其返回的值是预期值(E), 也就是旧值, 新值(N)是`var5+var4`

这里使用的是**do-while循环**。这种循环不多见，它的目的是**保证循环体内的语句至少会被执行一遍**。这样才能保证return 的值`v`是我们期望的值。

> The difference between `do-while` and `while` is that `do-while` evaluates its expression at the bottom of the loop instead of the top. Therefore, the statements within the `do` block are always executed at least once.    [Java Documents](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/while.html)

## CAS实现原子操作的三大问题&解决方案

CAS 虽然高效地解决了原子操作，但是还是存在一些缺陷的，主要表现在三个方面：

- 循环时间太长
- 只能保证一个共享变量原子操作
- ABA 问题

### 循环时间过长

> 如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋 CAS 长时间地不成功，则会给 CPU 带来非常大的开销。在 J.U.C 中，有些地方就限制了 CAS 自旋的次数，例如： BlockingQueue 的 SynchronousQueue 。

### 只能保证一个共享变量原子操作

1. 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；
2. 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

### ABA问题

所谓的ABA问题就是, 一个值原先是A, 然后改为B, 后面又改为A, 之后用CAS进行检测是检测不出来的. 但是实际上是更新了两次.

解决该问题通常是在变量前面追加`版本号或者时间戳`. 从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`类来解决ABA问题。

## References

* [深入分析CAS](http://www.iocoder.cn/JUC/sike/CAS/)
* [CAS与原子操作](http://concurrent.redspider.group/article/02/10.html#第十章-乐观锁和悲观锁)

