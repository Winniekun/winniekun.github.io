---
title: countdownlatch个人理解
mathjax: false
tags: 
- juc
- 并发
categories: java
---

## 序列

看完AQS之后，趁热打铁，开始霍霍JUC下的各种实现。今天轮到了同步工具---`CountDownLatch`

## 作用

CountDownLatch最头上的注释是这样子的：

> A synchronization aid that allows one or more threads to wait until
> a set of operations being performed in other threads completes.

CountDownLatch这个同步工具允许一条或者多条线程等待其他线程的一组操作完成后，再继续执行。

### **举例说明：**

定义的稍微又些抽象，所以再举几个例子来实际感受下。

#### 最直白的方式

主线程等待几个子线程收集龙珠，所有子线程龙珠收集结束之后，主线程再继续最后的任务---召唤神龙

```java
@Slf4j
public class CountDownLatchDemo {
    private final static Random R = new Random();

    public static class SearchTask implements Runnable {
        private int id;
        private CountDownLatch latch;

        public SearchTask(int id, CountDownLatch latch) {
            this.id = id;
            this.latch = latch;
        }

        @Override
        public void run() {
            System.out.println("开始寻找" + id + "号龙珠");
            int seconds = R.nextInt(10);
            long now = System.currentTimeMillis();
            try {
                Thread.sleep(seconds * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long end = System.currentTimeMillis();
            System.out.println("消耗 " + (end - now) / 1000 + "s, 找到了" + id + "号龙珠");
            latch.countDown();
        }
    }

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
        CountDownLatch latch = new CountDownLatch(list.size());
        for (Integer id : list) {
            Thread thread = new Thread(new SearchTask(id, latch));
            thread.start();
        }
        try {
            latch.await();
        } catch (InterruptedException e) {
            log.info("error happened ", e);
        }
        System.out.println("所有龙珠找到，召唤神龙");
    }
}
```

#### 稍微复杂的应用

两个线程池，一个线程池中的线程等待另一线程池的线程执行完，再继续执行剩余的业务。

```java
@Slf4j
public class CountDownLatchDemoII {
    public final static Random R = new Random();
    public static CountDownLatch countDown = new CountDownLatch(2);

    public static void main(String[] args) {
        ThreadPoolExecutor countdown = ThreadUtils.poolExecutor(2, 2);
        ThreadPoolExecutor await = ThreadUtils.poolExecutor(2, 2);
        log.info("两个线程池开始执行任务");
        for (int i = 0; i < 2; i++) {
            await.execute(CountDownLatchDemoII::awaitHandler);
        }
        for (int i = 0; i < 2; i++) {
            int finalI = i;
            countdown.execute(() -> countdownHandler(finalI));
        }
        countdown.shutdown();
        await.shutdown();
    }

    public static void countdownHandler(int i) {
        try {
            int seconds = R.nextInt(5);
            Thread.sleep(seconds * 1000);
            log.info("第 [{}] 个子任务执行结束", i);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 随机休息之后，执行countdown
        countDown.countDown();
    }

    public static void awaitHandler() {
        try {
            countDown.await();
            log.info("线程 [{}] 通过调用await，继续执行后续任务", Thread.currentThread().getName());
        } catch (InterruptedException e) {
            log.info("线程 [{}] 被异常中断 [{}]", Thread.currentThread().getName(), e);
        }
    }
}
```

#### 再复杂的应用

运动员跑步，听到裁判发号之后，再开始跑步，然后裁判等待所有运动员跑完，统计从发号到现在耗时多久

```java
@Slf4j
public class CountDownLatchDemoIII {
    // 1. 听到枪声，所有运动员开跑
    // 2. 所有运动员到达之后，比赛结束，统计时间
    private static CountDownLatch start = new CountDownLatch(8);
    private static CountDownLatch trumpet = new CountDownLatch(1);
    private static Random R = new Random();

    public static void main(String[] args) {
        ThreadPoolExecutor executor = ThreadUtils.poolExecutor(8, 8, "runner-pool");
        // 所有运动员进入预备状态
        for (int i = 0; i < 8; i++) {
            int finalI = i;
            executor.execute(()-> runHandler(finalI));
        }
        //开始，等全部跑完，统计最长时间
        try {
            long time = starter();
            log.info("全部跑完，共耗时: [{}]", time);
        } catch (InterruptedException e) {
            log.error("error : [{}]", e.getMessage());
        }
    }

    public static void runHandler(int runner) {
        long now = System.currentTimeMillis();
        try {
            //收到吹号，开始跑步
            trumpet.await();
            log.info("第 [{}] 号成员开跑", runner);
            int random = R.nextInt(10);
            Thread.sleep(random * 10);
        } catch (InterruptedException e) {
            log.info("error");
        }
        log.info("第 [{}] 号成员跑完，耗时: [{}]", runner, System.currentTimeMillis() - now);
        start.countDown();
    }
		
    // 发号和统计市场，放在同一个方法里
    public static long starter() throws InterruptedException {
        trumpet.countDown();
        long now = System.currentTimeMillis();
        start.await();
        long end = System.currentTimeMillis();
        return end - now;
    }
}
```

