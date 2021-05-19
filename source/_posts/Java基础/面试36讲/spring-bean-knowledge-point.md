---
title: Spring Bean的生命周期和作用域
date: 2020-07-01 20:38:31
tags: 
	- java基础
	- 面试36讲
categories: java
thumbnail:
---

## Spring Bean的生命周期

<!--more-->

Spring的生命周期从总体上可以分为*创建*和*销毁*两个部分. 

![image-20210405232540275](https://i.loli.net/2021/04/05/mnZHsqRzJoFYxAW.png)



1. 实例化Bean对象
2. 设置Bean属性
3. 若是通过各种**Aware**接口声明了依赖关系,则会注入 Bean 对容器基础设施层面的依赖
   1. 如果bean实现了BeanNameAware接口,  则将BeanId通过setBeanName注入Spring中. 
   2. 如果bean实现了BeanFactoryAware接口, 则将BeanFactory容器实例通过setBeanFactory注入Spring中
   3. 如果bean实现了ApplicationContextAware接口, 则将bean所在的应用的上下文通过setApplicationContext注入Spring中
4. 如果bean实现了BeanPostProcessor接口, 则调用前置初始化方法, postProcessorBeforeInitialization()
5. 如果bean实现了InitializingBean接口, 则调用afterPropertiesSet()方法, 
6. 调用自身的初始化方法
7. 如果bean实现了BeanPostProcessor接口, 则调用后置初始化方法, postProcessorAfterInitialization()
8. bean已经准备就绪, 可以使用, 一直驻留在应用上下文中, 直到应用上下文被销毁
9. 如果bean实现了DisposableBean接口, 则调用destroy()方法
10. 调用自身的destroy方法



## Spring Bean 的作用域

Spring Bean 有五个作用域，其中最基础的为加粗的两种

1. **singleton**
   1. Spring的默认作用于, 为每个IOC容器创建唯一的Bean实例
2. **prototype**
   1. 针对每个getBean请求, 都会创建一个Bean实例
3. request
   1. 每次Http请求都会创建新的Bean, 仅适用于WebApplicationContext
4. session
   1. 一个HttpSession定义一个Bean. 该作用于仅适用于WebApplicationContext
5. globalSession
   1. 同一个全局Http Session 定义一个Bean. 该作用域同样仅适用于WebApplicationContext



