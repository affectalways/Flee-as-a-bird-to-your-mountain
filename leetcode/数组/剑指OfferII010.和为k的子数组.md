# [剑指 Offer II 010. 和为 k 的子数组](https://leetcode.cn/problems/QTMn0o/)

难度中等86

给定一个整数数组和一个整数 `k` **，**请找到该数组中和为 `k` 的连续子数组的个数。

 

**示例 1：**

```
输入:nums = [1,1,1], k = 2
输出: 2
解释: 此题 [1,1] 与 [1,1] 为两种不同的情况
```

**示例 2：**

```
输入:nums = [1,2,3], k = 3
输出: 2
```

 

**提示:**

- `1 <= nums.length <= 2 * 104`
- `-1000 <= nums[i] <= 1000`
- `-107 <= k <= 107`

 

注意：本题与主站 560 题相同： https://leetcode-cn.com/problems/subarray-sum-equals-k/



## 解题思路

```
前缀和
```

```python
class Solution:
    def subarraySum(self, nums, k: int) -> int:
        length = len(nums)
        # 前缀和, 设置缓存数组长度为length + 1, 需要保存pres[0] = 0, 否则nums[0:1]获取和会出现问题
        pres = [0] * (length + 1)
        for i in range(1, length + 1):
            pres[i] = pres[i - 1] + nums[i - 1]

        res = 0
        for i in range(1, length + 1):
            for j in range(0, i):
                if (pres[i] - pres[j]) == k:
                    res += 1

        return res
```

