---
title: lombok介绍
date: 2020-10-02 20:08:21
tags: 基础
categories: java
---

## Lombok使用

> Lombok是一款好用顺手的工具，值得去好好去熟练使用和深究。**Lombok是一种Java™实用工具，可用来帮助开发人员消除Java的冗长代码，尤其是对于简单的Java对象（POJO）。它通过注释实现这一目的**。通过在开发环境中实现Lombok，开发人员可以节省构建诸如hashCode()和equals()这样的方法以及以往用来分类各种accessor和mutator的大量时间。

<!--more-->

## 使用步骤

1. 导入Maven依赖

   ```xml
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>${lomok.version}</version>
   </dependency>
   ```

2. 在IDE中安装对应的插件（**lombok plugin**）

## 常用注解详解

### 简化代码，增加可读性

#### @Getter&@Setter

```java
@Getter
@Setter
public class Person {
    private Integer id;
    private String name;
    private String age;
}
```

通过`@Getter和@Setter`之后，其就类似于如下的代码：

```java
public class Persons {
    private Integer id;
    private String name;
    private String age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }
}
```

#### @Data

其等同于如下的几个注解，使用的频率最高

- @Getter/@Setter
- @ToString
- @EqualsAndHashCode
- @RequiredArgsConstructor



#### @Buider

自动生成流式set写法，能够快速的设定`Object`值。但是需要注意的是，虽然该方法很好，但是因为大多数的框架都使用到了`set`方法进进行注入，**所以在开发的时候，我们一般是将`@Builder`和`@Data`一起使用**。

```java
// 方便查看输出对象
@ToString
@Builder
public class Person {
    private Integer id;
    private String name;
    private String age;

    public static void main(String[] args) {
        Person person = Person.builder().age("11").id(1).name("wkk").build();
        System.out.println(person);
    }
}

//Builder的使用等同于如下
public static class PersonBuilder {
        private Integer id;
        private String name;
        private String age;

        PersonBuilder() {
        }

        public Person.PersonBuilder id(Integer id) {
            this.id = id;
            return this;
        }

        public Person.PersonBuilder name(String name) {
            this.name = name;
            return this;
        }

        public Person.PersonBuilder age(String age) {
            this.age = age;
            return this;
        }

        public Person build() {
            return new Person(this.id, this.name, this.age);
        }

        public String toString() {
            return "Person.PersonBuilder(id=" + this.id + ", name=" + this.name + ", age=" + this.age + ")";
        }
    }
```



### 日志使用

一般情况下，我们做日志处理，都会先生成一个Logger的静态常量，使用`@Slf4j`之后，我们就可以完全不用new一个该常量了。

 ```java
public class Person{
    private static final Logger logger = LoggerFactory.getLogger(Person.class);
    private Integer id;
    private String name;
    private String age;
  
  	public static void main(String[] args){
      	logger.info("logger: {}", "aaaa");
    }
}

// Lombok实现
@Slf4j
public class Person {
    private Integer id;
    private String name;
    private String age;

    public static void main(String[] args) {
        System.out.println("fsda");
        log.info("Person: {}", "fdsa");
    }
}
 ```



### ~~流式对象的关闭~~



