---
title: ArrayDeque-analysis
date: 2020-07-08 15:27:56
tags: 
	- JDK源码
	- Queue
categories: 源码
thumbnail: 
---



## 依赖

<!--more-->

![ArrayDeque依赖](https://i.loli.net/2020/07/08/3I9zKjOpsNvPdMw.png)

实现了Deque接口, Serializable接口, Cloneable接口, 继承了AbstractCollection类, 同时可以看到的是Deque接口继承自Queue接口, 它是对Queue的一种增强. Deque 接口的实现类可以被当作 FIFO（队列）使用，也可以当作 LIFO（栈）来使用

## Queue和Deque

Deque是对Queue的增强, 我们可以先看下Queue的具体设计:

```java
public interface Queue<E> extends Collection<E> {
    
    // 添加
    boolean add(E e);
	
    // 添加 不同的实现使用的添加方式不同
    // list使用add
    // queue使用offer
    boolean offer(E e);

    // 删除
    E remove();
   
    // 弹出
    E poll();
    
    E element();

   	// 获取头部数据
    E peek();
}

```

Deque的设计

```java
public interface Deque<E> extends Queue<E> {
    
    void addFirst(E e);

    void addLast(E e);

    
    boolean offerFirst(E e);

    boolean offerLast(E e);

    
    E removeFirst();

    
    E removeLast();

    
    E pollFirst();

    
    E pollLast();

    
    E getFirst();

    
    E getLast();

   
    E peekFirst();

   
    E peekLast();

    
    boolean removeFirstOccurrence(Object o);

   
    boolean removeLastOccurrence(Object o);

    
    boolean add(E e);

    
    boolean offer(E e);

    
    E remove();
  
    E poll();
   
    E element();

    E peek();
    
    // *** 栈 ***//
    
    void push(E e);

    E pop();
    
    // *** Collection中的方法 ***

    boolean remove(Object o);

    boolean contains(Object o);

    public int size();

    Iterator<E> iterator();

    Iterator<E> descendingIterator();

}

```

Deque中新增了以下几类方法：

1. *First，表示从队列头操作元素；
2. *Last，表示从队列尾操作元素；
3. *push(e)，pop()，以栈的方式操作元素的方法；

其抽象的样子



## 字段&属性

```java
// 底层使用数组存储元素
transient Object[] elements; // non-private to simplify nested class access
// 队列头位置
transient int head;
// 队列尾位置
transient int tail;
// 最小初始容量
private static final int MIN_INITIAL_CAPACITY = 8;
```

使用数组存储元素, 默认的最小初始化容量为8, 同时有头尾两个标记位



## 构造函数

```java
// 默认构造函数数组容量为16
public ArrayDeque() {
    elements = new Object[16];
}

// 指定初始容量构造函数
public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

// 将集合c中的元素初始化到数组中
public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());
    addAll(c);
}

// 初始化数组
private void allocateElements(int numElements) {
    elements = new Object[calculateSize(numElements)];
}

// 和HashMap的类似, 求大于且最接近numelemnets的2的幂次且不小于8
private static int calculateSize(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;
        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    return initialCapacity;
}
```

通过构造方法，我们知道默认初始容量是16，最小容量是8。



## 入队

### 头部入队

```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    // 将head指针减1并与数组长度减1取模
    // 这是为了防止数组到头了边界溢出
    // 如果到头了就从尾再向前
    // 相当于循环利用数组
    elements[head = (head - 1) & (elements.length - 1)] = e;
    // 如果头尾挨在一起了，就扩容
    // 扩容规则也很简单，直接两倍
    if (head == tail)
        doubleCapacity();
}

```

### 扩容

```java
// 扩容为两倍
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    // 将旧数组head之后的元素拷贝到新数组中
    System.arraycopy(elements, p, a, 0, r);
    // 将旧数组下标0到head之间的元素拷贝到新数组中
    System.arraycopy(elements, 0, a, r, p);
    // 赋值为新数组
    elements = a;
    // head指向0，tail指向旧数组长度表示的位置
    head = 0;
    tail = n;
}

```

#### 可视化

![](https://i.loli.net/2020/07/08/lzG7NoCmk6ZJjpB.jpg)

### 尾部入队

```java
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    // 在尾指针的位置放入元素
    // 可以看到tail指针指向的是队列最后一个元素的下一个位置
    elements[tail] = e;
    // tail指针加1，如果到数组尾了就从头开始
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}
```

1. 入队有两种方式，从队列头或者从队列尾；
2. 如果容量不够了，直接扩大为两倍；
3. 通过取模的方式让头尾指针在数组范围内循环；
4. x & (len - 1) = x % len，使用&的方式更快(**len=2的幂次等式成立**)；



## 出队

### 头部出队

```java
public E pollFirst() {
    int h = head;
    @SuppressWarnings("unchecked")
    // 取队列头元素
    E result = (E) elements[h];
    // Element is null if deque empty
    if (result == null)
        return null;
    // 队头元素为空
    elements[h] = null;     // Must null out slot
    // 队头右移
    head = (h + 1) & (elements.length - 1);
    return result;
}
```



### 尾部出队

```java
public E pollLast() {
    // 因为队尾标记的是最后一个为不为空的元素的后一位
    int t = (tail - 1) & (elements.length - 1);
    @SuppressWarnings("unchecked")
    // 获取队尾元素
    E result = (E) elements[t];
    if (result == null)
        return null;
    // 队尾设置为空
    elements[t] = null;
    // 重新设置tail
    tail = t;
    return result;
}
```



以下是一次ArrayDeque的头部入队, 尾部出队的可视化操作

```java
ArrayDeque<Integer> arrayDeque = new ArrayDeque<>(6);
ArrayDequeUtil.getField(arrayDeque);
for (int i = 1; i <=10; i++) {
    arrayDeque.addFirst(i);
    ArrayDequeUtil.getField(arrayDeque);
}
arrayDeque.pollLast();
ArrayDequeUtil.getField(arrayDeque);
```



![模拟可视化操作](https://i.loli.net/2020/07/08/4uyjsf9nVXdGZTm.png)



## 栈的实现

### 入栈

以下是Deque实现入栈的操作

```java
public void push(E e) {
    addFirst(e);
}
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
```

最后, 其依赖的还是`addFirst()`, 也就是从头部添加元素

### 出栈

以下是Deque实现出栈的操作

```java
public E pop() {
    return removeFirst();
}
public E removeFirst() {
    E x = pollFirst();
    if (x == null)
        throw new NoSuchElementException();
    return x;
}
```

最后, 其依赖的还是`pollFirst()`, 也就是从头部删除元素,  也就是后入的元素先出

说白了**实现栈的方式就是仅仅只操作队列头即可**



## 总结&面试小问题

1. 什么是双端队列？
   * 双端队列是一种特殊的队列，它的两端都可以进出元素，故而得名双端队列
2. ArrayDeque是怎么实现双端队列的？
   * 底层为数组, 然后通过取模的方式构造一个循环数组, 出队入队是通过头尾指针循环利用数组实现的. 
3. ArrayDeque是线程安全的吗？
   * 不是线程安全
4. ArrayDeque的扩容机制？
   * ArrayDeque在容量不足时(head == tail)会出发扩容, 扩容为原先的两倍, 具体的扩容机制, 可以看上述可视化过程







