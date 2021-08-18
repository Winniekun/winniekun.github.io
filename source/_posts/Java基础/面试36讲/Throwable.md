---
title: Java中异常类体系
date: 2020-07-02 19:57:04
tags: 
	- java基础
	- 面试36讲
categories: java
thumbnail:
---

# 解答

**Exception**和**Error**均继承自**Trowable**, 如下图所示:

![依赖图](https://i.loli.net/2020/07/02/aIOqAKGJe8yZ9cX.png)

# Throwable

```java
public Throwable() {...}
public Throwable(String message) {...}
public Throwable(String message, Throwable cause) {...}
public Throwable(Throwable cause) {...}
...
```

通过构造函数，可以看出，核心的参数为两个：

1. message
   1. 表示异常信息
2. cause
   1. 造成异常的异常（套娃行为！！！）
   2. 上层的异常通过底层的异常触发一层一层的抛上来

## Exception、Error

在 Java 中只有 **Throwable** 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型, *Exception和Error代表了对不同异常情况的分类*

1. Exception:

   * 程序正常运行的情况下, 可以预料的意外情况, 可以并且应该被捕获, 之后进行处理, 最后仍能程序继续正常运行

   又可以被细分为**可检查异常**和**不可检查异常**

   1. 可检查异常:
      * 编译期间出现的异常, 必须进行捕获处理
   2. 不可检查异常:
      * 运行期间出现的异常, 譬如ArrayIndexOutOfBoundsException, NullPointerException之类

2. Error:

   * 正常情况下, 不大可能出现的异常,  绝大多数Error的出现会导致程序崩溃, 不便于也不需要捕获, 譬如OutOfMemoryError之类, 都是Error的子类

粗略了解了上述的`Exception`和`Error`的关系和概述之后，再来个详细的图，加深理解

![Java 异常大家族](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210603201047306.png)

有这么多的异常类型，主要还是为了能够更加准确的捕获异常，增加代码的可读性、可维护性。同时可以自定义异常，更加精确阐述异常原因。通常是继承`Exception`或者`Exception`中的某个子类。

## 异常处理

### try-catch

最直接的解决方式就是出现异常，解决异常。

```java
try {
  // 
  doSomething();
} catch (Exception e) {
  // 1. 记录异常
  log.error("something error");
  // 2. 执行异常的业务逻辑
  doExcepptionHandler();
  // 3. 也可以执行完异常逻辑之后，再抛出异常
  throw e
}
```

### finally

```java
try {
  
} catch (Exception e) {
  
} finally {
  
}
```

无论是否发生异常，都会执行。

也可以这样：

```java
try {
  
} finally {
  
}
```

不捕获出现的异常，异常继续向上传递。



# 总结

对于异常的处理，总有一层代码需要为异常负责，可能是知道如何处理异常的代码，可能是面对用户的代码，可能是主程序。

> 有了异常机制后，程序的正确逻辑和异常逻辑可以相分离，异常情况可以集中处理，异常还可以向上传递，不需要每层方法都需要处理。