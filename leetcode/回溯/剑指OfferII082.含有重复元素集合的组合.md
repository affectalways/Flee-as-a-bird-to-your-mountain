# [剑指 Offer II 082. 含有重复元素集合的组合](https://leetcode.cn/problems/4sjJUc/)

给定一个可能有重复数字的整数数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次，解集不能包含重复的组合。 

 

**示例 1:**

```
输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出:
[
[1,1,6],
[1,2,5],
[1,7],
[2,6]
]
```

**示例 2:**

```
输入: candidates = [2,5,2,1,2], target = 5,
输出:
[
[1,2,2],
[5]
]
```

**提示:**

- 1 <= candidates.length <= 100
- 1 <= candidates[i] <= 50
- 1 <= target <= 30

**注意**：本题与主站 40 题相同： https://leetcode-cn.com/problems/combination-sum-ii/



## 思路

**排序+python库**

```python
from itertools import chain
from itertools import combinations


class Solution:
    def combinationSum2(self, candidates, target: int):
        candidates.sort()
        t = chain.from_iterable(
            combinations(candidates, i) for i in range(len(candidates) + 1))
        ans = []
        for item in t:
            if sum(item) == target and item not in ans:
                ans.append(item)

        return ans

```

