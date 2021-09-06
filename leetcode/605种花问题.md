# 605种花问题


#### [605. 种花问题](https://leetcode-cn.com/problems/can-place-flowers/)

假设你有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花卉不能种植在相邻的地块上，它们会争夺水源，两者都会死去。

给定一个花坛（表示为一个数组包含0和1，其中0表示没种植花，1表示种植了花），和一个数 n 。能否在不打破种植规则的情况下种入 n 朵花？能则返回True，不能则返回False。

**示例 1:**

```
输入: flowerbed = [1,0,0,0,1], n = 1
输出: True
```


**示例 2:**

```
输入: flowerbed = [1,0,0,0,1], n = 2
输出: False
```


**注意:**

```
数组内已种好的花不会违反种植规则。
输入的数组长度范围为 [1, 20000]。
n 是非负整数，且不会超过输入数组的大小。
```



**思路**

```
遍历数组：
（1）对第一个元素，本身是0右边是0即可种花
（2）对于中间元素，本身是0，左右两边都是0，可以种花
（3）对于最后一个元素，本身是0，左边是0，可以种花
```



**代码**

```
package main

import "fmt"

func canPlaceFlowers(flowerbed []int, n int) bool {
	length := len(flowerbed)
	var count int
	for i, value := range flowerbed {
		if value == 0 && (i == 0 || flowerbed[i-1] == 0) && (i == length-1 || flowerbed[i+1] == 0) {
			flowerbed[i] = 1
			count += 1
		}
	}
	return count >= n
}

func main() {
	flowerbed := []int{1, 0, 0, 0, 1}
	n := 1
	result := canPlaceFlowers(flowerbed, n)
	fmt.Print(result)
}

```


