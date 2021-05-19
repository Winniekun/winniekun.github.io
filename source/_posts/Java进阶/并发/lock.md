---
title: lock
date: 2020-05-30 10:26:14
tags: 
	- 并发
	- 锁
categories: java
thumbnail:
---



## 简介

<!--more-->

锁用来控制多个线程的同步访问共享资源, 一个锁可以防止多个线程同时访问共享资源(读写锁允许多个线程并发访问共享资源), 在Java中,  内置锁---基于对象的锁 , 它一般是配合`synchronized`关键字使用的, 同时`synchronized`的一系列优化 , 衍生出了`偏向锁` , `轻量级锁`, `重量级锁`. 同时在`java.util.concurrent.locks`包下, 还提供了几个关于锁的接口, 抽象类及具体是实现的锁. 他们的拥有更高的灵活性和更加强大的功能.

**基础使用**

```java
Lock lock = new ReentraLock();
lock.lock();
try{
   // 业务逻辑   
}final{
    // 释放锁
    lock.unlock() 
}
```



## 锁的分类

锁可以根据以下几种方式来进行分类. 当然还有其他类型的锁类型, 如: 公平锁, 非公平锁; 偏向锁, 轻量级锁, 重量级锁; 乐观锁, 悲观锁; 

### 重入锁和非重入锁

能否重新进入锁, 也就是说锁支持一个线程对资源重复加锁.

synchronized关键字就是使用的重入锁。比如说，你在一个synchronized实例方法里面调用另一个本实例的synchronized实例方法，它可以重新进入这个锁，不会出现任何异常。



### 公平锁和非公平锁

公平和非公平的通俗意义就是`FIFO`先来后到 , 对于一个锁来说, 先对锁请求的线程会先被满足, 后对锁发起请求的线程会后被满足, 其就是公平锁

一般情况下，**非公平锁能提升一定的效率。但是非公平锁可能会发生线程饥饿（有一些线程长时间得不到锁）的情况**。所以要根据实际的需求来选择非公平锁和公平锁。

### 读写锁和排它锁

`synchronized`用的锁和`ReentrantLock` 都是排它锁,  也就是同一时刻仅允许一个线程访问资源

读写锁允许同一时刻多个线程并发访问资源, 其内部维护了一个`读锁`和`写锁` 通过分离读锁和写锁，使得在“读多写少”的环境下，大大地提高了性能。(操作系统中的读写者问题)

## juc.locks包

