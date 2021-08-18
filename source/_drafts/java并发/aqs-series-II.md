---
title: aqs系列-CLH同步队列
date: 2020-05-29 10:59:10
tags: 并发
categories: java
thumbnail:
---

## 前言

<!--more-->

知道了AQS中维护了一个FIFO队列(CLH同步队列), CLH同步队列是一个FIFO双向队列，AQS依赖它来完成同步状态的管理., 如果当前线程获取同步状态失败之后, AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。 在CLH同步队列中，一个节点表示一个线程，保存信息如下: 

1. 线程的引用（thread）
2. 线程的状态（waitStatus）
3. 线程节点的前驱节点（prev）
4. 线程节点后继节点（next）



## 源码

### 数据结构

```java
static final class Node {
    /** 共享 */
    static final Node SHARED = new Node();

    /** 独占 */
    static final Node EXCLUSIVE = null;

    // waitStatus的值，表示该结点（对应的线程）已被取消
    static final int CANCELLED =  1;

    // waitStatus的值，表示后继结点（对应的线程）需要被唤醒
    static final int SIGNAL    = -1;

    // waitStatus的值，表示该结点（对应的线程）在等待某一条件
    static final int CONDITION = -2;

    //waitStatus的值，表示有资源可用，新head结点需要继续唤醒后继结点（共享模式下，多线程并发释放资
    //源，而head唤醒其后继结点后，需要把多出来的资源留给后面的结点；设置新的head结点时，会继续唤醒其
    //后继结点
    static final int PROPAGATE = -3;

    // 等待状态，取值范围，-3，-2，-1，0，1(上述的四个, 0为初始化)
    volatile int waitStatus;

    /** 前驱节点 */
    volatile Node prev;

    /** 后继节点 */
    volatile Node next;

    /** 获取同步状态的线程 */
    volatile Thread thread;

    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }
	/**
	* 前驱节点
	*/
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {
    }

    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

### 入队

就是双向队列的入队操作, 首先确定尾节点(pred), 然后将新节点(node)插入尾节点之后, 步骤如下:

1. node.pre = pred;
2. pred.next = node;

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 先尝试快速设置尾节点, 若是失败再调用enq()方法设置尾节点
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

失败情况下`enq(node)`的执行过程:

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        // 初始化的时候会涉及
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } 
        // 正常情况下
        else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

AQS通过“死循环”的方式来保证节点可以正确添加，只有成功添加后，当前线程才会从该方法返回，否则会一直执行下去。上述的两个方法均通过`compareAndSetTail`方法设置尾节点, **确保线程安全添加尾节点**

![AQS-入队](https://i.loli.net/2020/05/29/ndaMV3CON2LcwBR.png)

### 出队

CLH同步队列遵循FIFO，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点. 设置首节点的方式很简单, 也是通用的用法.

1. head指向新设置的首节点
2. 新设置的首节点的`pre`指针断开
3. 原头结点的`next`指针断开

![AQS-出队](https://i.loli.net/2020/05/29/7sHogcXKCuptMxm.png)