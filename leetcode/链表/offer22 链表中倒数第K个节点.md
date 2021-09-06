---
title: "Offer22 链表中倒数第K个节点"
date: 2020-07-22T23:15:30+08:00
tags: ["leetcode"]
keywords: 
- leetcode
- blog
- 博客
- 领扣
- Offer22 链表中倒数第K个节点
- 算法
- 链表
description: "leetcode，Offer22 链表中倒数第K个节点"
categories: ["leetcode", "链表"]
draft: false
---

#### [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

 

**示例：**

```
给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```



**解题思路**

```
1.快指针和慢指针相差k
2.快指针到达链尾，慢指针所指方向就是倒数第k个链表
```

**代码**

```
class Solution(object):
    def getKthFromEnd(self, head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        fast = head
        for i in range(k):
            fast = fast.next

        cur = head
        while fast:
            cur = cur.next
            fast = fast.next
            
        return cur
```



**解题思路**

```
1.获取链表长度
2.开始遍历（链表长度-k）次数
```

**代码**

```
class Solution(object):
    def getKthFromEnd(self, head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        length = 0
        cur = head
        while cur:
            length += 1
            cur = cur.next

        cur = head
        index = length - k
        for i in range(index):
            cur = cur.next

        return cur
```

