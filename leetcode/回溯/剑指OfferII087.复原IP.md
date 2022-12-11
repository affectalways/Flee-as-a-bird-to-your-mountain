# [剑指 Offer II 087. 复原 IP ](https://leetcode.cn/problems/0on3uN/)

难度中等31收藏分享切换为英文接收动态反馈

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能从 `s` 获得的 **有效 IP 地址** 。你可以按任何顺序返回答案。

**有效 IP 地址** 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 **有效** IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 **无效** IP 地址。

 

**示例 1：**

```
输入：s = "25525511135"
输出：["255.255.11.135","255.255.111.35"]
```

**示例 2：**

```
输入：s = "0000"
输出：["0.0.0.0"]
```

**示例 3：**

```
输入：s = "1111"
输出：["1.1.1.1"]
```

**示例 4：**

```
输入：s = "010010"
输出：["0.10.0.10","0.100.1.0"]
```

**示例 5：**

```
输入：s = "10203040"
输出：["10.20.30.40","102.0.30.40","10.203.0.40"]
```

 

**提示：**

- `0 <= s.length <= 3000`
- `s` 仅由数字组成

 

注意：本题与主站 93 题相同：https://leetcode-cn.com/problems/restore-ip-addresses/ 



## 思路

```
回溯
```

```python
class Solution:
    def restoreIpAddresses(self, s: str) -> list[str]:
        ans = []

        if len(s) > 12:
            return ans

        def backtrack(index, path):
            if len(path) > 4:
                return
            if index == len(s) and len(path) == 4:
                ans.append('.'.join(path[:]))
                return

            for i in range(index, len(s)):
                tmp = s[index: i + 1]
                if int(tmp) < 256 and len(tmp) == len(str(int(tmp))):
                    backtrack(i + 1, path + [tmp])

        backtrack(0, [])
        
        return ans
```

