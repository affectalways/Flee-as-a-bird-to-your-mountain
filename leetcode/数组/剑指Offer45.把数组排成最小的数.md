# [剑指 Offer 45. 把数组排成最小的数](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

 

**示例 1:**

```
输入: [10,2]
输出: "102"
```

**示例 2:**

```
输入: [3,30,34,5,9]
输出: "3033459"
```

**提示:**

- 0 < nums.length <= 100

**说明:**

- 输出结果可能非常大，所以你需要返回一个字符串而不是整数

- 拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0



## 思路

**同179.最大数**

贪心解法
对于 nums 中的任意两个值 a 和 b，我们无法直接从常规角度上确定其大小/先后关系。

但我们可以根据「结果」来决定 a 和 b 的排序关系：

如果拼接结果 ab 要比 ba 好，那么我们会认为 a 应该放在 b 前面。

另外，注意我们需要处理前导零（最多保留一位）。



```python
import functools


class Solution:
    def largestNumber(self, nums) -> str:
        def cmp(a, b):
            if a + b == b + a:
                return 0
            elif a + b > b + a:
                return 1
            else:
                return -1

        strs = list(map(str, nums))
        strs.sort(key=functools.cmp_to_key(cmp), reverse=True)
        return ''.join(strs) if strs[0] != '0' else '0'
```

