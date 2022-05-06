# [剑指 Offer 63. 股票的最大利润](https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/)

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

 

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

**限制：**

```
0 <= 数组长度 <= 10^5
```



## 解题思路

#### 动态规划

用一维数组`dp`代表前`i`日的最大利润 

**状态转移方程：**

由于题目限定 “买卖该股票一次” ，因此

```
前 i 日最大利润 = max(前 (i-1) 日最大利润, 第 i 日价格 - 前 i 日最低价格)
```

**状态转移方程：**

```
dp[i] = max(dp[i−1], prices[i]−min(prices[0:i]))
```

```

作者：jyd
链接：https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/solution/mian-shi-ti-63-gu-piao-de-zui-da-li-run-dong-tai-2/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

作者：jyd
链接：https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/solution/mian-shi-ti-63-gu-piao-de-zui-da-li-run-dong-tai-2/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```
class Solution:
    def maxProfit(self, prices) -> int:
        # dp[0] = 0
        # dp[i] = max(dp[i-1], prices[i] - min(prices[0:i-1]))
        result = [0] * len(prices)
        for i in range(1, len(prices)):
            result[i] = max(result[i - 1], prices[i] - min(prices[0:i]))

        return max(result) if result else 0
```

