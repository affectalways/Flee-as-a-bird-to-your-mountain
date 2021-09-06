# 

```
title: "0207 链表相交"
date: 2020-07-15T21:20:33+08:00
tags: ["leetcode"]
keywords: 
- leetcode
- blog
- 博客
- 领扣
- 0207 链表相交
- 算法
- 链表
description: "leetcode，0207 链表相交"
categories: ["leetcode", "链表"]
draft: false
```



#### [面试题 02.07. 链表相交](https://leetcode-cn.com/problems/intersection-of-two-linked-lists-lcci/)



**我竟然没读懂题！！！！！！！！！！！**



给定两个（单向）链表，判定它们是否相交并返回交点。请注意相交的定义基于节点的引用，而不是基于节点的值。换句话说，如果一个链表的第k个节点与另一个链表的第j个节点是同一节点（引用完全相同），则这两个链表相交。

**示例 1：**

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Reference of the node with value = 8
输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
```

**示例 2：**

```
输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Reference of the node with value = 2
输入解释：相交节点的值为 2 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
```

**示例 3：**

```
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
解释：这两个链表不相交，因此返回 null。
```

**注意：**

```
 1.如果两个链表没有交点，返回 null 。
 2.在返回结果后，两个链表仍须保持原有的结构。
 3.可假定整个链表结构中没有循环。
 4.程序尽量满足 O(n) 时间复杂度，且仅用 O(1) 内存。
```



**思路**

```
双指针什么的，技巧性有点强。最朴素的做法是求长度

1.第一次遍历两个链表，记录长度
2.根据两个链表的长度，得出长度差n,让长的链表的指针先走n步
3.然后两个指针一起移动，判断两者是否相等（指向同一个内存地址）
```

**代码**

```
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:

    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        len_a = 0
        len_b = 0
        cur_a = headA
        cur_b = headB
        
        # 1.第一次遍历两个链表，记录长度
        while cur_a:
            len_a += 1
            cur_a = cur_a.next

        while cur_b:
            len_b += 1
            cur_b = cur_b.next
            
		# 2.根据两个链表的长度，得出长度差n,让长的链表的指针先走n步
        cur_a = headA
        cur_b = headB
        while len_b > len_a:
            cur_b = cur_b.next
            len_b -= 1
        while len_a > len_b:
            cur_a = cur_a.next
            len_a -= 1
		# 3.然后两个指针一起移动，判断两者是否相等（指向同一个内存地址）
        while cur_a:
            if cur_a == cur_b:
                return cur_a

            cur_a = cur_a.next
            cur_b = cur_b.next

        return None

def create_list(sequence):
    headA = None
    cur = None
    for i in sequence:
        node = ListNode(i)
        if headA is None:
            headA = cur = node
            continue
        cur.next = node
        cur = cur.next
    return headA


if __name__ == '__main__':
    intersectVal = 8
    listA = [4, 1, 8, 4, 5]
    headA = create_list(listA)
    listB = [5, 0, 1, 8, 4, 5]
    headB = create_list(listB)
    skipA = 2
    skipB = 3

```


