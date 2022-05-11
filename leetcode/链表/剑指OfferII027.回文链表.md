# [剑指 Offer II 027. 回文链表](https://leetcode.cn/problems/aMhZSa/)

给定一个链表的 头节点 head ，请判断其是否为回文链表。

如果一个链表是回文，那么链表节点序列从前往后看和从后往前看是相同的。

 

**示例 1：**

![](https://pic.leetcode-cn.com/1626421737-LjXceN-image.png)

```
输入: head = [1,2,3,3,2,1]
输出: true
```

**示例 2：**

![](https://pic.leetcode-cn.com/1626422231-wgvnWh-image.png)

```
输入: head = [1,2]
输出: false
```

**提示：**

- 链表 L 的长度范围为 [1, 105]

- 0 <= node.val <= 9

```
进阶：能否用 O(n) 时间复杂度和 O(1) 空间复杂度解决此题？

 
```



## 解题思路

**列表保存值，然后转字符串**

```python
class Solution:
    def isPalindrome(self, head: ListNode) -> bool:
        cur = head
        stack = list()
        while cur:
            stack.append(str(cur.val))
            cur = cur.next

        t = ''.join(stack)
        if t == t[::-1]:
            return True

        return False
```

