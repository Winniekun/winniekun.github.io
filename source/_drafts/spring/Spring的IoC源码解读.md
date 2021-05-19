# IoC源码解读

## 一些特性简介

### autowire

自动注入，通过autowire，我们可以略去很多人工显示的配置bean的操作，被依赖的发现和注入都交给了Spring，同时其有几个类型可选择，比如byName、byType等。

### FactoryBean

BeanFactory和FactoryBean傻傻分不清？虽然两者类似，但是功能不同。FactoryBean是一种工厂Bean，和普通的Bean不一样，FactoryBean是可以生产Bean的Bean。

### depends-on

当一个 bean 直接依赖另一个 bean，可以使用 `<ref/>` 标签进行配置。不过如某个 bean 并不直接依赖于其他 bean，但又需要其他 bean 先实例化好，这个时候就需要使用 depends-on 特性了。depends-on 特性比较简单，就不演示了。仅贴一下配置文件的内容，如下：

这里有两个简单的类，其中 Hello 需要 World 在其之前完成实例化。相关配置如下：

```xml
<bean id="hello" class="xyz.coolblog.depnedson.Hello" depends-on="world"/>
<bean id="world" class="xyz.coolblog.depnedson.World" />
```

### BeanPostProcessor

BeanPostProcessor 是 bean 实例化时的后置处理器，包含两个方法，其源码如下：

```java
public interface BeanPostProcessor {
    // bean 初始化前的回调方法
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

    // bean 初始化后的回调方法    
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

BeanPostProcessor是Spring框架的一个扩展点，可以通过BeanPostProcessor接口，我们能够插手Bean的实例化过程，比如AOP就是在bean实例将切面逻辑织入bean实例中，AOP也是正是通过BeanPostProcessor和IoC容器建立了联系

#### 示例

```java
@Slf4j
public class LoggerBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        log.info("Before  " + beanName + " Initialization");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        log.info("After  " + beanName + " Initialization");
        return bean;
    }
}


```

**配置如下**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.titizz.simulation.toyspring.learninng.LoggerBeanPostProcessor"/>

    <bean id="car" class="com.titizz.simulation.toyspring.Car">

    </bean>
</beans>
```

**测试如下**

```java
@Test
public void testBeanPostProcessor() {
    String xmlPath = "test-spring.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
}
```

