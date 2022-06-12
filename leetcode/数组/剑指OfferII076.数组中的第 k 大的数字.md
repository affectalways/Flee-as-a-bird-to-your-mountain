# [剑指 Offer II 076. 数组中的第 k 大的数字](https://leetcode.cn/problems/xx4gT2/)

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

**示例 1:**

```
输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
```

**示例 2:**

```
输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4
```

**提示：**

```
1 <= k <= nums.length <= 10^4
-10^4 <= nums[i] <= 10^4
```

**注意**：本题与主站 215 题相同： https://leetcode-cn.com/problems/kth-largest-element-in-an-array/



## 思路

```
逆序排序+下标选择
```



**代码**

```
class Solution:
    def findKthLargest(self, nums, k: int) -> int:
        # counter = Counter(nums)
        counter = sorted(nums, reverse=True)
        return counter[k - 1]

```