![locks包一览](https://i.loli.net/2020/05/30/HQlp3wbS9TuG8kA.png)

### 接口: Lock ReadWriteLock Condition

#### Lock

获取锁和释放锁的方法声明

```java
public interface Lock {
    void lock();
    void unlock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    Condition newCondition();
}
```

#### ReadWriteLock

返回读锁和写锁

```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```



#### Condition

感觉和Object类中的`wait`和`notify/notifyAll`机制很像, 不过更加高端吧

```java
public interface Condition {
    void await() throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    void awaitUninterruptibly();
    boolean awaitUntil(Date deadline) throws InterruptedException;
    void signal();
    void signalAll();
}
```

看下两者的区别吧

| 对比项                                         | Object监视器                  | Condition                                                   |
| ---------------------------------------------- | ----------------------------- | ----------------------------------------------------------- |
| 前置条件                                       | 获取对象的锁                  | 调用Lock.lock获取锁，调用Lock.newCondition获取Condition对象 |
| 调用方式                                       | 直接调用，比如object.notify() | 直接调用，比如condition.await()                             |
| 等待队列的个数                                 | 一个                          | 多个                                                        |
| 当前线程释放锁进入等待状态                     | 支持                          | 支持                                                        |
| 当前线程释放锁进入等待状态，在等待状态中不中断 | 不支持                        | 支持                                                        |
| 当前线程释放锁并进入超时等待状态               | 支持                          | 支持                                                        |
| 当前线程释放锁并进入等待状态直到将来的某个时间 | 不支持                        | 支持                                                        |
| 唤醒等待队列中的一个线程                       | 支持                          | 支持                                                        |
| 唤醒等待队列中的全部线程                       | 支持                          | 支持                                                        |



### 抽象类: AQS AQLS AOS

AQS之前已经细讲了, AQLS和AQS实现原理一样, 只不过 `state`类型改为了`long`  

AOS是AQS和AQLS的父类, 根据源码可以推测出, 其作用是表示锁和持有者(线程)的关系

![AOS](https://i.loli.net/2020/05/30/IEplk27fPmCozKs.png)

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    /** Use serial ID even though all fields transient. */
    private static final long serialVersionUID = 3737899427754241961L;

    /**
     * Empty constructor for use by subclasses.
     */
    protected AbstractOwnableSynchronizer() { }

    /**
     * 独占模式，锁的持有者  
     */
    private transient Thread exclusiveOwnerThread;

    /**
     * 设置锁持有者
     */
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    /**
     * 获取锁的持有线程
     */
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

### 具体实现类

#### ReentrantLock

之前讲过的`重入锁`和`非重入锁`以及`公平锁`和`非公平锁`在`ReentrantLock`均涉及到了. 我们可以根据源码看下具体的实现方式

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;

    abstract static class Sync extends AbstractQueuedSynchronizer {
        // 具体实现
    }

    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        // 具体实现
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        // 具体实现
    }
    
    public ReentrantLock() {
        sync = new NonfairSync();
    }

   
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    


    // Lock接口的方法实现
    ...
    // 自己一些逻辑
    ...
    public final boolean isFair() {
    	return sync instanceof FairSync;
	}
}
```

从整体上看, `ReentrantLock`有一个抽象内部类`Sync`继承AQS, 用于实现自己的`同步器`, 同时有两个非抽象内部类`NonfairSync`和`FairSync`, 继承了`Sync`, 从字面意思来看, 就是`公平同步器`和`非公平同步容器` 这意味着, `ReentrantLock`支持`公平锁`和`非公平锁`, 在构造函数里面可以传入一个`boolean`类型的参数，来指定它是否是一个公平锁，默认情况下是非公平的。这个参数一旦实例化后就不能修改，只能通过`isFair()`方法来查看。

我们来看下非公平锁实现的实现, 走一遍流程:

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;
    
    abstract void lock();
    
    // 非公平的获取资源
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        // 资源可用
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 若是当前线程已经获取了锁 可重入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    // other method
}
```

在实现非公平锁的同时, 通过增加了再次获取资源的处理逻辑实现了可重入锁: 判断当前线程是否为获取锁的线程 来判断操作是否成功, 如果是同一线程, 则`同步状态`值进行增加并返回`true` 表示获取同步成功. 那么同理, 释放锁在释放同步状态的时候, 减少同步状态值, 为`0` 则表示释放完全

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        // 全部释放的话直接把独占线程设置为null
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

非公平锁的实现, 就是后申请资源的线程和先申请资源的线程一样, 谁能通过`compareAndSetState`设置成功, 则开始执行任务.



#### ReentrantReadWriteLock

它与ReentrantLock的功能类似，同样是可重入的，支持非公平锁和公平锁。不同的是，它还支持”读写锁“。

```java
// 内部结构
private final ReentrantReadWriteLock.ReadLock readerLock;
private final ReentrantReadWriteLock.WriteLock writerLock;
final Sync sync;
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 具体实现
}
static final class NonfairSync extends Sync {
    // 具体实现
}
static final class FairSync extends Sync {
    // 具体实现
}
public static class ReadLock implements Lock, java.io.Serializable {
    private final Sync sync;
    protected ReadLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
    }
    // 具体实现
}
public static class WriteLock implements Lock, java.io.Serializable {
    private final Sync sync;
    protected WriteLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
    }
    // 具体实现
}

// 构造方法，初始化两个锁
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

// 获取读锁和写锁的方法
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

ReentrantReadWriteLock实现了读写锁，但它有一个小弊端，就是在“写”操作的时候，其它线程不能写也不能读。我们称这种现象为`写饥饿`.

#### StampedLock

`StampedLock`类是在Java 8 才发布的，也是Doug Lea大神所写，有人号称它为锁的性能之王。它没有实现Lock接口和ReadWriteLock接口，但它其实是实现了“读写锁”的功能，并且性能比ReentrantReadWriteLock更高。StampedLock还把读锁分为了“乐观读锁”和“悲观读锁”两种。

等有大段时间的时候, 再好好分析源码

## References

* Java并发编程的艺术
* [深入浅出Java多线程](http://concurrent.redspider.group/article/03/14.html)

