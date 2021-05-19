# 通过CAS实现通用的同步框架

## 通识

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
3. 等待队列



![AQS内部数据结构](https://i.loli.net/2020/05/29/INbc5ihoatWKFD4.png)

## AQS

![image-20210416134715109](https://i.loli.net/2021/04/16/P8p1ika62sMvy7G.png)

和通识的想法类似，使用volatile修饰的state来标记共享资源的状态，因为考虑到共享和独占的模式，以及锁的可重入性，所以并未使用`boolean`进行修饰

然后使用了一个双向链表来表示一个同步队列，同步队列使用FIFO形式

1. 二叉树转双向链表
2. 二叉树转链表



### 查看Node内部细节

![image-20210416140135438](https://i.loli.net/2021/04/16/m95lIf2h6EZOBSi.png)



### 如何获取资源

获取资源分之前考虑的两种场景：

1. 尝试获取锁，如果没有获取，则立即返回
2. 获取锁，如果锁未获取到，则进入队列

#### tryAcquire()

![image-20210416141053805](https://i.loli.net/2021/04/16/ZfIQjib1BGchePp.png)

通过`tryAcquire()`方法，可以获取得到如下信息

1. protected 修饰，意思就是仅供同包下或则子类进行修饰。
2. 默认是直接抛出不支持该操作的异常，也就是说，需要我们开发的继承类，自己实现该方法

那我们接下来看下具体的同步器是如何实现的---Reentrantlock

![image-20210416144237468](https://i.loli.net/2021/04/16/iVOmfksrdQGqPJ8.png)

![image-20210416144306494](https://i.loli.net/2021/04/16/UDu56j9floysNMn.png)



#### acquire()

![image-20210416144439710](https://i.loli.net/2021/04/16/XmKnZOtPf9egMCW.png)

说明所有的继承类都可以使用该方法，但是不允许进行重写

1. 如果常熟获取锁失败，则将当前线程封装为一个Node节点，通过CAS的形式添加至队列的队尾，addWaiter方法

2. 将线程放入同步队列之后，队列是动态的，正常的思路是相当于生产者-消费者模型，生产者在队列的末尾添加等待资源的线程，然后消费者在头节点获取操作共享资源的线程，不过`acquireQueued`方法和正常的思路不太一样。

   ```java
   /**
    * Acquires in exclusive uninterruptible mode for thread already in
    * queue. Used by condition wait methods as well as acquire.
    *
    * @param node the node
    * @param arg the acquire argument
    * @return {@code true} if interrupted while waiting
    */
   final boolean acquireQueued(final Node node, int arg) {
       boolean failed = true;
       try {
           boolean interrupted = false;
           // 自选操作
           // 如果当前节点的前置节点是头节点，尝试获取锁
           // 获取锁成功，设置当前节点为头节点。
           for (;;) {
               final Node p = node.predecessor();
               if (p == head && tryAcquire(arg)) {
                   setHead(node);
                   p.next = null; // help GC
                   failed = false;
                   return interrupted;
               }
               // 绝大多数都走这里
               // 获取锁失败的线程是否需要被挂起，并且成功被挂起，interrupted返回true
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

   1. 在正常执行的过程中，final中的failed永远为false，也就不会执行`cancelAcquire`方法
   2. 执行发生异常，final块中的`cancelAcquire`方法才会执行，正常执行，不会进入该方法
      1. `cancelAcquire`方法用于将该节点中的waistatus置为CANCELLED，以及其他的清理操作。

**整理**

头节点： 一个占位摆设，第二个节点才是真正要获取锁的节点

对于必须要获取锁的线程：

1. 首先会将其封装为Node节点，然后加入同步队列，在同步队列中，首先会进行自旋来判断前驱节点是否是头节点，如果是头节点且尝试去获取锁成功，则将自身设置为头节点，然后返回。

2. 但是，大多数时候，是无法获取锁的，所以会进入被挂起的方法，那么如何判断是否需要挂起？继续看源码

   ```java
   private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
       // 获取前置节点
       int ws = pred.waitStatus;
       // 前置节点的状态为SIGNAL，
       if (ws == Node.SIGNAL)
           /*
            * This node has already set status asking a release
            * to signal it, so it can safely park.
            */
           return true;
       if (ws > 0) {
           /*
            * Predecessor was cancelled. Skip over predecessors and
            * indicate retry.
            */
           do {
               node.prev = pred = pred.prev;
           } while (pred.waitStatus > 0);
           pred.next = node;
       } else {
           /*
            * waitStatus must be 0 or PROPAGATE.  Indicate that we
            * need a signal, but don't park yet.  Caller will need to
            * retry to make sure it cannot acquire before parking.
            */
           compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
       }
       return false;
   }
   
   private final boolean parkAndCheckInterrupt() {
       LockSupport.park(this);
       return Thread.interrupted();
   }
   ```

   **总结：** 判断是否需要挂起线程， 通过前置节点的waitStatus来判断，然后执行真正的挂起操作`parkAndCheckInterrupt`

如果当前线程所在的节点处于头节点的后面一个，则会不断的去尝试获取锁，直到拿锁成功。

否则进行挂起，那么如何判断是否被挂起：

1. 如果当前线程所在的节点之前，除了HEAD还有其他节点，并且waitstatus为signal，则需要被挂起。这样就能保证只有一个线程通过CAS的方式来获取锁，队列里面其他线程处于正在被挂起或则已经被挂起。

   ![image-20210416154606709](https://i.loli.net/2021/04/16/blsILjyYDdphvgo.png)

### 如何释放资源

自定实现tryacquire()方法，然后调用releas方法，

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
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

将获取锁的线程的waitsatus置为0，接下来从后往前，找到除了head之外，并且最靠前的节点（waitstatus<0）的节点，将其唤醒。

为什么从后往前

> 后面从后往前遍历是因为插入的时候，cas和next节点赋值的时候可能会有其他线程打断，导致从前往后遍历会出现null。可以去看看enq方法就知道了