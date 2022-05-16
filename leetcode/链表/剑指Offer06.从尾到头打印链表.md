# [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。



**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

 

**限制：**

```
0 <= 链表长度 <= 10000
```



## **思路**

- 创建一个列表
- 遍历链表，把链表中的val放入列表中
- 列表逆序返回

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def reversePrint(self, head: ListNode) -> list[int]:
        stack = []
        while head:
            stack.append(head.val)
            head = head.next

        return stack[::-1]

```

