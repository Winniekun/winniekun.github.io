---
title: DFS深入理解
date: 2020-03-22 16:32:35
tags: 回溯
categories: 算法
thumbnail: https://i.loli.net/2020/03/11/MTIjwoH5NOAY3CD.jpg
---

## DFS

DFS的应用十分广泛，用途最多的有以下两个方面：

1. 排列组合
2. 搜索

根据`啊哈！算法`中的介绍，和个人的理解，现做如下的总结

<!--more-->

### 应用一： 排列组合 

形象阐述： **不撞南墙不回头**

对于排列组合，当然可以随机组合，只需要把所有结果都存储即可。为了方便，并且不漏查，需要定义一个规则，按照顺序进行组合。譬如将三个扑克一号、二号、三号分别放入三个盒子中，每个盒子只能放一张扑克，约定规则，任何盒子都先放一号，然后二号，最后三号

```java
void dfs(int step){
    判断边界（记得return）
    尝试每一种可能 for(i=step; i<n; i++){
    	// 当前情况的处理
        继续下一步 dfs(step+1);
    }
    返回
}
```

排列组合的关键就是： `当下该如何做` 至于`下一步如何做`则与`当下该如何做`一样（这不就是递归么:smile:）



### 应用二： 搜索

当前怎么办，对于图上的搜索，先检查是否已到达终点，如果没有到达，则找出下一步可以走的地方。终点P `(p, q)`

当前位置为S `（x, y）`

#### 判断边界

```java
void dfs(int x, int y, int step){
    // 判断边界
    if(x==p && y==q){
		if(step < min){
            min = step;
        }
        return;
    }
}
```

#### 当前该怎么做

首先，在当前的位置，其可以走四个方向，规定顺序，默认顺时针（右、下、左、上），防止错乱

![dfs-图-搜索规则、.png](https://i.loli.net/2020/03/23/S4UgyHTKOFqon61.png)



```java
void dfs(int x, int y, int step){
    // 判断边界
    if(x==p && y==q){
        if(step < min){
            min = step;
        }
        return;
    }
    // 当前该怎么做
    for(k=0; k<=3; k++){
        // 计算下一个点的坐标
        tx = x + next[k][0];
        ty = y + next[k][1];
    }
    
}
```

####  判断是否越界，是否为障碍物，是否在路径中

```java
void dfs(int x, int y, int step){
    // 判断边界
    if(x==p && y==q){
        if(step < min){
            min = step;
        }
        return;
    }
    // 当前该怎么做
    for(k=0; k<=3; k++){
        // 计算下一个点的坐标
        tx = x + next[k][0];
        ty = y + next[k][1];
    }
    // 判断是否越界
    if(tx < 1 || tx > n || ty < 1 || ty > n){
        continue;
    }
    // 判断该点是否为障碍物或者已经在路径中
    if(a[tx][ty] == 0 && book[tx][ty] == 0){
        book[tx][ty] == 1; // 标记改点 已经走过
        dfs(tx, ty, step +1);
        book[tx][ty] == 0; // 尝试结束，取消改点的标记
    }
    
}
```

### 实战

[死磕系列---DFS](https://kongwiki.me/2020/03/23/死磕DFS/)

## references 

* 啊哈！算法
* 程序员的数学
* [pythontutor](http://pythontutor.makerbean.com/#mode=edit)



