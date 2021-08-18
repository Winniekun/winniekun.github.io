---
title: Spring AOP分析
mathjax: false
tags: aop
categories: spring
---

## 序

AOP的学习，以及其在Spring中的应用和实现

## 为什么会出现AOP



## AOP的术语

### Joinpoint

### Pointcut

### Advice

### Aspect

### Weaving

## Spring 中的AOP

### Joinpoint

对于AOP而言，`Joinpoint`有很多中类型，如构造方法调用、字段获取、字段设置、方法的调用等，不过Spring AOP 并不尝试提供所有的类型，更确切的来说，仅支持方法级别的调用。主要原因是Spring AOP 是基于 JDK 动态代理和 CGLIB 实现的动态AOP。尽管只支持方法级别的调用，不过已经满足了80%的需求了。之所以不完全支持AOP的所有功能，主要原因如下：

1. Spring 是一个微型框架，对于AOP的支持，最终目标是简单而又强大。为了剩余的80%，而使得自身框架更为臃肿显然不合适。如果需求正好是超过80%的需求，也可以通过AspectJ来做，并且Spring AOP也提供了对AspecJ的支持。
2. 对于字段设置、字段获取级别的Joinpoint，本身就违背了封装这一特性。并且完全可以通过getter、setter这些方法级别的调用来做。

### Pointcut

`org.springframework.aop.Pointcut`为Spring中切点的顶级接口，具体内容如下：

```java
public interface Pointcut {

	/**
	 * 被织入的对象
	 * Return the ClassFilter for this pointcut.
	 * @return the ClassFilter (never {@code null})
	 */
	ClassFilter getClassFilter();

	/**
	 * 被织入的方法
	 * Return the MethodMatcher for this pointcut.
	 * @return the MethodMatcher (never {@code null})
	 */
	MethodMatcher getMethodMatcher();


	/**
	 * 默认会为系统中所有的独享，以及对象上被支持的Joinpoint进行匹配
	 * Canonical Pointcut instance that always matches.
	 */
	Pointcut TRUE = TruePointcut.INSTANCE;

}
```

`MethodMatcher`

```java
public interface MethodMatcher {

	/**
	 * 重载方法1
	 * 进行拦截时忽略参数
	 */
	boolean matches(Method method, Class<?> targetClass);

	/**
	 * 是否考虑具体的参数
	 */
	boolean isRuntime();

	/**
	 * 重载方法2
	 * 进行拦截时，需要考虑具体参数
	 */
	boolean matches(Method method, Class<?> targetClass, Object... args);


	/**
	 * 
	 */
	MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;

}
```

1. 拦截时不考虑具体的参数（`isRuntime`返回`true`），这种类型的`MethodMatcher`称为`StaticMethodMatcher`，因为不用考虑具体的参数，所以可以通过框架内部进心缓存，进而提高性能
2. 拦截时考虑具体的参数（`isRuntime `返回`false`  ），这种类型的`MethodMatcher`称为`DynamicMethodMatcher`，因为每次拦截时都需要考虑参数，所以无法进行缓存。性能相比于`StaticmethodMatcher`差一些。

具体的实现类划分如下：

![Pointcut](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/Pointcut.png)

### Advice

和AOP无差别，分为前置，后置、异常、环绕、返回通知。

### Aspect

确定了`Pointcut`、`Advice`之后，就可以分门别类的进行组合了。需要注意的是，在Spring中没有完全明确的Aspect概念，所以在实现还特性上有写特殊。`Advisor`代表Spring中的`Advice`。不过不同的是，`Advisor`通常只支持一个`Pointcut`和一个`Advice`。这个和AOP中的`Aspect`是有区别的，所以可以将其理解为`Advice`的一个特例。对于`Advisor`本身的实现来说，可以分为两大类，如下：

![Advisor](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/Advisor.png)

