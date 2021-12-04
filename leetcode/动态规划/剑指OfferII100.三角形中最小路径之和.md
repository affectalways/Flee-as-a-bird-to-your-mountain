# [剑指 Offer II 100. 三角形中最小路径之和](https://leetcode-cn.com/problems/IlPe0q/)

给定一个三角形 triangle ，找出自顶向下的最小路径和。

每一步只能移动到下一行中相邻的结点上。相邻的结点 在这里指的是 下标 与 上一层结点下标 相同或者等于 上一层结点下标 + 1 的两个结点。也就是说，如果正位于当前行的下标 i ，那么下一步可以移动到下一行的下标 i 或 i + 1 。

 

**示例 1：**

```
输入：triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]
输出：11
解释：如下面简图所示：
   2
  3 4
 6 5 7
4 1 8 3
自顶向下的最小路径和为 11（即，2 + 3 + 5 + 1 = 11）。
```

**示例 2：**

```
输入：triangle = [[-10]]
输出：-10
```

**提示：**

```
1 <= triangle.length <= 200

triangle[0].length == 1
triangle[i].length == triangle[i - 1].length + 1

-104 <= triangle[i][j] <= 104
```

**进阶：**

```
你可以只使用 O(n) 的额外空间（n 为三角形的总行数）来解决这个问题吗？
```





## 思路

**动态规划**

**步骤**

```
以[[2], [3, 4], [6, 5, 7], [4, 1, 8, 3]]为例
```

|        | 第一列     | 第二列     | 第三列     | 第四列     |      |
| ------ | ---------- | ---------- | ---------- | ---------- | ---- |
| 第一行 | 2          |            |            |            |      |
| 第二行 | 3->2+3=5   | 4->2+4=6   |            |            |      |
| 第三行 | 6->6+5=11  | 5->6+5=11  | 7->4+7=11  |            |      |
| 第四行 | 4->11+4=15 | 1->11+1=12 | 8->11+8=19 | 3->11+3=14 |      |
|        |            |            |            |            |      |



```
class Solution:
    def minimumTotal(self, triangle) -> int:
        dp = [[0 for _ in range(len(triangle[-1]))] for _ in range(len(triangle))]
        dp[0][0] = triangle[0][0]
        for i in range(1, len(triangle)):
            dp[i][0] = dp[i - 1][0] + triangle[i][0]
            for j in range(1, len(triangle[i])):
                if i > 0 and j == 0:
                    dp[i][j] = dp[i - 1][j] + triangle[i][j]
                elif i > 0 and i == j:
                    dp[i][j] += dp[i - 1][j - 1] + triangle[i][j]
                elif i > 0:
                    dp[i][j] = min(dp[i - 1][j - 1], dp[i - 1][j]) + triangle[i][j]

        return min(dp[-1])
```

