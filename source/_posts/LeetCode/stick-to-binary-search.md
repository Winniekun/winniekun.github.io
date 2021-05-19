---
title: 死磕二分查找
date: 2020-05-23 14:21:39
tags: 
   - 二分搜索
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png
---

## 前言

LeetCode死磕系列六：**二分查找**

<!--more-->

二分查找是计算机科学中最基本、最有用的算法之一，在基础算法的学习中是非常重要的。

二分查找的最基本问题是在有序数组里查找一个特定的元素

> - 二分查找算法是典型的「减治思想」的应用，我们使用二分查找将待搜索的区间逐渐缩小，以达到「缩减问题规模」的目的；
> - 掌握二分查找的两种思路：
>   - 思路 1：在循环体内部查找元素：`while (left <= right)`；
>   - 思路 2：在循环体内部排除元素：`while (left < right)`。
> - 全部使用**左闭右闭**区间，不建议使用左闭右开区间，反而使得问题变得复杂；

### 应用

1、在半有序（旋转有序或者是山脉）数组里查找元素；

2、确定一个有范围的整数；

3、需要查找的目标元素满足某个特定的性质。

### 模板一

```java
class Solution {

    public int search(int[] nums, int target) {
        // 特殊用例判断
        int len = nums.length;
        if (len == 0) {
            return -1;
        }
        // 在 [left, right] 区间里查找 target
        int left = 0;
        int right = len - 1;
        while (left <= right) {
            // 为了防止 left + right 整形溢出，写成如下形式
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] > target) {
                // 下一轮搜索区间：[left, mid - 1]
                right = mid - 1;
            } else {
                // 此时：nums[mid] < target
                // 下一轮搜索区间：[mid + 1, right]
                left = mid + 1;
            }
        }
        return -1;
    }
}

```

- lower_bound：查找第一个大于或等于 target 的数字；
- upper_bound：查找第一个大于 target 的数字。

这一类问题的描述经常让人觉得头晕，使用模板一，就需要考虑返回 `left` 还是 `right`。

如果使用模板二，由于退出循环以后一定有 `left == right`，就只需要单独判断 `left`或`right` 是否满足题意。

### 模板二

```java
public int search(int[] nums, int left, int right, int target) {
    while (left < right) {
        // 选择中位数时下取整
        int mid = left + (right - left) / 2;
        if (check(mid)) {
            // 下一轮搜索区间是 [mid + 1, right]
            left = mid + 1
        } else {
            // 下一轮搜索区间是 [left, mid]
            right = mid
        }
    }
    // 退出循环的时候，程序只剩下一个元素没有看到。
    // 视情况，是否需要单独判断 left（或者 right）这个下标的元素是否符合题意
}


public int search(int[] nums, int left, int right, int target) {
    while (left < right) {
        // 选择中位数时上取整
        int mid = left + (right - left + 1) / 2;
        if (check(mid)) {
            // 下一轮搜索区间是 [left, mid - 1]
            right = mid - 1;
        } else {
            // 下一轮搜索区间是 [mid, right]
            left = mid;
        }
    }
    // 退出循环的时候，程序只剩下一个元素没有看到。
    // 视情况，是否需要单独判断 left（或者 right）这个下标的元素是否符合题意
}

```

* 特征：`while (left < right)`，这里使用严格小于 `<` 表示的临界条件是：当区间里的元素只有 2 个时，依然可以执行循环体。换句话说，退出循环的时候一定有 `left == right` 成立，**这一点在定位元素下标的时候极其有用**；
* 在循环体中，先考虑 `nums[mid]` 在满足什么条件下不是目标元素，进而考虑两个区间 `[left, mid - 1]` 以及 `[mid + 1, right]` 里元素的性质，目的依然是确定下一轮搜索的区间；
* 根据边界情况，看取中间数的时候是否需要上取整(**也就是mid 是否的计算是否需要加1**)

**注意事项：**

- 先写分支，再决定中间数是否上取整；
- 在使用多了以后，就很容易记住，只要看到 `left = mid` ，它对应的取中位数的取法一定是 `int mid = left + (right - left + 1) / 2;`。



## LeetCode 二分查找题目整理

* 704 二分查找 (**基础,经典应用**)
* 35 插入搜索位置
* 69  x的平方根
* 34 在排序数组中查找元素的第一个和最后一个位置(**经典, 很好的模板**)