![image-20210502181644978](https://i.loli.net/2021/05/02/iUgIOGvfmPo1ws5.png)

## Resource

将配置文件读入到内存中，供整个Spring应用使用，发生在应用正式执行前的预处理。

## BeanDefinition

**功能：**用于解决Bean的定义问题，包括Bean的名字，类型以及它的属性赋予了什么值或者引用。说白了也就是**解决了在IoC容器中定义一个Bean，使得IoC容器可以根据这个定义来生成实例。**

| 类名                           | 说明                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `BeanDefinition`               | 该类保存了 `Bean` 定义。包括 `Bean` 的 **名字** `String beanClassName`、**类型** `Class beanClass`、**属性** `PropertyValues propertyValues`。根据其 **类型** 可以生成一个类实例，然后可以把 **属性** 注入进去。`propertyValues` 里面包含了一个个 `PropertyValue` 条目，每个条目都是键值对 `String` - `Object`，分别对应要生成实例的属性的名字与类型。在 Spring 的 XML 中的 `property` 中，键是 `key` ，值是 `value` 或者 `ref`。对于 `value` 只要直接注入属性就行了，但是 `ref` 要先进行解析。`Object` 如果是 `BeanReference` 类型，则说明其是一个引用，其中保存了引用的名字，需要用先进行解析，转化为对应的实际 `Object`。 |
| `BeanDefinitionReader`         | 解析 `BeanDefinition` 的接口。通过 `loadBeanDefinitions(String)` 来从一个地址加载类定义。 |
| `AbstractBeanDefinitionReader` | 实现 `BeanDefinitionReader` 接口的抽象类（未具体实现 `loadBeanDefinitions`，而是规范了 `BeanDefinitionReader` 的基本结构）。内置一个 `HashMap rigistry`，用于保存 `String` - `beanDefinition` 的键值对。内置一个 `ResourceLoader resourceLoader`，用于保存类加载器。用意在于，使用时，只需要向其 `loadBeanDefinitions()` 传入一个资源地址，就可以自动调用其类加载器，并把解析到的 `BeanDefinition` 保存到 `registry` 中去。 |
| `XmlBeanDefinitionReader`      | 具体实现了 `loadBeanDefinitions()` 方法，从 XML 文件中读取类定义。 |

## BeanFactory的作用

以 `BeanFactory` 接口为核心发散出的几个类，都是用于解决 IoC 容器在 **已经获取 `Bean` 的定义的情况下，如何装配、获取 `Bean` 实例** 

的问题。

| 类名                         | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| `BeanFactory`                | 接口，标识一个 IoC 容器。通过 `getBean(String)` 方法来 **获取一个对象** |
| `AbstractBeanFactory`        | `BeanFactory` 的一种抽象类实现，规范了 IoC 容器的基本结构，但是把生成 `Bean` 的具体实现方式留给子类实现。IoC 容器的结构：`AbstractBeanFactory` 维护一个 `beanDefinitionMap` 哈希表用于保存类的定义信息（`BeanDefinition`）。获取 `Bean` 时，如果 `Bean` 已经存在于容器中，则返回之，否则则调用 `doCreateBean` 方法装配一个 `Bean`。（所谓存在于容器中，是指容器可以通过 `beanDefinitionMap` 获取 `BeanDefinition` 进而通过其 `getBean()` 方法获取 `Bean`。） |
| `AutowireCapableBeanFactory` | 可以实现自动装配的 `BeanFactory`。在这个工厂中，实现了 `doCreateBean` 方法，该方法分三步：1，通过 `BeanDefinition` 中保存的类信息实例化一个对象；2，把对象保存在 `BeanDefinition` 中，以备下次获取；3，为其装配属性。装配属性时，通过 `BeanDefinition` 中维护的 `PropertyValues` 集合类，把 `String` - `Value` 键值对注入到 `Bean` 的属性中去。如果 `Value` 的类型是 `BeanReference` 则说明其是一个引用（对应于 XML 中的 `ref`），通过 `getBean` 对其进行获取，然后注入到属性中。 |



## ApplicationContext

以 `ApplicationContext` 接口为核心发散出的几个类，主要是对前面 `Resouce` 、 `BeanFactory`、`BeanDefinition` 进行了功能的封装，解决 **根据地址获取 IoC 容器并使用** 的问题。

| 类名                           | 说明                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `ApplicationContext`           | 标记接口，继承了 `BeanFactory`。通常，要实现一个 IoC 容器时，需要先通过 `ResourceLoader` 获取一个 `Resource`，其中包括了容器的配置、`Bean` 的定义信息。接着，使用 `BeanDefinitionReader` 读取该 `Resource` 中的 `BeanDefinition` 信息。最后，把 `BeanDefinition` 保存在 `BeanFactory` 中，容器配置完毕可以使用。注意到 `BeanFactory` 只实现了 `Bean` 的 **装配、获取**，并未说明 `Bean` 的 **来源** 也就是 `BeanDefinition` 是如何 **加载** 的。该接口把 `BeanFactory` 和 `BeanDefinitionReader` 结合在了一起。 |
| `AbstractApplicationContext`   | `ApplicationContext` 的抽象实现，内部包含一个 `BeanFactory` 类。主要方法有 `getBean()` 和 `refresh()` 方法。`getBean()` 直接调用了内置 `BeanFactory` 的 `getBean()` 方法，`refresh()` 则用于实现 `BeanFactory` 的刷新，也就是告诉 `BeanFactory` 该使用哪个资源（`Resource`）加载类定义（`BeanDefinition`）信息，该方法留给子类实现，用以实现 **从不同来源的不同类型的资源加载类定义** 的效果。 |
| ClassPathXmlApplicationContext | 从类路径加载资源的具体实现类。内部通过 `XmlBeanDefinitionReader` 解析 `UrlResourceLoader` 读取到的 `Resource`，获取 `BeanDefinition` 信息，然后将其保存到内置的 `BeanFactory` 中。 |

> 对 `ApplicatinoContext` 的分层更为细致。`AbstractApplicationContext` 中为了实现 **不同来源** 的 **不同类型** 的资源加载类定义，把这两步分层实现。以“从类路径读取 XML 定义”为例，首先使用 `AbstractXmlApplicationContext` 来实现 **不同类型** 的资源解析，接着，通过 `ClassPathXmlApplicationContext` 来实现 **不同来源** 的资源解析。



# 详细步骤

## 获取单例Bean实例

获取单例Bean，主要就是`BeanFactory`的`getBean(String)`方法实现细节。整体的思路如下

![image-20210502212230323](https://i.loli.net/2021/05/02/ebRNXHVq3rcZFL7.png)

## 创建Bean实例

获取bean的方法为`getBean(....)`方法，对于已经实例化好的bean，`getBean`方法并不会再一次去创建，而是从缓存中获取。如果一个bean还未实例化，这个时候bean就无法命中缓存，此时就要根据BeanDefinition去创建这个bean了。



### 创建Bean实例的入口

### 创建Bean实例的方法

### 构造策略

### 循环依赖解决方式

### 属性填充

## Bean实例后序的初始化过程

