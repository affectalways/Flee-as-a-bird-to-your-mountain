# [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

输入一个字符串，打印出该字符串中字符的所有排列。

 

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

 

**示例:**

```
输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]
```

**限制：**

```
1 <= s 的长度 <= 8
```



### 解题思路

#### 全排列用  回溯

> 参考：https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/solution/mian-shi-ti-38-zi-fu-chuan-de-pai-lie-hui-su-fa-by/

排列方案的生成：

根据字符串排列的特点，考虑深度优先搜索所有排列方案。即通过字符交换，先固定第 1位字符（ n 种情况）、再固定第 2 位字符（ n−1 种情况）、... 、最后固定第 n 位字符（ 1 种情况）。

![](https://pic.leetcode-cn.com/1599403497-KXKQcp-Picture1.png)



```
class Solution:
    def permutation(self, s: str) -> List[str]:
        c, res = list(s), []
        def dfs(x):
            if x == len(c) - 1:
                res.append(''.join(c))   # 添加排列方案
                return
            dic = set()
            for i in range(x, len(c)):
                if c[i] in dic: continue # 重复，因此剪枝
                dic.add(c[i])
                c[i], c[x] = c[x], c[i]  # 交换，将 c[i] 固定在第 x 位
                dfs(x + 1)               # 开启固定第 x + 1 位字符
                c[i], c[x] = c[x], c[i]  # 恢复交换
        dfs(0)
        return res


```



#### 或直接使用python三方库

```python
from itertools import permutations


class Solution:
    def permutation(self, s: str) -> list[str]:
        group = permutations(s)
        res = []
        for item in group:
            value = ''.join(list(item))
            # 不允许有重复的
            if value not in res:
                res.append(value)

        return res

```

