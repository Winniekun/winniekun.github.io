---
title: Spring的bean注入方式
date: 2019-02-14 19:52:23
tags: spring
categories: spring
thumbnail: https://i.loli.net/2020/04/15/s91ziSuFQLRVEvH.png
---

## Bean的依赖注入方式

<!--more-->

![spring-bean.jpg](https://i.loli.net/2020/04/15/2OFWjfrC9NAyD14.jpg)

Bean配置信息定义了Bean的**实现及依赖关系**，Spring容器根据各种形式的Bean配置信息在容器内部建立Bean定义注册表，然后根据注册表加载、实例化Bean，并建立Bean和Bean的依赖关系，最后将这些准备就绪的Bean放到Bean缓存池中，以供外层的应用程序进行调用。

Bean的注入方式有两种：

1. XML中配置
   1. 构造方法注入
   2. 属性注入
   3. 工厂方法注入
2. 使用注解的方式注入

## XML中的配置依赖注入

### 构造方法注入

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;

    // 利用构造方法注入
    public UserServiceImpl(UserDao userDao){
        this.userDao = userDao;
    }
//    //利用set进行动态实现值的注入
//    public void setUserDao(UserDao userDao) {
//        this.userDao = userDao;
//    }

    public void getUser() {
        userDao.getUser();
    }

}
```

#### bean.xml配置

```xml
<bean id="userImpl" class="com.wkk.dao.impl.UserDaoImpl"/>
    <!-- collaborators and configuration for this bean go here -->
<bean id="mysqlImpl" class="com.wkk.dao.impl.UserDaoMysqlImpl"/>
<bean id="userServiceImpl" class="com.wkk.service.impl.UserServiceImpl">
       <!--<property name="userDao" ref="mysqlImpl"/>-->
    <constructor-arg ref="mysqlImpl"/>
</bean>
```



### 属性注入(setter方法)

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;

    //     利用set进行动态实现值的注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void getUser() {
    //      System.out.println("UserService实现类: 对用户数据的逻辑操作(用户角度)");
        userDao.getUser();
    }

}
```

#### bean.xml配置

```xml
<bean id="userImpl" class="com.wkk.dao.impl.UserDaoImpl"/>
    <!-- collaborators and configuration for this bean go here -->
<bean id="mysqlImpl" class="com.wkk.dao.impl.UserDaoMysqlImpl"/>
<bean id="userServiceImpl" class="com.wkk.service.impl.UserServiceImpl">
   <property name="userDao" ref="mysqlImpl"/>
</bean>
```



### ~~工厂方式注入~~

不写了，文档上没遇到， 然后平时感觉也没怎么遇到过。后面遇到再写

## 使用注解方式注入

### 使用@Autowired自动注入

```java
@Component
public class UserService {
    @Autowired
    @Qualifier("userDaoImpl2")
    private UserDao userDao;

    public void say(){
        userDao.say();
    }
}

@Component
public class UserDaoImpl1 implements UserDao {
    public void say() {
        System.out.println("这是UserDao的第一种实现方式");
    }
}


@Component
public class UserDaoImpl2 implements UserDao {
    public void say() {
        System.out.println("这是UserDao的第二种实现方式");
    }
}
```

#### 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = JavaConfig.class)
public class UserServiceTest {

    @Autowired
    UserService userService;
    @Test
    public void say() {
         Assert.assertNotNull(userService);
    }
}
```

`@Autowired`不仅能对属性进行注解，还可以对方法、构造函数等。`@Autowired`类似与`xml`中使用构造方法注入或属性注入

```xml
<property name="userDao" ref="mysqlImpl"/>
  or
<constructor-arg ref="mysqlImpl"/>
```

**如果使用了@Autowired或者Resource等，就不需要在定义Bean时使用属性注入和构造方法注入了**。意思就是使用注解，可以省略繁杂的xml的配置。

@Autowired注解在setter方法上和属性注入的区别就是：

**setter方法注入是setter方法+bean.xml配置，@Autowired方式省去了bean.xml的配置**

## References

* Spring官方文档

