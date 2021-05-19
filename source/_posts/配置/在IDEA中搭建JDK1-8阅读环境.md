---
title: 在IDEA中搭建JDK1.8阅读环境
date: 2020-01-15 20:27:15
tags: 配置
categories: 源码 
thumbnail:
---

## 前言

自己从去年12月份开始看JDK上的部分源码，之前因为是懒，觉得看源码嘛，看嘛，突出看。直接就是阅读源码，然后每个代码逻辑都是靠自己推测， 还有理解， 并没有看输出， 现在才发现在`IDEA`中`File | Settings | Build, Execution, Deployment | Debugger | Stepping`中可以将`Do not step into the classes`即可，当然，若是想直接在源码中添加注释，需要进行一些配置。

<!--more-->

## JDK1.8在IDEA中搭建阅读环境

### 第一步

解压系统JDK所在路径中的`sc.zip` 到自定义的文件中，譬如`src`中的`source`

![jdk-项目结构.png](https://i.loli.net/2020/03/25/s17ALRXZBvarG54.png)



### 第二步

导入到自己的源码工程文件中

### 第三步

修改工程文件中`SDKs`的`Sourcepath`  将`sc,zip`替换成图一中`sc.zip`解压到的文件。`javafx-src.zip`不动。如下图：

![jdk-sdks.png](https://i.loli.net/2020/03/25/5GUS1DymJxOMTCj.png)



![jdk-lib.png](https://i.loli.net/2020/03/25/DrORwTvnxWfJVeB.png)



## References

* [JDK1.8源码分析03之idea搭建源码阅读环境](https://www.jianshu.com/p/c00db010265b)
* [idea中搭建jdk1.8源码阅读环境](https://blog.csdn.net/u010999809/article/details/101922489)

