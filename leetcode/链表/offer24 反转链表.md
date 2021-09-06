---
title: "Offer24 反转链表"
date: 2020-07-16T22:24:25+08:00
tags: ["leetcode"]
keywords: 
- leetcode
- blog
- 博客
- 领扣
- Offer24 反转链表
- 算法
- 链表
description: "leetcode，Offer24 反转链表"
categories: ["leetcode", "链表"]
draft: false
---

#### [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/) 

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

 

**限制：**

```
0 <= 节点个数 <= 5000
```



**思路**

```
# Definition for singly-linked list.
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution(object):
    def reverseList(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        cur = head
        pre = None
        while cur:
            after = cur.next
            cur.next = pre
            pre = cur
            cur = after
        return pre


def create_link(tmp):
    head = None
    cur = None
    for i in tmp:
        node = ListNode(i)
        if head is None:
            head = node
            cur = head
        else:
            cur.next = node
            cur = cur.next
    return head


def traversal_link(head):
    cur = head
    while cur:
        print(cur.val)
        cur = cur.next


if __name__ == '__main__':
    head = create_link([1, 2, 3, 4, 5])
    # traversal_link(head)
    solution = Solution()
    cur = solution.reverseList(head)
    traversal_link(cur)
```