## 题解

### [704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

- 关于写`left = mid + 1;`还是写`right = mid - 1;` 上感到困惑，一个行之有效的思考策略是：

  **永远去想下一轮目标元素应该在哪个区间里**:

  - 如果目标元素在区间 `[left, mid - 1]` 里，就需要设置设置 `right = mid - 1`；
  - 如果目标元素在区间 `[mid + 1, right]` 里，就需要设置设置 `left = mid + 1`；

- 考虑不仔细是初学二分法容易出错的地方，这里切忌跳步，需要仔细想清楚每一行代码的含义；

- 循环可以继续的条件是 `while (left <= right)`，特别地，当 `left == right` 即当待搜索区间里只有一个元素的时候，查找也必须进行下去；

```java
public int search(int[] nums, int target) {
    int i = 0;
    int j = nums.length - 1;
    while (i <= j) {
        // 防止整型溢出
        // int mid = i + (j-i)/2;
        int mid = (i+j)/2;
        if(nums[mid] == target){
            return mid;
        }else if(nums[mid] < target){
            i = mid+1;
        }else if(nums[mid] > target){
            j = mid-1;
        }
    }
    return -1;
}
```

### [35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

使用上述的模板来的话,需要考虑是返回i, 还是j 使用模板二的话,不用考虑

```java
// 模板一解法
public static int searchInsert(int[] nums, int target) {
    if(target > nums[nums.length-1]){
        return nums.length;
    }
    int low = 0;
    int high = nums.length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if(target == nums[mid]){
            return mid;
        }else if(nums[mid] < target){
            low = mid+1;
        }else {
            high = mid-1;
        }
    }
    return high;
}
```



```java
// 模板二解法
public static int searchInsertII(int[] nums, int target) {
    int len = nums.length;
    if (len == 0) {
        return 0;
    }
    // 特判
    if (nums[len - 1] < target) {
        return len;
    }
    int left = 0;
    int right = len - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        // 严格小于 target 的元素一定不是解
        if (nums[mid] < target) {
            // 下一轮搜索区间是 [mid + 1, right]
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}
```

### [69. x 的平方根](https://leetcode-cn.com/problems/sqrtx/)

```java
public int mySqrt(int x) {
    if (x == 0) {
        return 0;
    }
    if (x == 1) {
        return 1;
    }
    int i = 1;
    int j = x / 2;
    while (i < j) {
        int mid = (i + j + 1) / 2;
        // 大于一定无
        if (mid > x / mid){
            // 下一轮搜索的区间是 [left, mid - 1]
            j = mid -1;
        }else {
            // 下一轮搜索的区间是 [mid, right]
            i = mid;
        }
    }
    return i;
}
```

### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

```java
public int[] searchRange(int[] nums, int target) {
    int i = 0;
    int j = nums.length - 1;
    int first = findFirstPosition(nums, target);
    if(first == 1){
        return new int[]{-1, -1};
    }
    int secondPosition = findSecondPosition(nums, target);
    return new int[]{first, secondPosition};
}
public int findFirstPosition(int[] nums, int target) {
    int i = 0;
    int j = nums.length - 1;
    while (i < j) {
        int mid = (i + j) / 2;
        // 小于一定没有解
        if (nums[mid] < target) {
            i = mid + 1;
        } else {
            j = mid;
        }
    }
    if (nums[i] == target) {
        return i;
    } else {
        return -1;
    }
}
public int findSecondPosition(int[] nums, int target) {
    int i = 0;
    int j = nums.length - 1;
    while (i < j){
        int mid = (i + j + 1)/2;
        if(nums[mid] > target){
            j = mid-1;
        }else {
            i = mid;
        }
    }
    if (nums[i] == target){
        return i;
    }else {
        return -1;
    }
}
```

### [278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/)

若是中间位置的版本为false, 说明第一个错误版本在[mid+1, right];

```java
public int firstBadVersion(int n) {
    int left = 1, right = n;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (!isBadVersion(mid)) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}
```





## References

* [二分查找算法](https://liweiwei1419.github.io/AlgoWiki/#/BinarySearch/01-introduction)

* [特别好用的二分查找法模板（第 2 版）](https://www.liwei.party/2019/06/17/leetcode-solution-new/search-insert-position/)

