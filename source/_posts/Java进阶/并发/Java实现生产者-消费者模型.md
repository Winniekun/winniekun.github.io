---
title: Java实现生产者-消费者模型
date: 2020-04-21 21:27:00
tags: 并发
categories: java
thumbnail:
---

## 前言

<!--more-->

最近在看线程池的实现原理，后面又看了源码，发现其内部是一个`生产者-消费者`模型，用户是提交的任务（task）是生产者，线程池中的线程（worker）是消费者。

然后感觉对`生产者-消费者`模型的实现，磕磕绊绊的，所以记录一下。下面会涉及多种生产者-消费者模型的实现，可以先抽象出关键的接口，并实现一些抽象类：

## 准备

### 生产者消费者接口

#### **生产者接口**

```java
public interface Producer {
    void produce() throws InterruptedException;
}
```

#### **消费者接口**

```java
public interface Consumer {
    void consume() throws InterruptedException;
}
```

### 生产者消费者抽象类

#### **生产者抽象类**

```java
abstract class AbstractProducer implements Runnable, Producer {
    @Override
    public void run() {
        while (true){
            try{
                produce();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

#### **消费者抽象类**

```java
abstract class AbstractConsumer implements Consumer, Runnable {
    @Override
    public void run() {
        while (true){
            try{
                consume();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

### Model接口

对应的实现`生产者-消费者`模型，不同的`生产者-消费者`实现模型都实现该接口，

```java
public interface Model {
    Runnable newRunnableConsumer();
    Runnable newRunnableProducer();
}
```

### Task

将Task作为生产和消费的基本单位

```java
public class Task {
    private int no;

    public Task(int no){
        this.no = no;

    }

    public int getNo() {
        return no;
    }

    public void setNo(int no) {
        this.no = no;
    }
}
```



## 实现一：BlockingQueue

BlockingQueue的写法最简单。核心思想是，**把并发和容量控制封装在缓冲区中**。而BlockingQueue的性质天生满足这个要求。

```java
package com.train.concurrent.procon;


import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Time: 2020/4/21下午8:54
 * @Author: kongwiki
 * @Email: kongwiki@163.com
 */
public class BlockingQueueModel implements Model {
    private BlockingQueue<Task> queue;
    private final AtomicInteger increTaskNo = new AtomicInteger(0);

    public BlockingQueueModel(int cap){
        this.queue = new LinkedBlockingDeque<>(cap);
    }


    @Override
    public Runnable newRunnableConsumer() {
        return new ConsumerImpl();
    }

    @Override
    public Runnable newRunnableProducer() {
        return new ProducerImpl();
    }

    private class ConsumerImpl extends AbstractConsumer implements Runnable, Consumer{

        @Override
        public void consume() throws InterruptedException {
            Task task = queue.take();
            Thread.sleep(500 + (long) (Math.random() * 500));
            System.out.println("consume: " + task.getNo());

        }
    }

    private class ProducerImpl extends AbstractProducer implements Runnable, Producer{
        @Override
        public void produce() throws InterruptedException {
            Thread.sleep((long) (500 + (Math.random()) * 1000));
            Task task = new Task(increTaskNo.getAndIncrement());
            System.out.println("produce: " + task.getNo());
            queue.put(task);
        }
    }

    public static void main(String[] args) {
        Model model = new BlockingQueueModel(3);
        for (int i = 0; i < 2; i++) {
            new Thread(model.newRunnableConsumer()).start();
        }

        for (int i = 0; i < 5; i++) {
            new Thread(model.newRunnableProducer()).start();
        }
    }
}

```



## 实现二：wait&notify

```java
package com.train.concurrent.procon;


import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Time: 2020/4/22上午7:40
 * @Author: kongwiki
 * @Email: kongwiki@163.com
 */
public class WaitNotifyModel implements Model {
    private int integer;
    private final Object buffer_lock = new Object();
    private final Queue<Task> buffer = new LinkedList<>();
    private int cap;

    public WaitNotifyModel(int cap){
        this.cap = cap;
    }

    @Override
    public Runnable newRunnableConsumer() {
        return new ConsumerImpl();
    }

    @Override
    public Runnable newRunnableProducer() {
        return new ProducerImpl();
    }

    public class ProducerImpl extends AbstractProducer implements Runnable, Producer{

        @Override
        public void produce() throws InterruptedException {
            Thread.sleep((long) (500 + (Math.random() * 1000)));
            synchronized (buffer_lock){
                while (buffer.size() == cap){
                    buffer_lock.wait();
                }
                Task task = new Task(integer++);
                buffer.add(task);
                System.out.println("produce: " + task.getNo());
                buffer_lock.notifyAll();
            }
        }
    }

    public class ConsumerImpl extends AbstractConsumer implements Runnable, Consumer{

        @Override
        public void consume() throws InterruptedException {
            synchronized (buffer_lock){
                while (buffer.size() == 0){
                    buffer_lock.wait();
                }
                Task task = buffer.poll();
                assert task != null;
                Thread.sleep((long) (500 + (Math.random() * 1000)));
                System.out.println("consume: " + task.getNo());
                buffer_lock.notifyAll();

            }
        }
    }

    public static void main(String[] args) {
        Model model = new WaitNotifyModel(3);
        for (int i = 0; i < 3; i++) {
            new Thread(model.newRunnableConsumer()).start();
        }

        for (int i = 0; i < 5; i++) {
            new Thread(model.newRunnableProducer()).start();
        }
    }
 }

```



## 实现三：lock && condition

```java
package com.train.concurrent.procon;

import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Time: 2020/4/22上午8:01
 * @Author: kongwiki
 * @Email: kongwiki@163.com
 */
public class LockConditionModel implements Model {
    private final Lock BUFFER_LOCK = new ReentrantLock();
    private final Condition BUFFER_COND = BUFFER_LOCK.newCondition();
    private int integer;
    private Queue<Task> buffer = new LinkedList<>();
    private int cap;

    public LockConditionModel(int cap){
        this.cap = cap;
    }

    @Override
    public Runnable newRunnableConsumer() {
        return new ConsumerImpl();
    }

    @Override
    public Runnable newRunnableProducer() {
        return new ProducerImpl();
    }

    public class ProducerImpl extends AbstractProducer implements Producer, Runnable{

        @Override
        public void produce() throws InterruptedException {
            Thread.sleep((long) (500 + (Math.random() * 1000)));
            BUFFER_LOCK.lockInterruptibly();
            try{
                while (buffer.size() == cap){
                    BUFFER_COND.await();
                }
                Task task = new Task(integer++);
                buffer.offer(task);
                System.out.println("produce: "+ task.getNo());
                BUFFER_COND.signalAll();
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                BUFFER_LOCK.unlock();
            }
        }
    }

    public class ConsumerImpl extends AbstractConsumer implements Consumer, Runnable{

        @Override
        public void consume() throws InterruptedException {
            BUFFER_LOCK.lockInterruptibly();
            try{
                while (buffer.size() == 0){
                    BUFFER_COND.await();
                }
                Task task = buffer.poll();
                assert task != null;
                System.out.println("consume: " + task.getNo());
                BUFFER_COND.signalAll();
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                BUFFER_LOCK.unlock();
            }
        }
    }

    public static void main(String[] args) {
        Model model = new LockConditionModel(3);
        for (int i = 0; i < 3; i++) {
            new Thread(model.newRunnableConsumer()).start();
        }

        for (int i = 0; i < 5; i++) {
            new Thread(model.newRunnableProducer()).start();
        }
    }
}

```

