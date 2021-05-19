---
title: 死磕DFS
date: 2020-02-23 19:59:12
tags: 
   - dfs
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png	
---

## 前言

LeetCode死磕系列三： DFS

刷了那么多题， 感觉DFS应用很广泛哇，遇到排列组合、搜索等问题，理解题意之后，直接无脑`DFS`都能行的通。

<!--more-->

## LeetCode DFS题目整理

* 200 Number of Islands





## 题解

### 200 Number of Islands

数陆地， 其中`1`表示陆地，`0`表示水

**思路**：

使用`dfs`, 从（0,0）开始，顺序`下、上、右、左`

每次访问到的陆地都改为水。这样就略去了标记为数组，减少空间

```java
 private static final int[][] DIRS = new int[][]{{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    private static final char L = '1', W = '0';

    public int numIslands(char[][] grid){
        int count = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == L) {
                    count++;
                    dfs(grid, i, j);
                }
            }
        }
        return count;
    }

    private void dfs(char[][] grid, int x, int y) {
        // 边界判断
        if (x < 0  || x > grid.length-1 || y < 0 || y > grid[0].length-1) {
            return;
        }
        if(grid[x][y] == L){
            grid[x][y] = W;
            for (int[] dir : DIRS) {
                dfs(grid, x+dir[0], y+dir[1]);
            }
        }
    }
```



