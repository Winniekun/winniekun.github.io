---
title: Java并发基础
date: 2019-12-20 13:12:14
tags: 并发
categories: java
thumbnail: https://i.loli.net/2020/03/11/fv1JyPacVRzM4ZO.jpg
---

## 操作系统中的进程、线程

<!--more-->

### 进程概念：

进程的引入是为了资源利用率和系统吞吐量

不同的角度，进程的定义不同。

1. 进程是程序的一次执行过程
2. 进程是一个程序及其数据在处理机上顺序执行时所发生的活动
3. **进程是具有独立功能的程序在一个数据集上运行的过程，是系统进行资源分配和调度独立单元**

### 线程概念：

线程的引入是为了减小程序在并发执行时所付出的时空开销，提高操作系统的并发性能

最直接的理解为: **轻量级线程**



## Java实现多线程

### 继承Thread类

```java
public class ExtendsThread extends Thread {

    @Override
    public void run() {
        // run 方法
        for (int i = 0; i < 20; i++) {
            System.out.println("run方法" + (i+1));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // 创建一个线程对象
        ExtendsThread thread = new ExtendsThread();
        // 调用start()方法 开启线程
        thread.start();

        // main线程 主线程
        for (int i = 0; i < 200; i++) {
            System.out.println("main方法" + (i+1));
        }
    }
}
```



### 实现Runnable接口

```java
public class ImplementsThread implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println("run方法: " + i);
        }
    }

    public static void main(String[] args) {
        ImplementsThread myThread = new ImplementsThread();
        Thread thread = new Thread(myThread);
        thread.start();

    }
}
```



## 线程的状态转换

### OS中的线程状态转换

> 在现代的OS中，线程被视为轻量级进程，所有其状态转换和进程一致

