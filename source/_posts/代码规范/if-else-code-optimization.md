---
title: if-else代码优化的几种方案
date: 2020-11-02 19:46:42
tags: if-else优化
categories: 代码质量
---

## 前言

代码中涉及到较为复杂的逻辑是，一般情况下会出现大量的`if-else`，虽然实现了整个功能，但是阅读起来比较耗时，并且维护的时候也十分的困难，同时只要有一个地方没有考虑清楚，修改起来就难免出现`bug`。最近在做公司某个节点的性能优化，就遇到了这个问题，然后在`review`的时候，同事和我讲了一些优化方案，以及默认的规范之后。自己重新修改之后，感觉很舒服，后续自己重新整理一下，所以就写了这篇博客。

<!--more-->

## 优化方案

### 优化一：提前return，去除不必要的else

如果`if-else`代码块包含`return`语句，可以考虑通过提前`return`，把多余`else`干掉，使代码更加优雅。

#### 优化前 

```java
if(condition){
  // do something
} else {
  // do other thing
  return;
}
```



#### 优化后

```java
if(!condition) {
  return;
} 
// do something
```



### 优化二：使用三元运算

#### 优化前

```java
int price;
if(condition){
  price = 10;
} else {
  price = 20;
}
```

#### 优化后

```java
int price = condition ? 10 : 20;
```



### 优化三：使用枚举

#### 优化前

```java
int orderStatus = getXXX(xxx);
String OrderStatusDes;
if(orderStatus == 0) {
    OrderStatusDes = "订单未支付";
}else if(OrderStatus == 1) {
    OrderStatusDes = "订单已支付";
}else if(OrderStatus == 2) {
   OrderStatusDes = "已发货";
}
```

#### 优化后

**先定义一个枚举类型**

```java
public enum OrderStatusEnum {
    UN_PAID(0,"订单未支付"),PAIDED(1,"订单已支付"),SENDED(2,"已发货"),;
     
    private int index;
    private String desc;
 
    public int getIndex() {
        return index;
    }
 
    public String getDesc() {
        return desc;
    }
 
    OrderStatusEnum(int index, String desc){
        this.index = index;
        this.desc =desc;
    }
 
    OrderStatusEnum ofCode(int orderStatus) {
        for (OrderStatusEnum temp : OrderStatusEnum.values()) {
            if (temp.getIndex() == orderStatus) {
                return temp;
            }
        }
        return null;
    }
}

```

有了枚举之后，以上if-else逻辑分支，可以优化为一行代码

```java
String OrderStatusDes = OrderStatusEnum.ofCode(orderStatus).getDesc();
```



### 优化四：合并条件表达式

如果有一系列条件返回一样的结果，可以将它们合并为一个条件表达式，让逻辑更加清晰。

#### 优化前

```java
double getVipDiscount() {
    if(age<18){
        return 0.8;
    }
    if("深圳".equals(city)){
        return 0.8;
    }
    if(isStudent){
        return 0.8;
    }
    //do somethig
}
```

#### 优化后

```java
double getVipDiscount(){
    if(age<18|| "深圳".equals(city)||isStudent){
        return 0.8;
    }
    //doSomthing
}
```

如果返回统一结果的条件过多，可以再编写一个方法，封装起来:smile:

### 优化五：使用Optional

有时候`if-else`比较多，是因为非空判断导致的，这时候你可以使用`Java9`的`Optional`进行优化。

#### 优化前

```java
String str = "wkk";
if (str != null) {
    System.out.println(str);
} else {
    System.out.println("Null");
}
```

#### 优化后

```java
Optional<String> strOptional = Optional.ofNullable("wkk");
strOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Null"));
```

`Java8`的`Optional`虽然没有这个方法，但是我们可以用Optional来优化非空的赋值操作

**示例：**

电池平台服役状态赋值：如果为null，赋值为0，否则为原来的值

```java
batteryBasicInfoCollectDto.setFlowControlState(Optional.ofNullable(batteryInfo.getFlowControlState()).orElse(0));
```

### 优化六：表驱动法

#### 优化前

**表驱动法**，又称之为表驱动、表驱动方法。表驱动方法是一种使你可以在表中查找信息，而不必用很多的逻辑语句（if或Case）来把它们找出来的方法。以下的demo，把map抽象成表，在map中查找信息，而省去不必要的逻辑语句。

```java
if (param.equals(value1)) {
    doAction1(someParams);
} else if (param.equals(value2)) {
    doAction2(someParams);
} else if (param.equals(value3)) {
    doAction3(someParams);
}
```

#### 优化后

```java
// 这里泛型是为方便演示，实际可替换为你需要的类型
Map<?, Function<?> action> actionMappings = new HashMap<>(); 

// 初始化
actionMappings.put(value1, (someParams) -> { doAction1(someParams)});
actionMappings.put(value2, (someParams) -> { doAction2(someParams)});
actionMappings.put(value3, (someParams) -> { doAction3(someParams)});
 
// 省略多余逻辑语句
actionMappings.get(param).apply(someParams);
```

### 优化七：优化逻辑结构，让正常流程走主干

#### 优化前

```java
public double getAdjustedCapital(){
    if(_capital <= 0.0 ){
        return 0.0;
    }
    if(_intRate > 0 && _duration >0){
        return (_income / _duration) *ADJ_FACTOR;
    }
    return 0.0;
}
```

#### 优化后

```java

public double getAdjustedCapital(){
    if(_capital <= 0.0 ){
        return 0.0;
    }
    if(_intRate <= 0 || _duration <= 0){
        return 0.0;
    }
  
    return (_income / _duration) *ADJ_FACTOR;
}
```

**将条件反转使异常情况先退出，让正常流程维持在主干流程，可以让代码结构更加清晰。**

### 优化八：策略模式+工厂方法消除if-else

**策略类定义**：一个策略接口和一组实现这个接口的策略类。

**特点**：客户端代码基于接口而非实现编程，可以灵活地替换不同的策略。

**假设需求**：根据不同勋章类型，处理相对应的勋章服务

#### 优化前

```java

String medalType = "guest";
if ("guest".equals(medalType)) {
    System.out.println("嘉宾勋章");
} else if ("vip".equals(medalType)) {
    System.out.println("会员勋章");
} else if ("guard".equals(medalType)) {
    System.out.println("展示守护勋章");
}
    
```

#### 优化后

首先，我们把每个条件逻辑代码块，抽象成一个公共的接口，可以得出以下代码：

```java
//勋章接口
public interface IMedalService {
    void showMedal();
}
```

我们根据每个逻辑条件，定义相对应的策略实现类，可得以下代码：

```java
//守护勋章策略实现类
public class GuardMedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("展示守护勋章");
    }
}
//嘉宾勋章策略实现类
public class GuestMedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("嘉宾勋章");
    }
}
//VIP勋章策略实现类
public class VipMedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("会员勋章");
    }
}
```

接下来，我们再定义策略工厂类，用来管理这些勋章实现策略类，如下：

```java
//勋章服务工产类
public class MedalServicesFactory {
 
    private static final Map<String, IMedalService> map = new HashMap<>();
    static {
        map.put("guard", new GuardMedalServiceImpl());
        map.put("vip", new VipMedalServiceImpl());
        map.put("guest", new GuestMedalServiceImpl());
    }
    public static IMedalService getMedalService(String medalType) {
        return map.get(medalType);
    }
}

```

使用了策略+工厂模式之后，代码变得简洁多了，如下：

```java
public class Test {
    public static void main(String[] args) {
        String medalType = "guest";
        IMedalService medalService = MedalServicesFactory.getMedalService(medalType);
        medalService.showMedal();
    }
}
```

