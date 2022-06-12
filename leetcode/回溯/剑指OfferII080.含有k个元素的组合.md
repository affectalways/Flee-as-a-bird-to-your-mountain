# [剑指 Offer II 080. 含有 k 个元素的组合](https://leetcode.cn/problems/uUsW3B/)

给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。

 

**示例 1:**

```
输入: n = 4, k = 2
输出:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

**示例 2:**

```
输入: n = 1, k = 1
输出: [[1]]
```

**提示:**

- 1 <= n <= 20
- 1 <= k <= n

**注意：**本题与主站 77 题相同： https://leetcode-cn.com/problems/combinations/



## 解题思路

```
python自带的库，itertool
```



```python
import itertools


class Solution:
    def combine(self, n: int, k: int) -> list[list[int]]:
        nums = [i for i in range(1, n + 1)]
        return list(itertools.combinations(nums, k))

```

