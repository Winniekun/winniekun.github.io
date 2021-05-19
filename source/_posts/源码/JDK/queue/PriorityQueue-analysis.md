---
title: PriorityQueue源码解读
date: 2020-08-21 20:00:56
tags: 
	- JDK源码
	- Queue
categories: 源码
thumbnail: Pr 源码解读
---

## PriorityQueue源码解读

<!--more-->

优先队列, 顾名思义就是按照元素的优先级进行弹出等操作, 那么在JDK中使用何种数据结构来实现优先队列嘞, Let's Go

## 依赖

![PriorityQueue](https://i.loli.net/2020/08/21/QOjliBWZwrHo72m.png)

emmm, 感觉从依赖上来看, 和其他的集合类一样, 实现其对应的接口(Marker Interface的作用?)

## 基础

在阅读基础的时候，我们需要先理解一下什么是堆，其数据结构是啥，有哪些存储方式。

1. 堆的底层数据结构为**完全二叉树**
2. 存储方式（既然是二叉树，那么其存储方式主要分为如下两种）
   1. 顺序存储结构
      1. 根节点下标为0
      2. 若节点p的下标为$i$，则左孩子$2\cdot i + 1$ 右孩子为$2\cdoti + 2$ 
      3. 若节点p的下标为$i$，则父节点的下标为$\lfloor i/2\rfloor$
   2. 链式存储结构

![image-20210226230402719](https://i.loli.net/2021/02/26/lrUPGcnu1EvSwy8.png)

由上述的顺序存储结构可知，堆是用**物理上的线性表示逻辑上的非线性的数据结构**

具体的关于堆的操作，如刚开始的建堆，以及插入元素，删除堆顶元素，迭代等操作，可自行百度。

## 属性

```java
// 默认初始化容量
private static final int DEFAULT_INITIAL_CAPACITY = 11;
// 底层使用Object[]数组实现, 和ArrayList这些一样
transient Object[] queue; // non-private to simplify nested class access
// 标记元素个数
private int size = 0;
// 比较器, 说明其实例均是可比较的
private final Comparator<? super E> comparator;
```

因为底层使用的是数组，同时其本身是支持动态插入和删除的，所以同理，和**ArrayList**同理，扩容应该是其核心的地方。同时也要关注其是否生成新的数组对象。



## 构造函数

### PriorityQueue()

```java
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}
```



### PriorityQueue(int initialCapacity)

```java
public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}
```



### PriorityQueue(Comparator<? super E> comparator)

```java
public PriorityQueue(Comparator<? super E> comparator) {
    this(DEFAULT_INITIAL_CAPACITY, comparator);
}
```



### PriorityQueue(int initialCapacity, Comparator<? super E> comparator)

```java
public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
```



还有几个构造函数主要是用于将原各种类型的数据放入`PriorityQueue`中



## 增(入队)

在阅读这些方法的时候, 想起来自己手撕算法的时候, 经常会记混`List`, `Stack`, `Queue`, `Map`这些的添加删除操作的API...

下面罗列的是个人觉得不错的代表数据结构特性的添加/删除操作的API

| 数据结构 | 添加    | 删除     |
| :------- | ------- | -------- |
| List/Set | add()   | remove() |
| Stack    | push()  | pop()    |
| Queue    | offer() | poll()   |
| Map      | put()   | remove() |

### offer()

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}
```

宏观上来看, offer()方法会先检测是否需要扩容, 之后再插入元素, 最后进行调整





## 删(出队)

```java
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    return result;
}
```



## 获取队头元素

```java
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}
```



## siftUp原理

假设在数组queue的k位置插入元素key（小根堆）

* 不断的比较key和k的父节点e （$queue\lfloor(k-1/2\rfloor)$）的关系
  1. 若$key  <  e$ ，则$queue[k] = e$, 同时$k$回溯至$e$
  2. 若$key \geq e$ 或者  $k$已经到达跟节点，则结束循环
* $queue[k]==e$

```java
// 按照大意写的代码
public void siftUp(int k, E x) {
  Comparable<? super E> key = (Comparable<? super E>) x;
  while (k > 0) {
    int parent = (k-1) >> 1;
    Object e = queue[parent];
    // 可使用compareTo
    if (key > e) {
      break;
    }
    k = parent;
    queue[k] = e;
  }
	queue[k] = key;
}
```



## siftDown原理

自顶向下，选择小的节点，不断的比较、交换，直到：

1. 下标越界
2. 节点的值同时小于等于左孩子和右孩子

假设数组queue最后一个元素的值为key（小根堆），下标k从0开始，当k存在左孩子时，执行以下循环：

*  若k有右孩子，则比较左右孩子的值，然后选出小的孩子child
* 比较key和c=queue[child]的大小
  1. 若$key \leq c$ 结束循环
  2. 若$key > c$, 则$queue[k] = c, k = child$， 继续循环
* $queue[k] = key$

```java
public void siftDown(int k, E x) {
  Comparable<? super E> key = (Comparable<? super E>) x;
  // 完全二叉树 注意是 
  int half = size >>> 1;
  while (k < half) {
    int left = (k << 1) + 1;
    Object c = queue[left];
    int right = left + 1;
   	if (right < size && c > queue[right]) {
      c = queue[child = right];
    }
    if (key < c) {
      break;
    }
   	queue[k] = c;
    k = child;
  }
  queue[k] = key;
  
}
```



## 扩容策略