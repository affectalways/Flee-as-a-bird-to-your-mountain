# [剑指 Offer II 016. 不含重复字符的最长子字符串](https://leetcode.cn/problems/wtcaE1/)

难度中等42

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长连续子字符串** 的长度。

 

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子字符串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子字符串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

**示例 4:**

```
输入: s = ""
输出: 0
```

 

**提示：**

- `0 <= s.length <= 5 * 104`
- `s` 由英文字母、数字、符号和空格组成

 

注意：本题与主站 3 题相同： https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/





## 解题思路

```
滑动窗口
```

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 滑动窗口
        l, r = 0, 1
        ml = 0

        # <= 的原因是python切片, 最后一位够不到
        while l < r <= len(s):
            partial = s[l:r]
            if len(set(partial)) != len(partial):
                l += 1
                continue

            r += 1
            ml = max(ml, len(partial))

        return ml

```

