# [剑指 Offer II 014. 字符串中的变位词](https://leetcode.cn/problems/MPnaiL/)

难度中等52

给定两个字符串 `s1` 和 `s2`，写一个函数来判断 `s2` 是否包含 `s1` 的某个变位词。

换句话说，第一个字符串的排列之一是第二个字符串的 **子串** 。

 

**示例 1：**

```
输入: s1 = "ab" s2 = "eidbaooo"
输出: True
解释: s2 包含 s1 的排列之一 ("ba").
```

**示例 2：**

```
输入: s1= "ab" s2 = "eidboaoo"
输出: False
```

 

**提示：**

- `1 <= s1.length, s2.length <= 104`
- `s1` 和 `s2` 仅包含小写字母

 

注意：本题与主站 567 题相同： https://leetcode-cn.com/problems/permutation-in-string/



## 解题思路

```
左右指针
```

```
初始化左右指针，left=0, right=len(s1)
同时右移左右指针，计算s2左右指针区间内的字符是否个数是否等于s1的字符个数
若相等，直接返回True
不相等，返回False
```

```python
from collections import Counter


class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        c1 = Counter(s1)
        length = len(s1)
        l, r = 0, length
        while r <= len(s2):
            tmp = s2[l:r]
            c2 = Counter(tmp)
            flag = True
            for key, value in c2.items():
                if key not in c1:
                    flag = False
                    break

                if value != c1[key]:
                    flag = False
                    break

            if flag:
                return True

            l += 1
            r += 1

        return False
```

