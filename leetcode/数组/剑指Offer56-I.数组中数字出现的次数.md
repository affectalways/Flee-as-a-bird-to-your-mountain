# [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

难度中等667

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

 

**示例 1：**

```
输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
```

**示例 2：**

```
输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]
```

 

**限制：**

- `2 <= nums.length <= 10000`

 

## 解题思路

题目要求时间复杂度 $O(N)$，空间复杂度 $O(1)$，因此首先排除 **暴力法** 和 **哈希表统计法** 。

> **简化问题：** 一个整型数组 `nums` 里除 **一个** 数字之外，其他数字都出现了两次。

设整型数组 $nums$ 中出现一次的数字为$x$ ，出现两次的数字为$ a, a, b, b, ...$即：
$$
nums=[a,a,b,b,...,x]
$$
异或运算有个重要的性质，两个相同数字异或为$0$ ，即对于任意整数 $a$ 有 $a \oplus a = 0 $。因此，若将 $nums$ 中所有数字执行异或运算，留下的结果则为 出现一次的数字 $x$ ，即：
$$
   a⊕a⊕b⊕b⊕...⊕x \\
= 0⊕0⊕...⊕x \\
= x
$$
异或运算满足交换律 $a \oplus b = b \oplus a $，即以上运算结果与 $nums$ 的元素顺序无关。代码如下：

```python
def singleNumber(self, nums: List[int]) -> List[int]:
    x = 0
    for num in nums:  # 1. 遍历 nums 执行异或运算
        x ^= num      
    return x;         # 2. 返回出现一次的数字 x
```

> **本题难点：** 数组 nums 有 **两个** 只出现一次的数字，因此无法通过异或直接得到这两个数字。



但是这是有两个不同的数字，所以用上面这种方法搞不定，还是用Python库吧

```python
from collections import Counter


class Solution:
    def singleNumbers(self, nums: list[int]) -> list[int]:
        t = Counter(nums)
        ans = []
        for key, value in t.items():
            if value == 1:
                ans.append(key)

        return ans
```

