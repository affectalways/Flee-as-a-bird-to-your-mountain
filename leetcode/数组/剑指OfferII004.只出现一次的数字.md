# [剑指 Offer II 004. 只出现一次的数字 ](https://leetcode.cn/problems/WGki4K/)

难度中等78收藏分享切换为英文接收动态反馈

给你一个整数数组 `nums` ，除某个元素仅出现 **一次** 外，其余每个元素都恰出现 **三次 。**请你找出并返回那个只出现了一次的元素。

 

**示例 1：**

```
输入：nums = [2,2,3,2]
输出：3
```

**示例 2：**

```
输入：nums = [0,1,0,1,0,1,100]
输出：100
```

 

**提示：**

- `1 <= nums.length <= 3 * 104`
- `-231 <= nums[i] <= 231 - 1`
- `nums` 中，除某个元素仅出现 **一次** 外，其余每个元素都恰出现 **三次**

 

**进阶：**你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

 

注意：本题与主站 137 题相同：https://leetcode-cn.com/problems/single-number-ii/





## 思路

**python库collections.Counter**

```python
from collections import Counter


class Solution:
    def singleNumber(self, nums: list[int]) -> int:
        c = Counter(nums)
        for key, value in c.items():
            if value == 1:
                return key

```