1. PointCutAdvisor
   1. Spring中最常用的Aspect，支持多种形式。主要是因为PointCut的类型、Advice类型有很多种，针对不同的组合，需要选择不同的Advisor。
      1. `DefaultPointcutAdvisor` 除了不能支持`Introduction`类型的`Advice`之外，剩余的任何类型的`Pointcut`和`Advice`都支持
      2. `NameMatchMethodPointcutAdvisor`，不支持`Introduction`类型的`Advice`并且指定`PointCut`类型为`NameMatchmethodPointcut`。
      3. `RegexpMethodPointcutAdvisor`，也限定了自身的`PointCut`，只能使用通过正则表达式相关的`Pointcut`实现类。也不支持`Introduction`类型的`Advice`。
2. IntroductAdvisor
   1. 仅支持应用于类级别的拦截，也就是只用用于`Introduction`型的`Advice`。
3. Ordered
   1. 具体的场景中，并不是一个切面对应于一个JoinPoint，可能是多个切面共同关注同一个JoinPoint，这时候执行顺序就显得尤为重要了。

### Weaving

![ProxyFactoryBean](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/ProxyFactoryBean.png)

#### ProxyFactory

作为具体织入类，其主要负责两个部分的内容：

1. 确定需要织入的对象
2. 织入具体的切面

因为Spring AOP使用代理模式实现AOP，具体技术为动态代理以及CGLIB两种。所以`ProxyFactory`会通过某些行为对这两种情况进行区分。当然，需要明确的是，对于Spring而言，`ProxyFactory`属于最基础的织入器，其另外两个兄弟`ProxyFactoryBean`、`AspectJProxyFactory`功能更为的丰富。`ProxyFactory`没有结合Spring的IoC，需要我们自己硬编码，创建出需要进行织入的对象。

```java
@Test
public void testInterface() {
    MockTask mockTask = new MockTask();
    ProxyFactory weaver = new ProxyFactory(mockTask);
    NameMatchMethodPointcutAdvisor advisor = new NameMatchMethodPointcutAdvisor();
    // 切点
    advisor.setMappedName("execute");
    // 通知
    advisor.setAdvice(new PerformanceMethodInterceptor());
    weaver.addAdvisor(advisor);
    ITask proxyObj = (ITask)weaver.getProxy();
    proxyObj.execute(null);
    System.out.println(proxyObj.getClass());
}
@Test
public void testClassWithInterface() {
    MockTask mockTask = new MockTask();
    ProxyFactory weaver = new ProxyFactory(mockTask);
    NameMatchMethodPointcutAdvisor advisor = new NameMatchMethodPointcutAdvisor();
    advisor.setMappedNames("execute");
    advisor.setAdvice(new PerformanceMethodInterceptor());
    weaver.addAdvisor(advisor);
    // 设置为通过cglib获取代理对象
    weaver.setProxyTargetClass(true);
    ITask proxy = (ITask) weaver.getProxy();
    proxy.execute(null);
    System.out.println(proxy.getClass());
}
@Test
public void testClass() {
    ProxyFactory weaver = new ProxyFactory(new Executable());
    NameMatchMethodPointcutAdvisor advisor = new NameMatchMethodPointcutAdvisor();
    advisor.setMappedName("execute");
    advisor.setAdvice(new PerformanceMethodInterceptor());
    weaver.addAdvisor(advisor);
    Executable proxy = (Executable)weaver.getProxy();
    proxy.execute();
    System.out.println(proxy.getClass());
}
```

#### ProxyFactoryBean



#### 基于注解的Spring AOP



#### 总结

`AspectJProxyFactory,ProxyFactoryBean,ProxyFactory` 大体逻辑都是：

1. 填充`AdvisedSupport`（`ProxyCreatorSupport`是其子类）的，然后交给父类`ProxyCreatorSupport`。
2. 得到`JDK或者CGLIB的AopProxy`
3. 代理调用时候被invoke**或者**intercept方法拦截 （分别在`JdkDynamicAopProxy`和`ObjenesisCglibAopProxy（CglibAopProxy的子类）`的中） 并且在这两个方法中调用`ProxyCreatorSupport`的`getInterceptorsAndDynamicInterceptionAdvice`方法去初始化`advice`和各个方法**直接映射关系并缓存**

## AOP在Spring中如何实现的

基于JDK的动态代理和CGLIB两种形式。

## 总结

初略的结合源码阐述了AOP在Spring中的应用，虽然有很多的细节没有涉及，但是对于整体而言，还是算是清晰。

