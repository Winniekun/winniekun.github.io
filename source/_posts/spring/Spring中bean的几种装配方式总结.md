---
title: Spring中bean的几种装配方式总结
date: 2019-01-14 19:50:54
tags: spring
categories: spring
thumbnail: https://i.loli.net/2020/04/15/s91ziSuFQLRVEvH.png
---

## Bean的装配

<!--more-->

**装配:** 创建应用对象之间的协作关系的行为， 也是DI的本质

### Java装配（显式）

### XML装配（显式）

### 自动化装配（隐式）

从以下两个角度来实现自动化装配

* 组件扫描
  * Spring会自动发现应用上下文中所创建的bean
* 自动装配
  * Spring会自动满足bean之间的依赖
    使用@Component注解，告诉Spring需要为此类创建bean， 因为Spring默认是不开启组件扫描的，
    所以需要编写一个配置文件，启用组件扫描（@ComponentScan）

#### 组件扫描    

##### 组件扫描的bean命名
Spring会根据类名指定ID（类名首字母小写）同时也可通过@Component(xxx)设置ID。

##### 组件扫描的基础包

默认会把使用@ComponentScan的包作为基础包，可以使用@ComponentScan(xxx)设置对应的基础包
当将所有的配置文件统一放在同一个包里面的时候，需要指定基础包

如果，所有的对象都是独立的，彼此之间没有任何依赖，那么只需要组件扫描即可。
但大多数情况下，很多对象都会依赖其他对象才能完成任务，所以就需要用到**自动装配**

#### 自动装配
自动装配就是让Spring自动满足bean依赖的一种方法。满足依赖的过程中，会在spring上下文中寻找某个bean需求的其他bean。

##### @Autowired

@Autowired将当前bean依赖的bean注入进来，注入方式可以有构造器注入，也可以是setter注入，看@Autowired注解在哪儿。这样就可以省去在xml中的装配

## References

* Spring实战第四版



