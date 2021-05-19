---
title: 死磕回溯
date: 2020-02-25 22:18:18
tags:
   - 回溯
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png
---

## 前言

LeetCode死磕系列四： 回溯

<!--more-->

1. 组合、排列
2. 切割问题
3. 子集问题
4. 棋盘问题

回溯可抽象为树形结构（n叉树）

for循环嵌套递归

优化： 剪枝操作

需要注意题目中一次的结果中元素能否重复，所有结果中，顺序不同，是否会造成一个新的结果。

## LeetCode 回溯题目整理

排列组合区分：

排列有序，相同元素不同顺序，则不同

组合无序，相同元素不同顺序，则相同(需要添加start索引)

* 77 combine
  * 组合问题

* 78 [Subsets](https://leetcode.com/problems/subsets/description/)
  * 组合问题
* 46 Permutations
  * 排列问题
* 47 PermutationsII
  * 排列问题
* 36 Combination Sum
  * 组合问题
* 40 Combination Sum II
  * 组合问题
* 216 Combination Sum III
  * 组合问题
* 90 Subsets II
  * 组合问题
* 60 Permutation Sequence
  * 和Permutations是一样的，只需要返回results中第k个值即可

---

二维问题

* 79 Word Search
  * 经典的二维问题的回溯
* 剑指offer第12题





## 题解

### 77 [Combine](https://leetcode-cn.com/problems/combinations/)





### 78 [Subsets](https://leetcode.com/problems/subsets/)

组合问题

```java
public static List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> results = new LinkedList<>();
    List<Integer> out = new LinkedList<>();
    backtrack(nums, results, out, 0);
    return results;
}
private static void backtrack(int[] nums, List<List<Integer>> results, List<Integer> track, int start) {
    results.add(new LinkedList<>(track));
    for (int i = start; i < nums.length; i++) {
        track.add(nums[i]);
        backtrack(nums, results, track, i+1);
        track.remove(track.size()-1);
    }
}
```

### 46 [Permutations](https://leetcode.com/problems/permutations/)

排列问题

```java
private List<List<Integer>> results = new LinkedList<>();
public List<List<Integer>> permute(int[] nums) {
    int k = 3;
    List<Integer> out = new LinkedList<>();
    backtrack(nums, out, k);
    return results;
}
private void backtrack(int[] nums, List<Integer> track, int k)
    if(track.size() == k){
        results.add(new LinkedList<>(track));
    }
    for (int i = 0; i < nums.length; i++) {
         // 剔除重复值
         if(track.contains(nums[i])){
                continue;
          }
          track.add(nums[i]);
          backtrack(nums, track, k);
          track.remove(track.size()-1);
    }
}
```

### 47 [PermutationsII](https://leetcode.com/problems/permutations-ii/)

排列问题

```java
private static List<List<Integer>> results = new ArrayList<>();
public static List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);
    List<Integer> out = new ArrayList<>();
    backtrack(nums, out, new boolean[nums.length]);
    return results;
}
private static void backtrack(int[] nums, List<Integer> out, boolean[] visit
    if (out.size() == nums.length) {
        results.add(new ArrayList<>(out));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        //重复元素只按顺序选择，若当前元素未被选择且前一元素与当前元素值相等也未被选择则跳过，
        // 这一可能情况与先选小序号后选大序号的相同元素相同
        if (i > 0 && nums[i] == nums[i - 1] && visited[i-1] &&!visited[i] ||
            continue;
        }
        out.add(nums[i]);
        visited[i] = true;
        backtrack(nums, out, visited);
        out.remove(out.size() - 1);
        visited[i] = false;
    }
}
```



### 36 [Combination Sum](https://leetcode.com/problems/combination-sum/)

组合问题

**元素无重复项**

**可重复使用**

```java
private List<List<Integer>> results = new ArrayList<List<Integer>>();
private List<Integer> curren = new ArrayList<Integer>();
public List<List<Integer>> combinationSum(int[] candidates, int target){
    Arrays.sort(candidates);
    if(candidates == null){
        return results;
    }
    backtrack(0, candidates, target, curren);
    return results;
}
private void backtrack(int start, int[] candidates, int target, List<Integer> out){
    if(target < 0){
        return;
    }
    if(target == 0){
        results.add(new ArrayList<>(out));
    }
    for (int i = start; i < candidates.length; i++) {
        out.add(candidates[i]);
        backtrack(i, candidates, target-candidates[i], curren);
        out.remove(out.size()-1);
    }
}
```



### 40 [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

组合问题

**元素有重复项**

**不能重复选**

```java
// 有重复项
private List<List<Integer>> results = new ArrayList<>();
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    if (candidates == null) {
        return results;
    }
    Arrays.sort(candidates);
    List<Integer> out = new ArrayList<>();
    backtrack(0, out, candidates, target);
    return results;
}
private void backtrack(int start, List<Integer> out, int[] candidates, int targe
    if(target < 0){
        return;
    }
    if(target == 0){
        results.add(new ArrayList<>(out));
    }
    for (int i = start; i < candidates.length; i++) {
        if(candidates[i] == candidates[i-1] && i> start){
            continue;
        }
        out.add(candidates[i]);
        backtrack(i+1, out, candidates, target-candidates[i]);
        out.remove(out.size()-1);
    }
}
```



### 216 [Combination Sum III](https://leetcode.com/problems/combination-sum-iii/)

组合问题

**元素无重复项，但是候选集改变**

```java
private List<List<Integer>> results = new ArrayList<>();
public List<List<Integer>> combinationSum3(int k, int n){
    List<Integer> out = new ArrayList<>();
    backtrack(1, out, n, k);
    return results;
}
private void backtrack(int start, List<Integer> out, int n, int k) {
    if(out.size() == k && n==0){
        results.add(new ArrayList<>(out));
    }
    for (int i = start; i <= n; i++) {
        if(i>=10){
            continue;
        }
        out.add(i);
        backtrack(i+1, out, n-i, k);
        out.remove(out.size()-1);
    }
}
```



### 90 [Subsets II](https://leetcode.com/problems/subsets-ii/)

组合问题

**元素有重复项**

```java
private List<List<Integer>> results = new ArrayList<>()
public List<List<Integer>> subsetsWithDup(int[] nums) {
    if(nums == null){
        return results;
    }
    Arrays.sort(nums);
    List<Integer> out = new ArrayList<>();
    backtrack(nums, out, 0);
    return results;
}
private void backtrack(int[] nums, List<Integer> out, i
    results.add(new ArrayList<>(out));
    for (int i = start; i < nums.length; i++) {
        if(i>start && nums[i] == nums[i-1]){
            continue;
        }
        out.add(nums[i]);
        backtrack(nums, out, i+1);
        out.remove(out.size()-1);
    }
}
```



---

### [79 Word Search](https://leetcode.com/problems/word-search/)

二维平面内判断是否包含有字符串，最基础的二维回溯问题. 和一维回溯问题不同的是，回溯函数去除了for循环，而是判断其相邻的四个格子： `(row+1, col)`  `(row-1, col)`  `(row, col+1)`  `(row, col-1)`

调用回溯的方法使用了for循环，因为二维的每个位置都需要进行深度遍历。

回溯部分：

1. 需要判断是否越界，越界的地方不走
2. 终止条件： 当前的第i步，对应的字符刚好是`word`的第i个字符

```java
public boolean exist(char[][] board, String word) {
    if (board.length == 0 || board[0].length == 0) {
        return false;
    }
    int m = board.length;
    int n = board[0].length;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (search(board, word, 0, i, j)) {
                return true;
            }
        }
    }
    return false;
}
private boolean search(char[][] board, String word, int start, int i, int j) {
    if (start == word.length()) {
        return true;
    }
    int m = board.length;
    int n = board[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n || board[i][j] != word.charAt(start)) {
        return false;
    }
    char c = board[i][j];
    board[i][j] = '#';
    boolean res = search(board, word, start + 1, i + 1, j) ||
            search(board, word, start + 1, i - 1, j) ||
            search(board, word, start + 1, i, j + 1) ||
            search(board, word, start + 1, i, j - 1);
    board[i][j] = c;
    return res;
}
```

