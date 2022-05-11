# [剑指 Offer II 020. 回文子字符串的个数](https://leetcode.cn/problems/a7VOhD/)

给定一个字符串 s ，请计算这个字符串中有多少个回文子字符串。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

 

**示例 1：**

```
输入：s = "abc"
输出：3
解释：三个回文子串: "a", "b", "c"
```

**示例 2：**

```
输入：s = "aaa"
输出：6
解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
```

**提示：**

- 1 <= s.length <= 1000
- s 由小写英文字母组成

```
注意：本题与主站 647 题相同：https://leetcode-cn.com/problems/palindromic-substrings/ 
```



## 解题思路

**马拉车算法** + **遍历字符串，对每个字符，都看作回文的中心，向两端延申进行判断直到非回文**

```python
class Solution:
    def countSubstrings(self, s: str) -> int:
        if len(s) == 1:
            return len(s)
		
        # 马拉车（简洁版）
        march = "*".join(list(s))
        total = 0
        length = len(march)
        paths = []
        # 中心扩散
        for i in range(length):
            left = i - 1
            right = i + 1
            # 如果等于*，直接跳过
            if march[i] != "*":
                paths.append(march[i])
                total += 1

            while left >= 0 and right < length:
                if march[left] == march[right]:
                    if march[left] != "*":
                        total += 1
                        paths.append(march[left: right + 1])
                else:
                    break

                left -= 1
                right += 1

        return total
```

