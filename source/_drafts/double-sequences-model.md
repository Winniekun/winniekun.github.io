---
title: two-sequences-model
mathjax: false
tags: 动态规划
categories: LeetCode
---

## 序

**动态规划之---双序列模型**

**提示：**

> 给定两个序列，然后对它们两个搞事情

**套路：**

> 定义dp\[i][j]：表示s[1: i]和t[1: j]的子问题求解
>
> 千方百计的将dp\[i][j]往dp\[i-1][j]、dp\[i][j-1]、dp\[i-1][j-1]转移



## 题目&题解

### [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

1. 如果p[j]是字母，那么s[i]必须是与其相同的字母才行。所以写成表达式 `dp[i][j] = (s[i]==p[j] && dp[i-1][j-1])`

2. 如果p[j]是'.'，那么s[i]可以是任意字母。所以写成表达式 `dp[i][j] = dp[i-1][j-1]`

3. 如果p[j]是'*'，那么星号的作用有两种情况。
   1. 第一种可以认为是重复0次，即考虑s[1..i]和p[1..j-2]是否匹配。那么记录 `prob1 = dp[i][j-2]`
   2. 第二种可以认为是重复了若干次。不管重复了几次，都要求s[i]必须是与p[j-1]相匹配的字母。这说明什么呢？刨除s[i]之外，前面的字符串也必须与p[1..j]匹配。所以记录 `prob2 = (s[i]==p[j-1]||p[j-1]=='.') && dp[i-1][j]`. 注意之前说的“s[i]必须是与p[j-1]相匹配”，不仅仅指二者是相同的字母，后者也可以是'.'。
   3. 那么综上，第3点情况下，dp[i][j] = prob1 || prob2

接下来我们考虑边界条件。我们注意到上面框架中有几处可能的越界情况：

1. 当j\==1时，dp\[i][j-2]的第二个下标未定义。但是我们发现当j==1时，p[j]不可能是星号，否则不是合法的表达式。所以这种边界情况我们不用担心。
2. 当j==2时，dp\[i][j-2]的第二个下标未定义。所以我们需要考虑dp\[i][0]的情况。这种情况下应该都是false，包含在了初始值中。
3. 当i==1时，dp\[i-1][j]的第一个下表未定义。所以我们需要考虑dp\[0][j]的情况。这对于p来说只可能是非常特殊的一类字符，即类似"a*b*c*"这类的，并且这里的星号都代表重复零次。其他任何p的表达式都不可能被parse成为一个空字符串。所以我们只要对这一类特殊情况做判断就行。



```java
public static boolean isMatch(String s, String p) {
    if (s == null || p == null) {
        if (s == null && p == null) {
            return true;
        }
        return false;
    }
    int rows = s.length();
    int cols = p.length();
    s = "#" + s;
    p = "#" + p;
    boolean[][] dp = new boolean[rows + 1][cols + 1];
    // dp[i][j] 以s[i] p[j]结尾，能否匹配
    // * 较为复杂
    // 1. 重复0次    s[i] 和 p[j-2]
    //       式例     aab
    //               c*a*b
    //    *将c这个字符干掉了 所以c*a*b能够匹配aab
    // 2. 重复若干次  s[i] 和 p[j-1] 匹配
    dp[0][0] = true;
    // 特殊条件处理
    // 当i==1时，dp[i-1][j]的第一个下表未定义。所以我们需要考虑dp[0][j]的情况。
    // 这对于p来说只可能是非常特殊的一类字符，即类似"a*b*c*"这类的，
    // 并且这里的星号都代表重复零次。其他任何p的表达式都不可能被parse成为一个空字符串。
    // 所以我们只要对这一类特殊情况做判断就行。
    for (int j = 2; j <= cols; j++) {
        if (p.charAt(j) == '*') dp[0][j] = dp[0][j - 2];
    }
    for (int i = 1; i <= rows; i++) {
        for (int j = 1; j <= cols; j++) {
            if (Character.isLowerCase(p.charAt(j))) {
                dp[i][j] = (s.charAt(i) == p.charAt(j) && dp[i - 1][j - 1]);
            } else if (p.charAt(j) == '.') {
                dp[i][j] = dp[i - 1][j - 1];
            } else if (p.charAt(j) == '*') {
                // 0次
                boolean zero = dp[i][j - 2];
                // 多次
                boolean multi = dp[i - 1][j] && (s.charAt(i) == p.charAt(j - 1) || p.charAt(j - 
                dp[i][j] = zero || multi;
            }
        }
    }
    return dp[rows][cols];
}
```



### [44. 通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)

和上述一直，只不过这里*只能匹配多次

```java
public boolean isMatch(String s, String p) {
    s = "#" + s;
    p = "#" + p;
    int rows = s.length();
    int cols = p.length();
    char[] ss = s.toCharArray();
    char[] pp = p.toCharArray();
    boolean[][] dp = new boolean[rows][cols];
    dp[0][0] = true;
    for (int j = 1; j < cols; j++) {
        if (pp[j] == '*') {
            dp[0][j] = true;
        } else {
            break;
        }
    }
    for (int i = 1; i < rows; i++) {
        for (int j = 1; j < cols; j++) {
            if (Character.isLowerCase(pp[j])) {
                dp[i][j] = (ss[i] == pp[j]) && dp[i-1][j-1];
            } else if (pp[j] == '?') {
                dp[i][j] = dp[i-1][j-1];
            } else if (pp[j] == '*') {
                dp[i][j] = dp[i-1][j] || dp[i][j-1];
            }
        }
    }
    return dp[rows-1][cols-1];
}
```

### [97. 交错字符串](https://leetcode-cn.com/problems/interleaving-string/)

