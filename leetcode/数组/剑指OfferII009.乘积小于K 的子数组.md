# [剑指 Offer II 009. 乘积小于 K 的子数组](https://leetcode.cn/problems/ZVAVXX/)

难度中等86

给定一个正整数数组 `nums`和整数 `k` ，请找出该数组内乘积小于 `k` 的连续的子数组的个数。

 

**示例 1:**

```
输入: nums = [10,5,2,6], k = 100
输出: 8
解释: 8 个乘积小于 100 的子数组分别为: [10], [5], [2], [6], [10,5], [5,2], [2,6], [5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于100的子数组。
```

**示例 2:**

```
输入: nums = [1,2,3], k = 0
输出: 0
```

 

**提示:** 

- `1 <= nums.length <= 3 * 104`
- `1 <= nums[i] <= 1000`
- `0 <= k <= 106`

 

注意：本题与主站 713 题相同：https://leetcode-cn.com/problems/subarray-product-less-than-k/ 



## 解题思路

```
双指针
```

这道题乍一看满足滑窗的条件，让我们找小于K的连续子数组的个数，但这不是求最大最小滑窗的长度，而是要求子数组的个数。有点不满足公式啊？
别着急否定，让我们来画个图考虑下滑窗右边界移动这个操作与滑窗内子数组个数的关系吧！

![](https://pic.leetcode-cn.com/1629824549-FNGuMn-image.png)

![](https://pic.leetcode-cn.com/1629824563-BcNUFo-image.png)

![](https://pic.leetcode-cn.com/1629824572-OErTKg-image.png)

![](https://pic.leetcode-cn.com/1629824581-BdOYbf-image.png)

![](https://pic.leetcode-cn.com/1629824589-bgpxid-image.png)

![](https://pic.leetcode-cn.com/1629824599-ZkpLip-image.png)

通过画图我们发现:

窗口每次移动后，ret都可以增加 right - left + 1个子数组。

这不就可以通过滑窗来解题了么？

但这里要注意一点：
如果数组中某个数字比K还大，则left会超过right，以保证有值，此时窗口长度为-1，无需计算。



```python
class Solution:
    def numSubarrayProductLessThanK(self, nums, k):
        left = ret = 0
        total = 1
        for right, num in enumerate(nums):
            total *= num
            while left <= right and total >= k:
                total //= nums[left]
                left += 1
            if left <= right:
                ret += right - left + 1
        return ret

```

