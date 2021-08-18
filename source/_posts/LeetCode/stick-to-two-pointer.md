---
title: 死磕双指针
tags: 双指针
categories: leetcode
mathjax: false
date: 2021-05-27 17:10:30
---


## 序

双指针，一个看了脑子里就有画面的算法技巧，主要方式有：

1. 快慢指针
   - 譬如链表环系列问题
   - 获取链表的第K个节点
     - 应该也属于快慢指针吧
     - 只不过是一个指针先移动一定步长，然后另一个两个指针再以同样的速度移动
2. 头尾指针
   - 适用于数组和字符串
   - 根据使用场景，用于优化遍历的时间复杂度
3. 滑动窗口

## 普通指针

### [26. 删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int i = 0;
        int N = nums.length;
        for (int j = 1; j < N; j++) {
            if (nums[i] != nums[j]) {
                nums[++i] = nums[j];
            }
        }
        return i + 1;
    }
}
// 通法

class Solution {
    public int removeDuplicates(int[] nums) {   
        return process(nums, 1);
    }
    int process(int[] nums, int k) {
        int idx = 0; 
        for (int x : nums) {
            if (idx < k || nums[idx - k] != x) nums[idx++] = x;
        }
        return idx;
    }
}
```

### [80. 删除有序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

```java
public int removeDuplicates(int[] nums) {
    return process(nums, 2);
}
private int process(int[] nums, int k) {
    int idx = 0;
    for (int x : nums) {
        if (idx < k || nums[idx - k] != x) {
            nums[idx++] = x;
        }
    }
    return idx;
}
```

### [283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)



### [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

归并排序的中的一个步骤





## 快慢指针

环系列

### [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

```java
public boolean hasCycle(ListNode head) {
    if (head == null) {
        return false;
    }
    ListNode fast = head;
    ListNode slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow) {
            return true;
        }
    }
    return false;
}
```



### [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)



## 头尾指针

可以理解为对整个数组的暴力搜索的优化，正常情况下，是得到所有的区间k的子数组，然后一次计算结果。双指针之后，可以在遍历的过程就的出结果，避免的重复的遍历，并且优化的了遍历的时间复杂度

### [11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

```java
public int maxArea(int[] height) {
    // 双指针(头尾指针)
    int N = height.length;
    int left = 0;
    int right = N - 1;
    int area = 0;
    while (left < right) {
        int hight = Math.min(height[left], height[right]);
        int innerArea = hight * (right - left);
        area = Math.max(area, innerArea);
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return area;
}
```

### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

```java
public List<List<Integer>> threeSum(int[] nums) {
    // 思路
    // 核心思路：双指针 头尾指针
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    int N = nums.length;
    for (int i = 0; i < N - 2; i++) {
        int rst = -nums[i];
        int left = i + 1;
        int right = N - 1;
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        while (left < right) {
            int cur = nums[left] + nums[right];
            if (cur == rst) {
                res.add(new ArrayList<>(Arrays.asList(nums[i], nums[left], nums[right])));
                // 处理后序重复的元素
                while (left < right && nums[left + 1] == nums[left]) {
                    left++;
                }
                while (left < right && nums[right] == nums[right - 1]) {
                    right--;
                }
                left++;
                right--;
            } else if (cur > rst) {
                right--;
            } else {
                left++;
            }
        }
    }
    return res;
}
```

### [16. 最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

和上述类似，不过不用考虑元素重复问题，但是需要考虑最近，所以使用一个变量，用于检测当前结果是否比之前更好。

```java
public int threeSumClosest(int[] nums, int target) {
    int N = nums.length;
    Arrays.sort(nums);
    // 如何判断最近？
    // min(closest, curDistance)
    int closest = Integer.MAX_VALUE;
    int res = nums[0] + nums[nums.length - 1] + nums[1];
    for (int i = 0; i < N - 2; i++) {
        int low = i + 1;
        int height = N - 1;
        while (low < height) {
            int innerCur = nums[i] + nums[low] + nums[height];
            if (innerCur < target) {
                low++;
            } else if (innerCur > target) {
                height--;
            } else {
                return target;
            }
            if (Math.abs(innerCur - target) < closest) {
                closest = Math.abs(innerCur - target);
                res = innerCur;
            }
        }
    }
    return res;
}
```

### [18. 四数之和](https://leetcode-cn.com/problems/4sum/)

同样的思路，不过可以做一些细节处理，提前退出循环。

```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> quadruplets = new ArrayList<>();
    if (nums == null || nums.length < 4) {
        return quadruplets;
    }
    Arrays.sort(nums);
    int length = nums.length;
    for (int i = 0; i < length - 3; i++) {
        // 重复
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        // 不符合条件
        if (nums[i] + nums[i + 1] + nums[i + 2] + nums[i + 3] > target) {
            break;
        }
        // 结果小
        if (nums[i] + nums[length - 3] + nums[length - 2] + nums[length - 1] < target) {
            continue;
        }
        for (int j = i + 1; j < length - 2; j++) {
            // 重复
            if (j > i + 1 && nums[j] == nums[j - 1]) {
                continue;
            }
            // 不符合条件
            if (nums[i] + nums[j] + nums[j + 1] + nums[j + 2] > target) {
                break;
            }
            // 太小
            if (nums[i] + nums[j] + nums[length - 2] + nums[length - 1] < target) {
                continue;
            }
            int left = j + 1, right = length - 1;
            while (left < right) {
                int sum = nums[i] + nums[j] + nums[left] + nums[right];
                if (sum == target) {
                    quadruplets.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left + 1]) {
                        left++;
                    }
                    left++;
                    while (left < right && nums[right] == nums[right - 1]) {
                        right--;
                    }
                    right--;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }
    }
    return quadruplets;
}
```

### [259. 较小的三数之和](https://leetcode-cn.com/problems/3sum-smaller/)

```java
public int threeSumSmaller(int[] nums, int target) {
    Arrays.sort(nums);
    int N = nums.length;
    int count = 0;
    for (int i = 0; i < N - 2; i++) {
        int left = i + 1;
        int right = N - 1;
        while (left < right) {
            int innerCur = nums[i] + nums[left] + nums[right];
            if (innerCur < target) {
                int innerCount = right - left;
                count += innerCount;
                left++;
            } else {
                right--;
            }
        }
    }
    return count;
}
```

### [30. 串联所有单词的子串](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/)



## 滑动窗口

### [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

```java
public int minSubArrayLen(int target, int[] nums) {
    int N = nums.length;
    int left = 0;
    int right = 0;
    int ans = Integer.MAX_VALUE;
    int sum = 0;
    while (right < N) {
        sum += nums[right];
        while (left <= right && sum >= target) {
            ans = Math.min(ans, right - left + 1);
            sum -= nums[left];
            left++;
        }
        right++;
    }
    return ans == Integer.MAX_VALUE ? 0 : ans;
}
```



### [713. 乘积小于K的子数组](https://leetcode-cn.com/problems/subarray-product-less-than-k/)

双指针（滑动窗口）：left 和 right指针开始时均指向 nums数组的开头。

right指针遍历 nums数组，每遍历一个元素就更新一次乘积，即 product = product * nums[right]：

- 若当前更新后的 product < k，则直接统计当前窗口中的元素个数（**这恰好就是当前子数组中符合条件的全部子集个数**）
  - 我们每次都统计以nums[right]为结尾符合条件的子数组个数

- 若更新后的 product >= k，则此时需要缩小窗口的大小，即令当前 product 除以当前 left 指针指向的元素值，然后令 left指针右移一位，直到当前的 produc < k 为止，然后统计当前窗口中的元素个数，即为此时符合条件的子数组的个数，继续令 right 指针右移。
- 如此反复，直到 right 指针指向 nums 数组的末尾为止。

```java
public int numSubarrayProductLessThanK(int[] nums, int k) {
    // 滑动窗口
    int N = nums.length;
    int left = 0;
    int sum = 1;
    int count = 0;
    for (int right = 0; right < N; right++) {
        sum *= nums[right];
        while (left <= right && sum >= k) {
            sum /= nums[left];
            left++;
        }
        count += right - left + 1;
    }
    return count;
}
```

