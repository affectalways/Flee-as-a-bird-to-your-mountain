# [剑指 Offer II 086. 分割回文子字符串](https://leetcode.cn/problems/M99OJA/)

给定一个字符串 s ，请将 s 分割成一些子串，使每个子串都是 回文串 ，返回 s 所有可能的分割方案。

回文串 是正着读和反着读都一样的字符串。

 

**示例 1：**

```
输入：s = "google"
输出：[["g","o","o","g","l","e"],["g","oo","g","l","e"],["goog","l","e"]]
```

**示例 2：**

```
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
```

**示例 3：**

```
输入：s = "a"
输出：[["a"]]
```

**提示：**

- 1 <= s.length <= 16
- s 仅由小写英文字母组成


注意：本题与主站 131 题相同： https://leetcode-cn.com/problems/palindrome-partitioning/



## 解题思路

```
回溯 + 马拉车算法
```

