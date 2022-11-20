# [剑指 Offer II 008. 和大于等于 target 的最短子数组](https://leetcode.cn/problems/2VG8Kg/)

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

 

**示例 1：**

```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**示例 2：**

```
输入：target = 4, nums = [1,4,4]
输出：1
```

**示例 3：**

```
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

**提示：**

- 1 <= target <= 109
- 1 <= nums.length <= 105
- 1 <= nums[i] <= 105

**进阶：**

- 如果你已经实现 O(n) 时间复杂度的解法, 请尝试设计一个 O(n log(n)) 时间复杂度的解法。


**注意：**本题与主站 209 题相同：https://leetcode-cn.com/problems/minimum-size-subarray-sum/



## 解题思路

```
滑动窗口
```

```
1.初始化start,end指向第一个元素
2.若片段和小于target, end右移, 并continue; end右移代表扩大滑动窗口范围，因为元素都>0，所以扩大滑动窗口范围等同于片段和变大
3.若片段和大于等于target，最小长度取最小值，start右移
4.直至end超出长度范围
```



```python
class Solution:
  def minSubArrayLen(self, target: int, nums) -> int:
    if not nums:
      return 0

    # 滑动窗口
    start = end = 0
    # 最小长度初始化为无穷大
    min_length = float('inf')
    while end < len(nums):
      # 若片段和小于target, end右移, 并continue; end右移代表扩大滑动窗口范围，
      # 因为元素都>0，所以扩大滑动窗口范围等同于片段和变大
      if sum(nums[start: end + 1]) < target:
        end += 1
        continue

      # 最小长度取最小值，start右移
      min_length = min(min_length, end - start + 1)
      start += 1

    return min_length
```

