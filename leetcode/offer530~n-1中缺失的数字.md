# Offer53 0~n-1中缺失的数字


#### [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

**示例 1:**

```
输入: [0,1,3]
输出: 2
```

**示例 2:**

```
输入: [0,1,2,3,4,5,6,7,9]
输出: 8
```

**限制：**

```
1 <= 数组长度 <= 10000
```



**解题思路**
**思路一：依次遍历**

```
(1) 从头开始依次遍历数组每一位元素。
(2) 找到第一位值不等于索引的元素，则缺失值为当前索引。
```

**代码**

```
package main

import "fmt"

func missingNumber(nums []int) int {
	for index, value := range nums {
		if index == 0 {
			continue
		}
		if (value - nums[index-1]) > 1 {
			return value - 1
		}

	}
	return 0
}

func main() {
	nums := []int{0, 1, 3}
	number := missingNumber(nums)
	fmt.Println(number)
}

```



**思路二：二分查找**

```
(1) 根据二分查找找到第一个元素值不等于索引的索引就是缺失值。
(2) 元素值都和索引相等则缺失值等于数组长度。
```

```
class Solution {
    public int missingNumber(int[] nums) {
        // 定义左右指针分别指向数组元素值的边界。
        int left = 0, right = nums.length;
        while (left < right) {
            // 找到中间值。
            int mid = left + (right - left) / 2;
            if (nums[mid] > mid) {
                // 数组索引值与索引不对应，则缺失值在左侧。
                right = mid;
            } else {
                // 数组索引值等于索引，则缺失值在右侧。
                left = mid + 1;
            }
        }
        return right;
    }
}

```


