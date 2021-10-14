#### [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

```
输入：n = 2
输出：2
```

**示例 2：**

```
输入：n = 7
输出：21
```

**示例 3：**

```
输入：n = 0
输出：1
```

**提示：**

```
0 <= n <= 100
```



### 动态规划

```
class Solution:
    def numWays(self, n: int) -> int:

        if n == 1 or n == 0:
            return 1
        elif n == 2:
            return 2
        first = 1
        second = 2
        result = float("-inf")
        for i in range(3, n + 1):
            result = (first + second) % 1000000007
            first = second
            second = result

        return result
```

