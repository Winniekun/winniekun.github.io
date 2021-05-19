---
title: join()、wait()、sleep()、yield()的区别
date: 2020-04-19 18:33:06
tags: 并发
categories: java
thumbnail:
---

## 前言

分开使用明白，但是合在一起就不会了，所以整理下。

<!--more-->

## join()



join()方法是Thread类的一个实例方法。它的作用是让当前线程陷入“等待”状态，等`join`的这个线程执行完成后，再继续执行当前线程。

## wait()

会释放锁

## sleep()

不会释放锁

## yield()

不会释放锁

## 区别

### wait()和sleep()的区别

- wait可以指定时间，也可以不指定；而sleep必须指定时间。
- wait释放cpu资源，同时释放锁；sleep释放cpu资源，但是不释放锁，所以易死锁。
- wait必须放在同步块或同步方法中，而sleep可以再任意位置

