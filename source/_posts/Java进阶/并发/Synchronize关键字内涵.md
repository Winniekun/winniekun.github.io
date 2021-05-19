---
title: Synchronize关键字内涵
date: 2020-01-18 12:35:12
tags: 并发
categories: java
thumbnail: https://i.loli.net/2020/03/11/fv1JyPacVRzM4ZO.jpg
---

## 内涵与表象

<!--more-->

synchronized关键字的解释如下：

> Java语言的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

根据定义，会考虑两个方面的问题：

1. synchronized是如何**保证在同一时刻最多只有一个线程执行该段代码**
2. **保证在同一时刻最多只有一个线程执行该段代码**，这又带来什么意义

老司机级别的解释：

> Java中的synchronize，通过使用**内置锁**，来实现对变量的同步操作，进而实现了对**变量**操作的**原子性**和**其他线程**对变量的**可见性**，从而确保了并发情况下的线程安全。

## 使用

[Synchronized关键字](http://concurrent.redspider.group/article/02/9.html)

## 原子性

synchronized关键字的一个作用就是**确保了原子性**。

譬如使用`count++`而言，原先其执行需要三个步骤：

1. 读取count值
2. count+1
3. 写入count 值

其包含了三个步骤，且每次的步骤都依赖于上一次的结果。所以不是原子性的

```java
synchronized(this){
    count++;
}
```

使用如上的方式，就确保了原子性。因为原子操作是线程安全的，这其实也是我们经常使用synchronize来实现线程安全的原因。

## 可见性

synchronize的确保原子性，其实是从**使用synchronize的线程**的角度来讲的，而如果我们从**其他线程**的角度来看，那么synchronize则是实现了**可见性**。可见性的意思就是：**变量的修改，可以被其他的线程看到**。其类似于MySQL中隔离性的读提交隔离级别。

如count++， 抢不到锁的线程，等拥有锁的线程执行完之后，才能拥有锁，然后去执行count++，这个时候，新拥有的锁的线程在更改count的时候，使用的count的初始值是上一个线程更新之后的结果。

### synchronized和volatile的区别

其实两者的本质区别就是synchronized关键字既能确保**原子性**，又能确保**可见性**，而volatile关键字只确保了**可见性**，其是一种更轻量级的同步机制。

## 内置锁

关于synchronize，我们经常使用的隐喻就是锁，首先进入的线程，拿到了锁的唯一一把钥匙，至于其他线程，就只能阻塞（Blocked）；等到线程走出synchronize之后，会把锁释放掉，也就是把钥匙扔出去，下一个拿到钥匙的线程，就可以结束阻塞状态，继续运行。
但是锁从哪来呢？随随便便抓起一个东西就可以作为锁么？
还真是这样，**Java中每一个对象都有一个与之关联的锁**，称为**内置锁**

> Every object has an intrinsic lock associated with it. —— [The Java™ Tutorials](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html)

1. 当我们使用synchronize修饰非静态方法时，用的是调用该方法的实例的内置锁，也就是**this**;
2. 当我们使用synchronize修饰静态方法时，用的是调用该方法的所在的**Class对象**的内置锁;
3. 当我们使用的是synchronize代码块，我们经常用的是synchronize(this)，也就是把对象实例作为锁。

Java SE 1.6为了减少获得锁和释放锁带来的性能消耗,引入了`偏向锁`和`轻量级锁`,在Java SE 1.6中,锁一共有4种状态,级别从低到高依次是:`无锁状态`、`偏向锁状态`、`轻量级锁状态`和`重量级锁状态`,这几个状态会随着竞争情况逐渐升级.

### 偏向锁



### 轻量锁

### 重量锁



## 可重入：

```java
public class Widget{
    public synchronized void doSomething(){
        ...
    }
}

public class LoggingWidget extends Widget {
    public synchronized void doSomething(){
        sout("...");
        super.doSomething();
    }
}
```

如果没有可重入性，那么该代码会产生死锁。

### 可重入的定义：

> 若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant或re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。 --- [维基百科](https://zh.wikipedia.org/wiki/可重入)

从设计上讲，当一个线程请求一个由其他线程持有的对象锁时，该线程会阻塞。当线程请求自己持有的对象锁时，如果该线程是重入锁，请求就会成功，否则阻塞。

synchronized拥有强制原子性的内部锁机制，是一个可重入锁。因此，在一个线程使用synchronized方法时调用该对象另一个synchronized方法或调用父类的synchronized方法，即一个线程得到一个对象锁后再次请求该对象锁，是**永远可以拿到锁的**。

在Java内部，同一个线程调用自己类中其他synchronized方法/块时不会阻碍该线程的执行，同一个线程对同一个对象锁是可重入的，同一个线程可以获取同一把锁多次，也就是可以多次重入。**原因是Java中线程获得对象锁的操作是以线程为单位的，而不是以调用为单位的。**



## 总结

1. Java中每个对象都有一个**内置锁**。
2. synchronized使用对象自带的内置锁来进行加锁，从而保证在同一时刻最多只有一个线程执行代码。
3. 所有的加锁行为，都可以带来两个保障——**原子性**和**可见性**。其中，原子性是相对锁所在的线程的角度而言，而可见性则是相对其他线程而言。
4. **锁的持有者是`线程`**，而不是`调用`，这也是锁的为什么是**可重入**的原因。

## References

* Java并发编程实战
* [深入浅出Java多线程](http://concurrent.redspider.group/RedSpider.html)
* [Java多线程：synchronized的可重入性](https://www.cnblogs.com/cielosun/p/6684775.html)