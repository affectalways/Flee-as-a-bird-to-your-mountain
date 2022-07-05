# [剑指 Offer II 073. 狒狒吃香蕉](https://leetcode.cn/problems/nZZqjQ/)

难度中等31收藏分享切换为英文接收动态反馈

狒狒喜欢吃香蕉。这里有 `n` 堆香蕉，第 `i` 堆中有 `piles[i]` 根香蕉。警卫已经离开了，将在 `h` 小时后回来。

狒狒可以决定她吃香蕉的速度 `k` （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 `k` 根。如果这堆香蕉少于 `k` 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉，下一个小时才会开始吃另一堆的香蕉。 

狒狒喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。

返回她可以在 `h` 小时内吃掉所有香蕉的最小速度 `k`（`k` 为整数）。

 



**示例 1：**

```
输入：piles = [3,6,7,11], h = 8
输出：4
```

**示例 2：**

```
输入：piles = [30,11,23,4,20], h = 5
输出：30
```

**示例 3：**

```
输入：piles = [30,11,23,4,20], h = 6
输出：23
```

 

**提示：**

- `1 <= piles.length <= 104`
- `piles.length <= h <= 109`
- `1 <= piles[i] <= 109`

 

**注意：**本题与主站 875 题相同： https://leetcode-cn.com/problems/koko-eating-bananas/



## 解题思路

**二分查找**

```
# 虽然目前不清楚狒狒吃香蕉的速度，但是可以得知狒狒的速度肯定需要大于等于 1，
# 同时也要小于等于最大的一堆香蕉数量 max，
# 因为若大于 max 每小时也只能吃一堆，所以更大的速度是没有意义的。
# 这时候可以得到狒狒吃香蕉的速度应该在 [1, max]，可以使用二分查找算法确定速度。
# 取 1 和 max 的中间值 K，计算出速度为 K 时吃完香蕉所需时间 T。

# 给定一个K，可以遍历piles求需要的时间T，这个T必须小于等于H
# K越大T越小，
# 如果T > H，说明吃得太慢了，需要增加K，搜索区间就变成[K+1, right]
# 如果T = H，满足条件，但希望找到符合条件的更小的K，搜索区间就变成[left, K-1]
# 如果T < H，说明吃的太快了，可以适当减小K，搜索区间就变成[left, K-1]
```

```python
import math


class Solution:
    def minEatingSpeed(self, piles, h: int) -> int:
        left, right = 1, max(piles)
        while left <= right:
            # 取中值
            mid = (left + right) // 2
            T = 0
            
            for pile in piles:
                T += math.ceil(pile / mid)
                # 大于h，直接break
                if T > h:
                    break
            
            if T == h:
                # 如果等于, 继续左移
                right = mid - 1
            elif T > h:
                left = mid + 1
            elif T < h:
                right = mid - 1
        return left
```