根据上述的几个例子，绘制一个形象化的图示

![CountDownLatch](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210531202947069.png)

## 源码解读

对于CountDownLatch一个核心的问题就是，主任务通过什么方式确定子任务已经全部执行完毕了。带着这个狠心的问题，我们就可以开心的去看源码了

折叠了注释之后，其实`CountDownlatch`的内容并不多。最为核心的就是基于AQS自定义的同步器，因为其他公共方法都是基于这个同步器进行实现的。

![image-20210531205329765](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210531205329765.png)

### Sync

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;
    Sync(int count) {
        setState(count);  
    }
    int getCount() {
        return getState();
    }
    
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

接下来，就结合主任务的调用和子任务的调用，深入理解如何实现主任务和子任务之间的通讯机制，使得主任务能够在所有子任务执行结束后继续执行。因为内部的sync还是调用了大量的父类AQS的final方法，为了避免频繁的切换，调用链先展示如下图。

> 截自 [寒食君](https://www.bilibili.com/video/BV1Uy4y117S6)

![countdownlatch调用时序](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210531151231079.png)

### 主任务的await流程

![CountDownLatch-await](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/CountDownLatch-await.png)

```java
// CountDownLatch
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
// 返回正数  获取锁成功，唤醒后继节点
// 返回0    获取锁成功，不唤醒后继节点
// 返回负数  获取锁失败
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}

// AQS
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                // 只要 state 不等于 0，那么这个方法返回 -1
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

1. 调用`await`

2. 调用AQS中的`acquireSharedInterruptibly`

3. 调用自定义同步器中实现的`tryAcquireShared`

   1. 这里因为初始化的时候已经定义了有多少个子任务需要进行了，也就是state的值
   2. 在state==0时，说明子任务已经全部执行结束
   3. 这时已经没有其他子任务，主任务被唤醒，主任务也没必要通过cas获取锁了， 多此一举
   4. 返回1，是因为需要唤醒后值的共享模式的节点，因为主任务个数》1

4. 调用AQS中的`doAcquireSharedInterruptibly`

   获取失败，也就是子任务还没有完全结束

   1. `addWaiter`入队

   2. 死循环

      只要子任务没有执行完，所有的主任务被阻塞，等待唤醒

### 子任务的release流程

```java
// sync
public void countDown() {
    sync.releaseShared(1);
}
// aqs
public final boolean releaseShared(int arg) {
    // 只有当 state 减为 0 的时候，tryReleaseShared 才返回 true
    // 否则只是简单的 state = state - 1 那么 countDown() 方法就结束了
    // 将 state 减到 0 的那个操作才是最复杂的
    if (tryReleaseShared(arg)) {
        // 唤醒 await 的线程
        doReleaseShared();
        return true;
    }
    return false;
}
// sync
protected boolean tryReleaseShared(int releases) {
    // 自旋的方式实现state-1
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
// aqs
// 太难顶了，看不得晕晕的
private void doReleaseShared() {
    
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
// aqs
private void unparkSuccessor(Node node) {
    
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    
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

## 总结

1. CountDownLatch基于AQS共享锁实现，在创建时会根据传入的`count`参数将AQS提供的`state`的值直接置为`count`的值。
2. 当线程调用CountDownLatch实例的`await()`方法，会执行抢锁操作，判断是否抢锁成功的依据是`state`是否为0，如果不为0则代表抢锁失败，直接将线程包装成节点装入同步队列进行等待。
3. 每次调用CountDownLatch实例的`countDown()`，都会使`state`的值减1，如果减1之后`state`的值减为0时，就会唤醒同步队列中阻塞的线程，由于是共享模式，这些线程会依次被传播唤醒继续执行。