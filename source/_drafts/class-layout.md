---
title: class-layout
mathjax: false
tags: jvm
categories: java
---

## 序

对于一个Java文件，如果要被JVM所运行，首先需要先被编译成class文件，然后JVM才能继续后续的加载、链接、初始化，才正式的进入使用环节。那么Class文件具体的布局内容是什么样的呢？其格式类型肯定要遵循JVM的某种要求，这样才能被正常识别。Redis中的AOF文件、RDB文件同理。

## 布局

上述提了需要遵循JVM的要求，生成符合规范的class文件。转变成class文件一个目的就是：瘦身。在包含所有的信息的情况下，要使的文件的大小尽可能小。如果只是简单的去除空格肯定是不行的。

> 其仍然是个文本文件，瘦身是让其使用一个紧凑的数据结构来表示自身Java 文件里的信息，然后告诉JVM这个数据结构中每个字节都代表什么。

### 举例说明：

```java
public class FlashObject {

    private String name;
    private int age;
    
    public String getName() {
        return name;
    }

    public int add(int a, int b) {
        return a + b;
    }

}
```



### 类信息

### 常量池





## 详解



## 总结

## references

* 深入分析Java web技术内幕
* 深入理解Java虚拟机
* [你管这破玩意儿叫Class](https://mp.weixin.qq.com/s/pekAvJY84qSefHi69d3qgw)

