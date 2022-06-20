# [剑指 Offer II 035. 最小时间差](https://leetcode.cn/problems/569nqc/)

给定一个 24 小时制（小时:分钟 **"HH:MM"**）的时间列表，找出列表中任意两个时间的最小时间差并以分钟数表示。

 

**示例 1：**

```
输入：timePoints = ["23:59","00:00"]
输出：1
```

**示例 2：**

```
输入：timePoints = ["00:00","23:59","00:00"]
输出：0
```

 

**提示：**

- `2 <= timePoints <= 2 * 104`
- `timePoints[i]` 格式为 **"HH:MM"**

 

注意：本题与主站 539 题相同： https://leetcode-cn.com/problems/minimum-time-difference/



## 思路

首先，遍历时间列表，将其转换为“分钟制”列表 mins，比如，对于时间点 13:14，将其转换为 13 * 60 + 14。

接着将“分钟制”列表按升序排列，然后将此列表的最小时间 mins[0] 加上 24 * 60 追加至列表尾部，用于处理最大值、最小值的差值这种特殊情况。

最后遍历“分钟制”列表，找出相邻两个时间的最小值即可。

```
def transfer_minutes(t):
    return int(t[:2]) * 60 + int(t[3:])


class Solution:
    def findMinDifference(self, timePoints):
   		# 得到分钟制
        minutes = [transfer_minutes(item) for item in timePoints]
        # 排序
        minutes.sort()
        # 将此列表的最小时间 mins[0] 加上 24 * 60 追加至列表尾部
        minutes.append(minutes[0] + 1440)
        res = float('inf')
        # 遍历
        for i in range(1, len(minutes)):
            res = min(res, minutes[i] - minutes[i - 1])

        return res
```

