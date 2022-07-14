# [剑指 Offer II 104. 排列的数目](https://leetcode.cn/problems/D0F0SV/)

给定一个由 **不同** 正整数组成的数组 `nums` ，和一个目标整数 `target` 。请从 `nums` 中找出并返回总和为 `target` 的元素组合的个数。数组中的数字可以在一次排列中出现任意次，但是顺序不同的序列被视作不同的组合。

题目数据保证答案符合 32 位整数范围。

 

**示例 1：**

```
输入：nums = [1,2,3], target = 4
输出：7
解释：
所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)
请注意，顺序不同的序列被视作不同的组合。
```

**示例 2：**

```
输入：nums = [9], target = 3
输出：0
```

 

**提示：**

- `1 <= nums.length <= 200`
- `1 <= nums[i] <= 1000`
- `nums` 中的所有元素 **互不相同**
- `1 <= target <= 1000`

 

**进阶：**如果给定的数组中含有负数会发生什么？问题会产生何种变化？如果允许负数出现，需要向题目中添加哪些限制条件？

 

注意：本题与主站 377 题相同：https://leetcode-cn.com/problems/combination-sum-iv/



## 解题思路

```
动态规划
```

- 本次的DP思路与上一题 [剑指 Offer II 103. 最少的硬币数目](剑指OfferII103.最少的硬币数目.md) 一致，只不过多了一个排列组合的条件；

- 只要将状态转移方程修改为dp[i] += dp[i - num]即可，累加不同排列组合的状态。


```python
class Solution:
    def combinationSum4(self, nums: List[int], target: int) -> int:
        dp = [1] + [0] * target
        for i in range(1, target + 1):
            for num in nums:
                if num <= i:
                    dp[i] += dp[i - num]
        
        return dp[target]
```

