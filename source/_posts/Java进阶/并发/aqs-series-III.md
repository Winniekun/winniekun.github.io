---
title: aqs系列-主要方法源码解析
date: 2020-05-29 19:36:52
tags: 并发
categories: java
thumbnail:
---

## 前言

<!--more-->

之前的两篇文章有涉及到资源共享的模式, 如CLH同步队列中的`Node`静态内部类的定义中的两个参数

```java
/** 共享 */
static final Node SHARED = new Node();

/** 独占 */
static final Node EXCLUSIVE = null;
```

这两个资源共享模式的具体解释如下:

1. 独占模式(Exclusive)
   1. 资源是独占的, 一次只能有一个线程获取该资源(ReentrantLock)
2. 共享模式(Shared)
   1. 同时可以被多个线程获取, 具体的资源数量通过参数指定(Semaphore)

## AQS主要方法解析

AQS的设计是基于**模板方法模式**的，它有一些方法必须要子类去实现的，它们主要有：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。



自己实现一个独占锁

```java
public class Mutex implements Lock {

    // 静态内部类 自定义同步容器
    private static class Sync extends AbstractQueuedSynchronizer {
        // 是否处于占用状态
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 状态为0的时候获取锁
        @Override
        protected boolean tryAcquire(int arg) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // 释放锁 将状态设置为0
        @Override
        protected boolean tryRelease(int arg) {
            if (getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
        // 返回一个Condition
        Condition newCondition(){
            return new ConditionObject();
        }

    }
    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```





### 获取资源(同步状态)

#### 入口: acquire()

获取资源的入口是acquire(int arg)方法。arg是要获取的资源的个数，在独占模式下始终为1。我们先来看看这个方法的逻辑：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

1. 首先会调用自定义的同步器实现的`tryAcqure`方法获取资源, 
2. 若是获取失败, 则创建独占式的同步节点, 并且通过`addWaiter()`方法将该节点添加到CLH同步队列尾部,该方法之前讲过. 
3. 最后调用`acquireQueued()` 方法, 使得该节点以死循环的方式获取资源

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 死循环方式获取资源
        for (;;) {
            // 获取该节点的前驱节点
            final Node p = node.predecessor();
            // 如果 前驱结点为head节点(独占模式中获取资源的线程)且当前的节点尝试获取资源成功
            // 等同于超市排队结账, 正在结账的后面一个顾客要观察正在结账的人
            // 是否结账完成, 之后就轮到他去结账了.
            if (p == head && tryAcquire(arg)) {
                // 将head指向该节点
                // head所指的结点，就是当前获取到资源的那个结点或null。
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 如果自己可以休息了，就进入waiting状态，直到被unpark()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

> 这里parkAndCheckInterrupt方法内部使用到了LockSupport.park(this)
>
> LockSupport类是Java 6 引入的一个类，提供了基本的线程同步原语。LockSupport实际上是调用了Unsafe类里的函数，归结到Unsafe里，只有两个函数：
>
> - park(boolean isAbsolute, long time)：阻塞当前线程
> - unpark(Thread jthread)：使给定的线程停止阻塞

所以**结点进入等待队列后，是调用park使它进入阻塞状态(Waiting状态)的。只有头结点的线程是处于活跃状态的**。

![AQS获取资源流程图](https://i.loli.net/2020/05/29/FTyQ7dnxHhRwIk6.png)

### 释放资源

相比而言, 释放资源就显的简单一些

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    // 如果状态是负数，尝试把它设置为0
    int ws = node.waitStatus;
    if (ws < 0)
        // cas 保证了原子操作
        compareAndSetWaitStatus(node, ws, 0);
    
    // 头结点的后继节点
    Node s = node.next;
    // waitStatus>0 表示后继节点被取消 无法获取资源
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从尾部开始向前遍历, 然后唤醒 获取资源执行任务
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```



