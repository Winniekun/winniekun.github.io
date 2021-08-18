---
title: 第二类区间模型
mathjax: false
tags: 区间DP
categories: leetcode
---

## 序



## 题目&题解

### [131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

```
给你一个字符串 s，请你将 s 分割成一些子串，使每个子串都是 回文串 。返回 s 所有可能的分割方案。

回文串 是正着读和反着读都一样的字符串。

 

示例 1：

输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
示例 2：

输入：s = "a"
输出：[["a"]]

```

#### 思路

#### 题解

```java
private List<List<String>> res = new ArrayList<>();
private boolean[][] dp;
private int N;
private String s;
public List<List<String>> partition(String s) {
    // 动归
    // dp[i][j] 区间i-j构成的字符串能够成为回文串
    // 如果是，添加 之后构造回溯dfs
    this.s = s;
    char[] chars = s.toCharArray();
    N = s.length();
    dp = new boolean[N][N];
    // 初始化
    for (int i = 0; i < N; i++) {
        dp[i][i] = true;
    }
    // 奇偶性
     for (int i = 0; i + 1 < N; i++) {
         dp[i][i+1] = chars[i] == chars[i+1];
     }
    for (int len = 3; len <= N; len++) {
        for (int i = 0; i + len - 1 < N; i++) {
            int j = i + len - 1;
            if (chars[i] == chars[j]) {
                dp[i][j] = dp[i+1][j-1];
            } else {
                dp[i][j] = false; // 说明 区间[i:j]不能构成回文串
            }
        }
    }
    // dfs
    Deque<String> tmp = new ArrayDeque<>();
    dfs(0, tmp);
    return res;
}
private void dfs(int starter, Deque<String> tmp) {
    if (starter == N){
        res.add(new ArrayList<>(tmp));
        return;
    }
    for (int j = starter; j < N; j++){
        if (dp[starter][j]){
            tmp.addLast(s.substring(starter, j + 1));
            dfs(j+1, tmp);
            tmp.removeLast();
        }
    }
}
```

### [312. 戳气球](https://leetcode-cn.com/problems/burst-balloons/)

```
有 n 个气球，编号为0 到 n - 1，每个气球上都标有一个数字，这些数字存在数组 nums 中。

现在要求你戳破所有的气球。戳破第 i 个气球，你可以获得 nums[i - 1] * nums[i] * nums[i + 1] 枚硬币。 这里的 i - 1 和 i + 1 代表和 i 相邻的两个气球的序号。如果 i - 1或 i + 1 超出了数组的边界，那么就当它是一个数字为 1 的气球。

求所能获得硬币的最大数量。

 

示例 1：
输入：nums = [3,1,5,8]
输出：167
解释：
nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> []
coins =  3*1*5    +   3*5*8   +  1*3*8  + 1*8*1 = 167
示例 2：

输入：nums = [1,5]
输出：10
```



#### 思路

#### 题解

```java
public int maxCoins(int[] nums) {
    // 戳破s[i:j]的所有气球，最大的硬币数量
    // 🪙
    int n = nums.length;
    int[] array = new int[n + 2];
    System.arraycopy(nums, 0, array, 1, n);
    array[0] = 1;
    array[array.length - 1] = 1;
    int[][] dp = new int[n + 2][n + 2];
    for (int len = 1; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            for (int k = i; k <= j; k++) {
                dp[i][j] = Math.max(dp[i][j], dp[i][k-1] + dp[k+1][j] + array[i-1] * array[k] * array[j+1]);
            }
        }
    }
    return dp[1][n];
}
public static void main(String[] args) {
    int[] nums = {3,1,5,8};
    int len = nums.length;
    int[] array = new int[nums.length + 1];
    array[0] = 1;
    System.arraycopy(nums, 0, array, 1, len);
    for (int i : array) {
        System.out.println(i);
    }
}
```



### [375. 猜数字大小 II](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/)

#### 思路



#### 题解

```java
public int getMoneyAmount(int n) {
    // dp[i][j]表示对于i~j的区间进行猜数游戏所需要的最小代价
    int[][] dp = new int[n+2][n+2];
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE / 2;
            for (int k = i; k <= j; k++) {
                dp[i][j] = Math.min(dp[i][j], k + Math.max(dp[i][k-1], dp[k+1][j]));
            }
        }
    }
    return dp[1][n];
}
```

### [516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

#### 思路

#### 题解

```java
public static int longestPalindromeSubseq(String s) {
    // 照抄问题 dp[i][j] => 字符串S[i:j]里是回文串的最长subsequence的长度。
    // 第一层是空间大小
    // 第二层循环起始点
    int n = s.length();
    s = "#" + s;
    int[][] dp = new int[n+1][n+1];
    for (int i = 1; i <= n; i++) {
        dp[i][i] = 1;
    }
    char[] array = s.toCharArray();
    for (int len = 2; len <= n; len++) { // 先遍历区间
        for (int i = 1; i + len - 1 <= n; i++) { // 后遍历起始位置
            int j = i + len - 1; // 右边端点
            if (array[i] == array[j]) { // 若相等
                dp[i][j] = dp[i+1][j-1] + 2;
            } else {
                int first = dp[i][j-1];
                int second = dp[i+1][j];
                dp[i][j] = Math.max(first, second);
            }
        }
    }
    return dp[1][n];
}
```

### [664. 奇怪的打印机](https://leetcode-cn.com/problems/strange-printer/)

**思路：**

**题解：**

```java
public int strangePrinter(String s) {
    int n = s.length();
    int[][] f = new int[n][n];
    for (int i = 0; i < n; i++) {
        f[i][i] = 1;
    }
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len  <= n; i++) {
            int j = i + len - 1;
            if (s.charAt(i) == s.charAt(j)) {
                f[i][j] = f[i][j - 1];
            } else {
                int minn = Integer.MAX_VALUE;
                for (int k = i; k < j; k++) {
                    minn = Math.min(minn, f[i][k] + f[k + 1][j]);
                }
                f[i][j] = minn;
            }
        }
    }
    return f[0][n - 1];
}
```

### [730. 统计不同回文子序列](https://leetcode-cn.com/problems/count-different-palindromic-subsequences/)

**思路：**



**题解：**

## References

* 动态规划的套路-从入门到精通到弃坑

