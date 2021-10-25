# [剑指 Offer II 021. 删除链表的倒数第 n 个结点](https://leetcode-cn.com/problems/SLwz0R/)

给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

 

**示例 1：**

![](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```

**提示：**

```
链表中结点的数目为 sz
1 <= sz <= 30
0 <= Node.val <= 100
1 <= n <= sz
```



## 解题思路

**快慢指针**

> 需要一个哑节点

```
class Solution:
    def removeNthFromEnd(self, head: ListNode, n: int) -> ListNode:
        root_dummy = ListNode(0)
        root_dummy.next = head

        fast = slow = root_dummy
        for i in range(n):
            fast = fast.next

        while fast.next:
            fast = fast.next
            slow = slow.next

        slow.next = slow.next.next
        return root_dummy.next
```

