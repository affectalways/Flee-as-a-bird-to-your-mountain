# [剑指 Offer II 098. 路径的数目](https://leetcode-cn.com/problems/2AoeFn/)

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

**示例 1：**

![](https://assets.leetcode.com/uploads/2018/10/22/robot_maze.png)

```
输入：m = 3, n = 7
输出：28
```

**示例 2：**

```
输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。

1. 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右
3. 向下 -> 向右 -> 向下
```

**示例 3：**

```
输入：m = 7, n = 3
输出：28
```

**示例 4：**

```
输入：m = 3, n = 3
输出：6
```

**提示：**

```
1 <= m, n <= 100
题目数据保证答案小于等于 2 * 109
```



## 思路

**动态规划**

```
dp[i][j]表示到达坐标(i, j)的路径数
很明显的结论：

走最上面和最左边的路径只有一种，因为不能往左或往上
因为只能往下或往右, 所以到达(i ,j)格的路径总数是(i-1, j)和(i, j-1)之和
动态转移方程: dp[i][j] = dp[i-1][j] + dp[i][j-1]
```



```
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        dp = [[0 for _ in range(n)] for _ in range(m)]
        for i in range(n):
            dp[0][i] = 1
        for i in range(m):
            dp[i][0] = 1

        for i in range(1, m):
            for j in range(1, n):
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]

        return dp[m - 1][n - 1]

```

