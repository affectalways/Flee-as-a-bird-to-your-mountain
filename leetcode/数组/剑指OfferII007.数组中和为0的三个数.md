# [剑指 Offer II 007. 数组中和为 0 的三个数](https://leetcode.cn/problems/1fGaJU/)

难度中等65收藏分享切换为英文接收动态反馈

给定一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 `a` ，`b` ，`c` *，*使得 `a + b + c = 0` ？请找出所有和为 `0` 且 **不重复** 的三元组。

 

**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
```

**示例 2：**

```
输入：nums = []
输出：[]
```

**示例 3：**

```
输入：nums = [0]
输出：[]
```

 

**提示：**

- `0 <= nums.length <= 3000`
- `-105 <= nums[i] <= 105`

 

注意：本题与主站 15 题相同：https://leetcode-cn.com/problems/3sum/





## 思路

1. 从小到大排序，得到排序后的nums
2. 创建一个空列表res，保存最终结果
3. 依次遍历nums[:-2], 下标`i`,并用left、right指针分别指向`i+1`, `len(nums)-1`
   - nums[i] + nums[left] == -nums[right]并且不在res内，添加到res，并left +=1, right -=1
   - nums[i] + nums[left] < -nums[right],则left += 1
   - nums[i] + nums[left] > -nums[right]，则right -= 1
   - **注意：**因为需要遍历nums[i+1:] 剩余元素，所以需要用一个循环，并加限制条件left < right
4. 最终返回结果

> 时间复杂度: O(n^2)

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()

        result = list()
        for i in range(len(nums) - 2):
            if nums[i] > 0:
                # 当前数字>0, 直接break
                break

            # 双指针遍历
            l = i + 1
            r = len(nums) - 1
            while l < r:
                if nums[l] + nums[r] + nums[i] > 0:
                    # 右指针左移
                    r -= 1
                elif nums[l] + nums[r] + nums[i] < 0:
                    # 左指针右移
                    l += 1
                else:
                    if [nums[i], nums[l], nums[r]] not in result:
                        # 不存在就添加
                        result.append([nums[i], nums[l], nums[r]])
                        
                    # 左指针右移，右指针左移
                    l += 1
                    r -= 1

        return result
```

