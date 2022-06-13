# [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]*k[1]*...*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

**示例 1：**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

**提示：**

- `2 <= n <= 58`



## 解题思路

```
动态规划
```

**动规流程**
**dp数组及下标含义**
dp[i]表示长度为i的绳子切割段的最大乘积

**dp数组初始化**
由于要切割的段数最少为2
所以dp[0] = dp[1] = 0, dp[2] = 1

**确定递推顺序**

- 分成两段，即为(i-j) * j
- 分成多段，即为dp[j] * (i-j)

所以dp[i] = max(dp[i], max(dp[j], j)*(i-j))

**返回结果**

**状态转移方程：**

```
dp[i] = max(dp[i], max((i-1) * j, j * dp[i-j])
```

![](https://pic.leetcode-cn.com/82b25ac6bcb742f31e5202e4af993d98abfea6a0c385379b214440bbb84b9bb4-14.jpg)

```
class Solution:
    def cuttingRope(self, n: int) -> int:
        dp = [0] * (n + 1)
        dp[2] = 1
        for i in range(3, n + 1):
            for j in range(2, i):
                dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]))
        return dp[n]



```

