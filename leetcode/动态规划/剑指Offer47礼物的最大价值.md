# [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

**示例 1:**

```
输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```

**提示：**

```
0 < grid.length <= 200
0 < grid[0].length <= 200
```





## 解题思路

#### 动态规划

1. 用二维数组`dp[i, j]`表示第i行, 第j列最多价值的礼物
2. 因为只能向右或向下，所以`i, j`的结果可以由`i-1,j``+grid[i][j]`，`i1,j-1``+grid[i][j]`中最大的得出

**状态转移方程**：

```python
if i == 0 and j == 0:
	dp[i][j] = grid[i][j]
elif i == 0 and j != 0:
    dp[i][j] = grid[i][j] + dp[i][j - 1]
elif i != 0 and j == 0:
	dp[i][j] = grid[i][j] + dp[i - 1][j]
else:
    dp[i][j] = grid[i][j] + max(dp[i - 1][j], dp[i][j - 1])
```

**边界：**

```
0 <= i <= m
0 <= j <= n
```



```python
class Solution:
    def maxValue(self, grid) -> int:
        if not grid:
            return 0

        n = len(grid)
        m = len(grid[0])
        dp = [[0 for _ in range(m)] for _ in range(n)]
        for i in range(n):
            for j in range(m):
                if i == 0 and j == 0:
                    dp[i][j] = grid[i][j]
                elif i == 0 and j != 0:
                    dp[i][j] = grid[i][j] + dp[i][j - 1]
                elif i != 0 and j == 0:
                    dp[i][j] = grid[i][j] + dp[i - 1][j]
                else:
                    dp[i][j] = grid[i][j] + max(dp[i - 1][j], dp[i][j - 1])

        return dp[-1][-1]
```

