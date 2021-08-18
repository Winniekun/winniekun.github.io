---
title: <clinit>()和<init>()区别
date: 2020-04-30 19:10:18
tags: jvm
categories: java
thumbnail: 
---



#  \<clinit\>()和\<init\>()区别

## 1. \<clinit\> 

在Java类的`初始化`过程中，编译器按照**按语句在源文件中出现的顺序**，依此收集类中的所有**静态代码块儿中的内容和静态变量的赋值动作**合并成\<clinit\>方法。

如果没静态代码块儿和静态变量，则不会产生\<clinit\>。

并且 `<clinit>()` 不需要显式调用父类（接口除外，接口不需要调用父接口的初始化方法，只有使用到父接口中的静态变量时才需要调用）的初始化方法 `<clinit>()`，虚拟机会保证在子类的 `<clinit>()` 方法执行之前，父类的 `<clinit>()` 方法已经执行完毕。

> 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的<clinit>()方法，其他线程都需要阻塞等待，直到活动线程执行<clinit>()方法完毕。如果在一个类的<clinit>()方法中有耗时很长的操作，就可能造成多个进程阻塞(需要注意的是，其他线程虽然会被阻塞，但如果执行<clinit>()方法后，其他线程唤醒之后不会再次进入<clinit>()方法。同一个加载器下，一个类型只会初始化一次。)，在实际应用中，这种阻塞往往是很隐蔽的。



## 2.\<init\>

对象构造时用来初始化对象的，完成构造器中的内容以及代码块儿中的内容。



## 执行顺序

```java
public class ClinitInit {
    public static ClinitInit test;

    static {
        System.out.println("static 开始");
        // 下面这句编译器报错，非法向前引用
        // System.out.println("x=" + x);
        test = new ClinitInit();
        System.out.println("static 结束");

    }

    public ClinitInit() {
        System.out.println("构造 开始");
        System.out.println("x=" + x + ";y=" + y);
        // 构造器可以访问声明于他们后面的静态变量
        // 因为静态变量在类加载的准备阶段就已经分配内存并初始化0值了
        // 此时 x=0，y=0
        x++;
        y++;
        System.out.println("x=" + x + ";y=" + y);
        System.out.println("构造器结束");
    }

    public static ClinitInit getInstance() {
        return test;
    }

    public static int x = 6;
    public static int y = 9;

    public static void main(String[] args) {
        ClinitInit obj = ClinitInit.getInstance();
        System.out.println("x=" + obj.x);
        System.out.println("y=" + obj.y);
    }
}
```

**虚拟机首先执行的是类加载初始化过程中的 `<clinit>()` 方法，也就是静态变量赋值以及静态代码块中的代码，如果 `<clinit>()` 方法中触发了对象的初始化，也就是 `<init>()` 方法，那么会进入执行 `<init>()` 方法，执行 `<init>()` 方法完成之后，再回来继续执行 `<clinit>()` 方法。**

上面代码中，先执行 static 代码块，此时调用了构造器，构造器中对类变量 x 和 y 进行加 1 ，之后继续完 static 代码块，接着执行下面的 `public static int x = 6;` 来重新给类变量 x 赋值为 6，因此，最后输出的是 x=6， y=1。

如果希望输出的是 x=7，y=1，很简单，将语句 `public static int x = 6;` 移至 static 代码块之前就可以了。