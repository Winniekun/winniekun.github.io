---
title: Java线程之间通信方式
date: 2020-04-19 16:49:54
tags: 并发
categories: java
thumbnail:
---

## 线程通信

<!--more-->

合理的使用Java多线程可以更好地利用服务器资源。一般来讲，线程内部有自己私有的线程上下文，互不干扰。但是当我们需要多个线程之间相互协作的时候，就需要我们掌握Java线程的通信方式。本文将介绍Java线程之间的几种通信原理。

### 锁同步机制

使用`synchronized`

```java
public class lockSyn {
    static class ThreadA implements Runnable{
        @Override
        public void run() {
            synchronized (this) {
                for (int i = 0; i < 10; i++) {
                    System.out.println("ThreadA " + i);
                }
            }
        }
    }

    static class ThreadB implements Runnable{

        @Override
        public void run() {
            synchronized (this) {
                for (int i = 0; i < 10; i++) {
                    System.out.println("ThreadB " + i);
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();

    }
}
```

这段程序能够保证先执行线程A，之后再执行线程B。

### 等待/通知机制

使用`wait()/notify()`机制

```java
public class waitNotify {
    private static Object lock = new Object();
    static class A implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                for (int i = 0; i < 5; i++) {
                    System.out.println("A:" + i);
                    lock.notify();
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    static class B implements Runnable{

        @Override
        public void run() {
            synchronized (lock){
                for (int i = 0; i < 5; i++) {
                    System.out.println("B:" + i);
                    lock.notify();
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new A()).start();
        Thread.sleep(1000);
        new Thread(new B()).start();


    }
}
```

**wait():** 会释放锁。



### 信号量机制

使用volatile定义信号量(涉及JMM---Java内存模型)

```java
public class semaphore {
    private static volatile int signal = 0;

    static class ThreadA implements Runnable{

        @Override
        public void run() {
            while (signal < 10){
                if(signal % 2 == 0){
                    System.out.println("ThreadA: " + signal);
                    signal++;
                }
            }
        }
    }

    static class ThreadB implements Runnable{

        @Override
        public void run() {
            while (signal < 10){
                if(signal % 2 == 1){
                    System.out.println("ThreadB: "+ signal);
                    signal++;
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}
```

 