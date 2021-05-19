---
title: final、static、this、super关键字总结
date: 2019-04-10 09:47:33
tags: java基础
categories: java
thumbnail: https://i.loli.net/2019/07/02/5d1afa029874e50160.jpg
---

## final  static this super关键字总结

### final 关键字

主要用于三个地方：类、方法、变量

1. 类：**用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。**
2. 方法：
   1. 方法锁定： 防止任何继承了类修改该方法
   2. ~~效率~~
3. 变量： **对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。**

<!--more-->

### static 关键字

主要为四个地方放： 修饰成员变量和成员方法， 静态代码块、静态内部类、静态导包

1. 修饰静态成员变量和成员方法

   被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：`类名.静态变量名` `类名.静态方法名()`

2. 静态代码块

   静态代码块定义在类中方法外, ==静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次.==

3. 静态内部类

   静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：

   1. 它的创建是不需要依赖外围类的创建。
   2. 它不能使用任何外围类的非static成员变量和方法。

4. 静态导包

   `mport static` 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

   ```java
   import static com.train.keywords.Test.*;
   
   public class FinalForKnow {
       // final 用于 类 方法 变量
       private static final String a = "1123";
   
       public static void main(String[] args) {
           System.out.println(a);
           System.out.println(name);
           System.out.println(age);
           // 1123
           // 维坤坤
           // 0
       }
   }
   
   
   public class Test {
       public static  int age;
       public static String name = "维坤坤";
   }
   ```



### this 关键字

引用类的当前实例

### super 关键字

从子类访问父类的方法和变量

```java
public class Super {
    protected int number;
     
    protected showNumber() {
        System.out.println("number = " + number);
    }
}
 
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

- 在构造器中使用 `super（）` 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行。
- this、super不能用在static方法中。

