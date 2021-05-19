---
title: 死磕单调栈
date: 2020-06-14 19:56:11
tags: 
   - 单调栈
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png
---

## 单调栈

前几天刷每日一题的时候, 遇到了一个很有趣的题, 刚开始自己直接暴力AC了, 然后优化的时候, 发现了单调栈这么个特殊的数据结构, 很有趣.

首先, 单调栈的作用范围不大, 只适合用来做一类题目--`Next Greater Element` 



## 题目

### [接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

思路参考[甜姨题解](https://leetcode-cn.com/problems/trapping-rain-water/solution/dan-diao-zhan-jie-jue-jie-yu-shui-wen-ti-by-sweeti/)

```java
public int trap(int[] height) {
        // 思路 维护一个最小栈
        if (height == null || height.length == 0) {
            return 0;
        }
        int ans = 0;
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < height.length; i++) {
            while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
                int curIdx = stack.pop();
                while(!stack.isEmpty() && height[stack.peek()] == height[curIdx]) {
                    stack.pop();
                }
                if (!stack.isEmpty()) {
                    int left = stack.peek();
                    int len = i - left -1 ;
                    int high = Math.min(height[i], height[left]) - height[curIdx];
                    ans += len * high;
                }
            }
            stack.push(i);
        }
        return ans;
    }
```



