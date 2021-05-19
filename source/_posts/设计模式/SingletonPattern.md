---
title: 单例模式
date: 2020-07-02 21:35:20
tags: 创建型模式
categories: 设计模式
thumbnail:
---

## 前言

<!--more-->

> “你知道茴香豆的‘茴’字有几种写法吗？”

纠结单例模式有几种写法有用吗? 有点用, 面试中经常选择其中一种或几种写法作为话头, 以此展开考察面试者的code style 以及其他相关的知识点. 但是过于纠结这些写法, 反而类似于"茴"字有几个写法...

思想, 思想, 思想. 重要事情说三遍



### 类加载顺序

类加载(classLoader)机制一般遵从下面的加载顺序

如果类还没有被加载：

- 先执行父类的静态代码块和静态变量初始化，静态代码块和静态变量的执行顺序跟代码中出现的顺序有关。
- 执行子类的静态代码块和静态变量初始化。
- 执行父类的实例变量初始化
- 执行父类的构造函数
- 执行子类的实例变量初始化
- 执行子类的构造函数

同时，加载类的过程是线程私有的，别的线程无法进入。

如果类已经被加载：

静态代码块和静态变量不在重复执行，再创建类对象时，只执行与实例相关的变量初始化和构造方法。

### static关键字

一个类中如果有成员变量或者方法被static关键字修饰，那么该成员变量或方法将独立于该类的任何对象。它不依赖类特定的实例，被类的所有实例共享，只要这个类被加载，该成员变量或方法就可以通过类名去进行访问，它的作用用一句话来描述就是，不用创建对象就可以调用方法或者变量，这简直就是为单例模式的代码实现量身打造的。



## 饿汉模式

```java
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {

    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

在类加载的时候就完成了实例化, 避免了多线程的同步问题. 当然缺点也是有的, 因为类加载时就实例化了, 没有达到Lazy Loading (懒加载) 的效果, 如果该实例没被使用, 内存就浪费了. 



## 懒汉模式

顾名思义,  就是初始化的时候, 不会主动的去创建实例, 而是在调用**getInstance()**的时候, 才会被动的去创建. 

```java
public class Singleton {
    private static Singleton instance = null;

    private LazySingleton(){

    }

    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

在单线程的情况下, 没有任何问题, 但是因为其为线程不安全的,  在多线程的情况下, 譬如两个线程A,  B都执行了**getInstance()**方法, 并且都执行到了第9行代码, 然后A因为其他原因, 休眠了一会儿, 待B创建了实例对象之后, A有创建了一个, 显然, 这是不符合单利模式的

### 解决1

在**getInstance()**方法内部添加同步代码块, 或者直接将该方法改为同步方法

```java
public class Singleton {
    private static Singleton instance = null;

    private LazySingleton(){

    }

    public synchronized static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

好处是写起来简单, 且绝对线程安全; 坏处是并发性能极差, 事实上完全退化到了串行. 单例只需要初始化一次, 但就算初始化以后, synchronized的锁也无法避开, 从而**getInstance()**完全变成了串行操作. **性能不敏感的场景建议使用**。

### 解决2

也就是臭名昭著的双重检验方法

```java
public class Singleton {

    private static Singleton instance;

    // 双重锁检验
    public static Singleton getInstance() {
        if (instance == null) { // 第7行
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 第10行
                }
            }
        }
        return instance;
    }
}
```

由于指令重排序的问题, 可能会被重排序为如下:

```java
instance = new Singleton(); // 第10行

// 可以分解为以下三个步骤
1 memory=allocate();// 分配内存 相当于c的malloc
2 ctorInstanc(memory) //初始化对象
3 s=memory //设置s指向刚分配的地址

// 上述三个步骤可能会被重排序为 1-3-2，也就是：
1 memory=allocate();// 分配内存 相当于c的malloc
3 s=memory //设置s指向刚分配的地址
2 ctorInstanc(memory) //初始化对象
```

**而一旦假设发生了这样的重排序，比如线程A在第10行执行了步骤1和步骤3，但是步骤2还没有执行完。这个时候另一个线程B执行到了第7行，它会判定instance不为空，然后直接返回了一个未初始化完成的instance！**



### 解决3

```java
public class Singleton {

    private static volatile Singleton instance;

    // 双重锁检验
    public static Singleton getInstance() {
        if (instance == null) { // 第7行
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 第10行
                }
            }
        }
        return instance;
    }
}
```

这样就保证了线程的安全



## 枚举模式

```java
// 枚举
// ThreadSafe
public enum Singleton4 {
  SINGLETON;
}
```

使用javap进行反编译得到如下结果

```java
// 枚举
// ThreadSafe
public class Singleton4 extends Enum<Singleton4> {
  ...
  public static final Singleton4 SINGLETON = new Singleton4();
  ...
}
```

可以看到, 其本质也还是饿汉模式



## 静态内部类实现

```java
/**
 * JVM在类的初始化阶段（class被加在后，且被线程使用之前）会执行类的初始化
 * 执行类的初始化期间，JVM会去获取一个锁，锁会同步多个线程对同一个类的初始化
 * @author weikunkun
 * @since 2021/3/20
 */
public class InstanceFactory {
    private static class InstanceHolder {
        public static Object object = new Object();
    }

    public static Object getInstance() {
        return InstanceHolder.object;
    }
}
```

