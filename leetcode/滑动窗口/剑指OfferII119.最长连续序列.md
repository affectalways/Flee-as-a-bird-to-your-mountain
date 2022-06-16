# [剑指 Offer II 119. 最长连续序列](https://leetcode.cn/problems/WhsWhI/)

难度中等33收藏分享切换为英文接收动态反馈

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

 

**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**示例 2：**

```
输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

 

**提示：**

- `0 <= nums.length <= 104`
- `-109 <= nums[i] <= 109`

 

**进阶：**可以设计并实现时间复杂度为 `O(n)` 的解决方案吗？

 

注意：本题与主站 128 题相同： https://leetcode-cn.com/problems/longest-consecutive-sequence/



## 思路

```
去重+排序+滑动窗口
```

```
class Solution:
    def longestConsecutive(self, nums: list[int]) -> int:
        nums = list(set(nums))
        nums.sort()

        max_len = 1
        start = 0
        for end in range(1, len(nums)):
            if nums[end] - nums[end - 1] == 1:
                max_len = max(max_len, end - start + 1)
            else:
                start = end

        return max_len
```

