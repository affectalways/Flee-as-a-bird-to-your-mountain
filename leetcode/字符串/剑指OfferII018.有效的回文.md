# [剑指 Offer II 018. 有效的回文](https://leetcode.cn/problems/XltzEq/)

给定一个字符串 s ，验证 s 是否是 回文串 ，只考虑字母和数字字符，可以忽略字母的大小写。

**本题中，将空字符串定义为有效的 回文串** 。

 

**示例 1:**

```
输入: s = "A man, a plan, a canal: Panama"
输出: true
解释："amanaplanacanalpanama" 是回文串
```

**示例 2:**

```
输入: s = "race a car"
输出: false
解释："raceacar" 不是回文串
```

**提示：**

- 1 <= s.length <= 2 * 105

- 字符串 s 由 ASCII 字符组成


注意：本题与主站 125 题相同： https://leetcode-cn.com/problems/valid-palindrome/



## 解题思路

```
只取字符和数字，然后反转
```

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        new_s = ''
        for i in s:
            if i.isdigit():
                new_s += i
            elif i.isalpha():
                new_s += i.lower()
        new_s = new_s.replace(' ', '')
        if new_s == new_s[::-1]:
            return True

        return False
```

