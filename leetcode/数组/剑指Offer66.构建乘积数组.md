# [剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/)

给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B[i] 的值是数组 A 中除了下标 i 以外的元素的积, 即 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

 

**示例:**

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

**提示：**

- 所有元素乘积之和不会溢出 32 位整数

- a.length <= 100000





## 解题思路

**python库functools.reduce**

```python
from functools import reduce


class Solution:
    def constructArr(self, a):
        path = []
        for i in range(len(a)):
            if i + 1 > len(a):
                t = a[0:i]
            else:
                t = a[0:i] + a[i + 1:]
                path.append(reduce(lambda x, y: x * y, t))

        return path
```

