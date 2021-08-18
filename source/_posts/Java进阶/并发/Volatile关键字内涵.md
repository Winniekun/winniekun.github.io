---
title: Volatile关键字内涵
date: 2020-01-18 12:35:26
tags: 并发
categories: java
thumbnail: https://i.loli.net/2020/03/11/fv1JyPacVRzM4ZO.jpg
---

## 内涵与表象

<!--more-->

volatile主要有一下的功能：

1. 保证变量的内存可见性

   > 内存可见性：通俗来说就是，线程A对一个volatile变量的修改，对于其它线程来说是可见的，即线程每次获取volatile变量的值都是最新的

2. 禁止volatile变量与普通变量**重排序**



![volatile.png](https://i.loli.net/2020/04/24/sBE5dy3Dfh7ctLI.png)

### **定义: **

> Volatile是轻量级的synchronized。它在多处理器开发过程中保证的共享变量的**可见性**、**有序性**

## 使用

### volatile的使用场景

但是使用volatile必须**满足两个条件**：

1. 对变量的写操作不依赖当前值，如多线程下执行a++，是无法通过volatile保证结果准确性的

2. 该变量没有包含在具有其它变量的不变式中(如下例子解释)

   ```java
   public class NumberRange {
       private volatile int lower = 0;
        private volatile int upper = 10;
   
       public int getLower() { return lower; }
       public int getUpper() { return upper; }
   
       public void setLower(int value) { 
           if (value > upper) 
               throw new IllegalArgumentException(...);
           lower = value;
       }
   
       public void setUpper(int value) { 
           if (value < lower) 
               throw new IllegalArgumentException(...);
           upper = value;
       }
   }
   ```

   若是有两个线程同时分别执行了**setLower(8)**和**setUpper(5)** 然后 均通过了判断, 则最后的范围从$[0, 10]$ 变为了 $[8, 5]$成为了一个无效的范围, 这样就出现了问题. 这种场景下就只能使用**synchronized**, 同一时间只允许getLower和getUpper方法中其中一个执行

#### 状态标记量

```java
public class ServerHandler {
    private volatile isopen;
    public void run() {
        if (isopen) {
           //促销逻辑
        } else {
          //正常逻辑
        }
    }
    public void setIsopen(boolean isopen) {
        this.isopen = isopen
    }
}
```



#### 单例模式的double check

单例模式的一种实现方式，但很多人会忽略volatile关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是100%，说不定在未来的某个时刻，隐藏的bug就出来了。

```java
public class Singleton{
    private volatile static Singleton instance;
    
    public static Singleton getInstance(){
        if(instance == null){
            sychronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```



在JMM中, 有主内存和本地内存, 每个线程都有一个本地内存, 且共享主内存中的数据, `普通变量`和`volatile变量`的区别如下:

1. 普通变量进行读操作的时候, 首先会读取本地内存中的数据, 若是不存在, 则去主内存中拷贝一份在本地内存, 写操作只会写入到本地内存. 这样其他变量就不会读取变量的最新值
2. voaltile遍历进行读操作的时候,  JMM会将本地内存的数据设置为无效, 要求线程从主内存读取数据, 写操作时, JMM会将工作内存中的数据刷新到主内存中. 这样其他遍历就能够读取变量的最新值

## 可见性、有序性实现原理

### **可见性：**

1. volatile修饰的变量**进行写操作**转换成汇编语言，会添加**Lock前缀的指令**

   **lock前缀的指令**在多核处理器的情况下，会引发以下的两个事情：

   1. 会将当前处理器缓存的数据，写回主内存
   2. 同时写回内存的操作，会使得其他处理器缓存的该内存地址的数据无效

2. 多处理器下，为了确保多处理器的缓存是一致的，会去实现缓存一致性协议，每个处理器通过**嗅探**在总线上传播的数据来检测自己缓存的数据是否过期。如果发现缓存的内存地址被修改，会将自身缓存的内存地址置为无效，然后下次操作该数据的时候，重新从主存中将数据重新读取缓存

### 有序性

#### 禁止重排序

JVM通过**内存屏障**来实现限制处理器的重排序。

什么是内存屏障？硬件层面，内存屏障分两种：读屏障（Load Barrier）和写屏障（Store Barrier）。内存屏障有两个作用：

1. 阻止屏障两侧的指令重排序；
2. 强制把写缓冲区/高速缓存中的脏数据等写回主内存，或者让缓存中相应的数据失效。

编译器在**生成字节码时**，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。编译器选择了一个**比较保守的JMM内存屏障插入策略**，这样可以保证在任何处理器平台，任何程序中都能得到正确的volatile内存语义。这个策略是：

- 在每个volatile写操作前插入一个StoreStore屏障；
- 在每个volatile写操作后插入一个StoreLoad屏障；
- 在每个volatile读操作后插入一个LoadLoad屏障；
- 在每个volatile读操作后再插入一个LoadStore屏障。

大概示意图是这个样子：

![内存屏障](https://i.loli.net/2020/05/27/b6yp7RVDshJ9aFq.png)

再介绍一下volatile与普通变量的重排序规则:

1. 如果第一个操作是volatile读，那无论第二个操作是什么，都不能重排序；
2. 如果第二个操作是volatile写，那无论第一个操作是什么，都不能重排序；
3. 如果第一个操作是volatile写，第二个操作是volatile读，那不能重排序。

## References

* Java并发编程的艺术
* 深入浅出Java多线程