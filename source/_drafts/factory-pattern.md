---
title: 工厂模式
mathjax: false
tags: 创建型
categories: 设计模式
---

## 序

工厂模式

## 简单工厂模式

称为静态工厂方法模式，是创建型的一种。在简单工厂模式中，可以根据实际的参数不同返回不同的实例。同时在简单工厂模式中会定义一个类负责创建其他类的实例，被创建的实例也通常具有共同的父类。

![简单工厂模式关系图](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806213224887.png)





### 实战小Demo

<img src="https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806212559455.png" alt="简单工厂模式" style="zoom:50%;" />



### 实际应用

JDK中的DateFormate

```java
private static DateFormat get(LocaleProviderAdapter adapter, int timeStyle, int dateStyle, Locale loc) {
    DateFormatProvider provider = adapter.getDateFormatProvider();
    DateFormat dateFormat;
    if (timeStyle == -1) {
        dateFormat = provider.getDateInstance(dateStyle, loc);
    } else {
        if (dateStyle == -1) {
            dateFormat = provider.getTimeInstance(timeStyle, loc);
        } else {
            dateFormat = provider.getDateTimeInstance(dateStyle, timeStyle, loc);
        }
    }
    return dateFormat;
}
```

### 小总结

**优点：**

1. 对象的创建和对象的使用分离，创建交给工厂类负责， 上层应用不用担心如何创建

**缺点：** 

1. 不够灵活，如果需要新增一个产品，就需要在原工厂类内部添加一个分支，违反了开闭原则。同时如果多个判断条件共同决定创建对象，则后期修改会越来越复杂
2. 工厂类集中了所有产品的创建逻辑，职责过重，一旦不能正常工作，整个系统都要受到影响

适用于创建对象较少的场景

## 工厂方法模式

> 定义一个用于创建对象的接口，让子类决定实例化哪个类。工厂方法模式使一个类的实例化延迟到实现该产品的具体子工厂。

工厂方法模式中，将简单工厂中的核心工厂变为一个抽象接口。负责给出不同工厂应该实现的方法，自身不再负责创建各种产品，而是将具体的创建操作交给实现该接口的子工厂类来做。

![工厂方法模式](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806222348723.png)

原因：面性对象的程序设计中一个很重要的原则就是开闭原则，其规定：程序对于扩展是开放的，对于修改是关闭的。

### 实际应用

JDK中的Collection接口中Iterator的实现。Collection中不同的实现类生产适合于自己的迭代器对象

**Factory：** Collection

	1. SubFactoryA：LinkedList
	2. SubFactoryB：ArrayList

**Product：**Iterator

1. ProductA： ListItr
2. ProductB：Itr

![关系图](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806224409440.png)

## 抽象工厂模式

![抽象工厂模式关系图](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806230508468.png)



![一些概念](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20210806232425961.png)

在工厂方法模式中具体工厂负责生产具体的产品，每一个具体工厂对应一种具体产品，工厂方法具有唯一性，一般情况下，一个具体工厂中只有一个或者一组重载的工厂方法。但是有时候我们希望一个工厂可以提供多个产品对象，而不是单一的产品对象，如一个电脑厂不仅仅能够生产CPU还要能够生产主板、硬盘、内存条这些。



抽象工厂模式为创建一组对象提供了一种解决方案。与工厂方法模式相比，抽象工厂模式中的具体工厂不只是创建一种产品，它负责创建一族产品。抽象工厂模式定义如下：

> 提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式。

### 实际应用

Spring的BeanFactory。

## 总结

抽象工厂模式是工厂方法模式的进一步延伸，由于它提供了功能更为强大的工厂类并且具备较好的可扩展性，在软件开发中得以广泛应用，尤其是在一些框架和 API 类库的设计中，例如在 Java 语言的 AWT（抽象窗口工具包）中就使用了抽象工厂模式，它使用抽象工厂模式来实现在不同的操作系统中应用程序呈现与所在操作系统一致的外观界面。抽象工厂模式也是在软件开发中最常用的设计模式之一。

**优点：**

- 抽象工厂模式隔离了具体类的生成，使得客户并不需要知道什么被创建。由于这种隔离，更换一个具体工厂就变得相对容易，所有的具体工厂都实现了抽象工厂中定义的那些公共接口，因此只需改变具体工厂的实例，就可以在某种程度上改变整个软件系统的行为。
- 当一个产品族中的多个对象被设计成一起工作时，它能够保证客户端始终只使用同一个产品族中的对象。
- 增加新的产品族很方便，无须修改已有系统，符合“开闭原则”。

**缺点：**

增加新的产品等级结构麻烦，需要对原有系统进行较大的修改，甚至需要修改抽象层代码，这显然会带来较大的不便，违背了“开闭原则”。

