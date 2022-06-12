# [剑指 Offer II 081. 允许重复选择元素的组合](https://leetcode.cn/problems/Ygoe9J/)

给定一个无重复元素的正整数数组 candidates 和一个正整数 target ，找出 candidates 中所有可以使数字和为目标数 target 的唯一组合。

candidates 中的数字可以无限制重复被选取。如果至少一个所选数字数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 target 的唯一组合数少于 150 个。

 

**示例 1：**

```
输入: candidates = [2,3,6,7], target = 7
输出: [[7],[2,2,3]]
```

**示例 2：**

```
输入: candidates = [2,3,5], target = 8
输出: [[2,2,2,2],[2,3,3],[3,5]]
```

**示例 3：**

```
输入: candidates = [2], target = 1
输出: []
```

**示例 4：**

```
输入: candidates = [1], target = 1
输出: [[1]]
```

**示例 5：**

```
输入: candidates = [1], target = 2
输出: [[1,1]]
```

**提示：**

- 1 <= candidates.length <= 30
- 1 <= candidates[i] <= 200
- candidate 中的每个元素都是独一无二的。
- 1 <= target <= 500

**注意**：本题与主站 39 题相同： https://leetcode-cn.com/problems/combination-sum/



## 解题思路

#### 回溯

对于这类寻找所有可行解的题，我们都可以尝试用「搜索回溯」的方法来解决。

```
以下这张图就可以解释一切！
```

![](https://assets.leetcode-cn.com/solution-static/39/39_fig1.png)

```python
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        paths = []
        if not candidates:
            return paths

        def dfs(rest, path):
            if sum(path) == target:
                # 注意：不能使用path.sort()，因为paths.append(path), 该方法使用的是递归,
                # path最终会被修改为空列表, paths.append的是path的引用, 如果直接path.sort(),
                # 然后paths.append(path), paths append的是空列表
                path = sorted(path)
                if path not in paths:
                    paths.append(path)
                return
            elif sum(path) > target:
                # 若大于，直接返回
                return

            for digit in candidates:
                path.append(digit)
                dfs(rest - digit, path)
                path.pop(-1)

        dfs(target, [])

        return paths
```