![系统进程状态转换图.png](https://i.loli.net/2020/04/03/J5RLXSjhAuxZMU1.png)

### Java线程的6个状态

```java
public enum State {
    /**
     * 
     */
    NEW,
    /**
	 * 
     */
    RUNNABLE,
    /**
     * 
     */
    BLOCKED,
    /**
     * 
     */
    WAITING,
    /**
     * 
     */
    TIMED_WAITING,
    /**
     * 
     */
    TERMINATED;
}
```

#### new

为线程还没有`start`时的状态，也就是该线程尚未启动。

```java
Thread thread = new Thread(()-> {});
System.out.println(thread.getState()); // new
```

#### runnable

线程的运行状态，包含了操作系统中的`ready`和`running`状态。

`Thread`源码对其解释

```tex
/**
 * Thread state for a runnable thread.  A thread in the runnable
 * state is executing in the Java virtual machine but it may
 * be waiting for other resources from the operating system
 * such as processor.
 */
```

#### blocked

阻塞状态，无法访问共享资源，等待其他线程释放资源，然后访问。

处于BLOCKED状态的线程正等待锁的释放以进入同步区

形象理解---以下摘自[深入浅出Java多线程](http://concurrent.redspider.group/)

> 假如今天你下班后准备去食堂吃饭。你来到食堂仅有的一个窗口，发现前面已经有个人在窗口前了，此时你必须得等前面的人从窗口离开才行。
> 假设你是线程t2，你前面的那个人是线程t1。此时t1占有了锁（食堂唯一的窗口），t2正在等待锁的释放，所以此时t2就处于BLOCKED状态。

#### waiting

等待状态。等待其他的线程的唤醒才能进入`runnable`

如下三个方法会进入`waiting`状态：

1. Object.wait(): 当前线程处于等待状态，直到另一个线程唤醒它
2. Thread.join(): 当前线程等待调用join()方法的线程结束后才能继续执行
3. LockSupport.park(): 除非获得调用许可，否则禁用当前线程进行线程调度。

形象理解---以下摘自[深入浅出Java多线程](http://concurrent.redspider.group/)

> 你等了好几分钟现在终于轮到你了，突然你们有一个“不懂事”的经理突然来了。你看到他你就有一种不祥的预感，果然，他是来找你的。
>
> 他把你拉到一旁叫你待会儿再吃饭，说他下午要去作报告，赶紧来找你了解一下项目的情况。你心里虽然有一万个不愿意但是你还是从食堂窗口走开了。
>
> 此时，假设你还是线程t2，你的经理是线程t1。虽然你此时都占有锁（窗口）了，“不速之客”来了你还是得释放掉锁。此时你t2的状态就是WAITING。然后经理t1获得锁，进入RUNNABLE状态。
>
> 要是经理t1不主动唤醒你t2（notify、notifyAll..），可以说你t2只能一直等待了。

#### timed_waiting

超时等待状态。线程等待具体的时间， 时间到了之后，自动运行。

调用如下方法会进入超时等待状态

- Thread.sleep(long millis)：使当前线程睡眠指定时间；
- Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
- Thread.join(long millis)：等待当前线程最多执行millis毫秒，如果millis为0，则会一直执行；
- LockSupport.parkNanos(long nanos)： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；
- LockSupport.parkUntil(long deadline)：同上，也是禁止线程进行调度指定时间；

形象理解---以下摘自[深入浅出Java多线程](http://concurrent.redspider.group/)

> 到了第二天中午，又到了饭点，你还是到了窗口前。
>
> 突然间想起你的同事叫你等他一起，他说让你等他十分钟他改个bug。
>
> 好吧，你说那你就等等吧，你就离开了窗口。很快十分钟过去了，你见他还没来，你想都等了这么久了还不来，那你还是先去吃饭好了。
>
> 这时你还是线程t1，你改bug的同事是线程t2。t2让t1等待了指定时间，t1先主动释放了锁。此时t1等待期间就属于TIMED_WATING状态。
>
> t1等待10分钟后，就自动唤醒，拥有了去争夺锁的资格。

#### terminated

线程运行结束

### Java中的线程状态转换

![线程状态转换图.png](https://i.loli.net/2020/04/03/18iASf6ChmWBtlb.png)

### BLOCKED和RUNNABLE之间的转换

```java
@Test
public void blockedTest() throws InterruptedException {
    Thread a = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod();
        }
    }, "a");
    Thread b = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod();
        }
    }, "b");
    a.start();
    // Thread.sleep(1000);
    b.start();
    System.out.println(a.getName() + " " + a.getState());
    System.out.println(b.getName() + " " + b.getState());
}
private synchronized void testMethod(){
    try{
        Thread.sleep(2000);
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

创建两个线程，然后使用同步方法争夺锁，使用`testMethod`

`main`线程依步执行， 按照推测来讲，输出结果应该如下(a先调用同步方法，然后a又调用了sleep()， 输出`TIMED_WAITING`， 接下来b会执行，然后因为a还没有释放锁，所以b输出`BLOCKED`)

```
a TIMED_WAITING
b BLOCKED
```

但是输出结果并不确定，也可能是如下(a线程已经执行到了run方法，而b线程才刚开始，还未去调用run方法)

```
a TIMED_WAITING
b RUNNABLE
```

`main`线程只保证了`a`，`b`两个线程调用`start()`方法（转化为`RUNNABLE`状态），还没等两个线程真正开始争夺锁，就已经打印此时两个线程的状态（`RUNNABLE`）了。

```
a RUNNABLE
b RUNNABLE
```

可以在主线程`main`的`a.start()` 和 `b.start()`之间让其休眠一段时间

### WAITING和RUNNABLE之间的转换

根据转换图我们知道有3个方法可以使线程从RUNNABLE状态转为WAITING状态。我们主要介绍下**Object.wait()**和**Thread.join()**

**Object.wait()**

> 调用wait()方法前线程必须持有对象的锁。
>
> 线程调用wait()方法时，会释放当前的锁，直到有其他线程调用notify()/notifyAll()方法唤醒等待锁的线程。

**Thread.join()**

> 调用join()方法不会释放锁，会一直等待当前线程执行完毕（转换为TERMINATED状态）。

### TIMED_WAITING和RUNNABLE之间的转换



## 线程的中断

> Java中目前还没有安全直接的方法来停止线程

Thread类里提供的关于线程中断的几个方法：

- Thread.interrupt()：中断线程。这里的中断线程并不会立即停止线程，而是设置线程的中断状态为true（默认是flase）；
- Thread.interrupted()：测试当前线程是否被中断。线程的中断状态受这个方法的影响，意思是调用一次使线程中断状态设置为true，连续调用两次会使得这个线程的中断状态重新转为false；
- Thread.isInterrupted()：判断当前线程是否被中断。与上面方法不同的是调用这个方法并不会影响线程的中断状态。





## 操作系统中的进程（线程）状态和Java中的线程状态关系

从实际意义上来讲，操作系统中的线程除去`new`和`terminated`状态，一个线程真实存在的状态，只有：

1. ready：表示线程已经被创建，正在等待系统调度分配CPU使用权。
2. running：表示线程获得了CPU使用权，正在进行运算
3. waiting：表示线程等待（或者说挂起），让出CPU资源给其他线程使用

为什么除去new和terminated状态？是因为这两种状态实际上并不存在于线程运行中，所以也没什么实际讨论的意义。

对于Java中的线程状态：

无论是`Timed Waiting`，`Waiting`还是`Blocked`，对应的都是操作系统线程的`waiting`状态。
而`Runnable`状态，则对应了操作系统中的`ready`和`running`状态。

而对不同的操作系统，由于本身设计思路不一样，对于线程的设计也存在种种差异，所以JVM在设计上，就已经声明：

```java
虚拟机中的线程状态，不反应任何操作系统线程状态
```



## 后续的Java并发套路

在《Java并发编程实践》中，将委托称为**实现线程安全的一个最有效策略**。委托其实就是把原来通过使用synchronize和volatile来实现的线程安全，委托给Java提供的一些基础构建模块（当然你也可以写自己的基础构建模块）去实现，这些基础构建模块包括：

- **同步容器**：比如Vector，同步容器的原理大多就是synchronize，所以用的不多；
- **并发容器**：比如本文用到的ConcurrentHashMap，当然还有CopyonWriteArrayList、LinkedBlockingQueue等，根据你的需求使用不同数据结构的并发容器；
- **同步工具类**：比如本文用到的Future和FutureTask，还有闭锁（Latches）、信号量（Semaphores）、栅栏（Barriers）等

这些基础构建模块，再加上synchronize和volatile，就形成了**Java并发的基础知识**。



## References

* [深入浅出Java多线程](http://concurrent.redspider.group/)
* [Java线程和操作系统线程的关系](https://blog.csdn.net/CringKong/article/details/79994511)
* Java并发编程实战



