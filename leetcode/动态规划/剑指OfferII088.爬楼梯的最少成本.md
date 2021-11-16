# [剑指 Offer II 088. 爬楼梯的最少成本](https://leetcode-cn.com/problems/GzCJIP/)

数组的每个下标作为一个阶梯，第 i 个阶梯对应着一个非负数的体力花费值 cost[i]（下标从 0 开始）。

每当爬上一个阶梯都要花费对应的体力值，一旦支付了相应的体力值，就可以选择向上爬一个阶梯或者爬两个阶梯。

请找出达到楼层顶部的最低花费。在开始时，你可以选择从下标为 0 或 1 的元素作为初始阶梯。

 

**示例 1：**

```
输入：cost = [10, 15, 20]
输出：15
解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。
```

 **示例 2：**

```
输入：cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
输出：6
解释：最低花费方式是从 cost[0] 开始，逐个经过那些 1 ，跳过 cost[3] ，一共花费 6 。
```

**提示：**

```
2 <= cost.length <= 1000
0 <= cost[i] <= 999
```



## 思路

**动态规划**

```
设楼顶为n，爬到n的最少花费一定是 爬到n-1的最少花费和爬到n-2的最少花费中更少的哪个。

dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])
dp[0] = 0
dp[1] = 0
```

```
class Solution:
    def minCostClimbingStairs(self, cost) -> int:
        # 设楼顶为n，爬到n的最少花费一定是 爬到n-1的最少花费和爬到n-2的最少花费中更少的哪个。
        # dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])
        n = len(cost)
        dp = [0] * (n + 1)
        for i in range(2, n + 1):
            dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])

        return dp[n]


s = Solution()
cost = [10, 15, 20]
res = s.minCostClimbingStairs(cost)
print(res)

```

