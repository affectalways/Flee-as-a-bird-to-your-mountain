# [剑指 Offer II 084. 含有重复元素集合的全排列 ](https://leetcode.cn/problems/7p8L0Z/)

给定一个可包含重复数字的整数集合 nums ，按任意顺序 返回它所有不重复的全排列。

 

**示例 1：**

```
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
```

**示例 2：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**提示：**

- 1 <= nums.length <= 8
- -10 <= nums[i] <= 10

**注意：**本题与主站 47 题相同： https://leetcode-cn.com/problems/permutations-ii/



## 解题思路

#### python库函数：itertools.permutations

```
from itertools import permutations


class Solution:
    def permuteUnique(self, nums):
        items = permutations(nums)
        ans = []
        for item in items:
            if list(item) in ans:
                continue

            ans.append(list(item))

        return ans
```



