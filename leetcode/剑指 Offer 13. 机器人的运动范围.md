#### [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

 

**示例 1：**

```
输入：m = 2, n = 3, k = 1
输出：3
```

**示例 2：**

```
输入：m = 3, n = 1, k = 0
输出：1
```

**提示：**

```
1 <= n,m <= 100
0 <= k <= 20
```



### 思路

#### 广度优先遍历 BFS

- **BFS 实现：** 通常利用队列实现广度优先遍历。

##### 算法解析：

- **初始化：** 将机器人初始点 (0, 0) 加入队列 queue ；
- **迭代终止条件：** queue 为空。代表已遍历完所有可达解。
- **迭代工作：**
  - 单元格出队： 将队首单元格的 索引、数位和 弹出，作为当前搜索单元格。
  - 判断是否跳过： 若 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过 时，执行 continue 。
  - 标记当前单元格 ：将单元格索引 (i, j) 存入 Set visited 中，代表此单元格 已被访问过 。
  - 单元格入队： 将当前元素的 下方、右方 单元格的 索引、数位和 加入 queue 。
- **返回值**： Set visited 的长度 len(visited) ，即可达解的数量。



```
class Solution:
    def sums(self, x):
        # 计算单个int所有位数和
        s = 0
        while x != 0:
            s += x % 10
            x = x // 10
        return s

    def value_count(self, x, y):
        return self.sums(x) + self.sums(y)

    def movingCount(self, m: int, n: int, k: int) -> int:
        # 广度优先遍历bfs
        # 使用队列实现
        # 1.初始化变量，队列和visited（访问过的变量）
        stack, visited = [[0, 0]], set()
        while stack:
            # 2.bfs
            x, y = stack.pop(0)
            # 3.坐标已经访问、超出边界、大于k的情况，不添加visited
            if (x, y) in visited or x >= m or y >= n or self.value_count(x, y) > k:
                continue
            # 4.符合条件的放入visited
            visited.add((x, y))
            # 5.横坐标+1，纵坐标+1，不需要考虑-1的情况（因为肯定已经访问了）
            stack.append([x + 1, y])
            stack.append([x, y + 1])

        # 6.放回所有访问过的长度
        return len(visited)


s = Solution()
result = s.movingCount(1, 2, 1)
print(result)

```

