# 0201 移除重复节点


#### [面试题 02.01. 移除重复节点](https://leetcode-cn.com/problems/remove-duplicate-node-lcci/)

编写代码，移除未排序链表中的重复节点。保留最开始出现的节点。

**示例1:**

```
 输入：[1, 2, 3, 3, 2, 1]
 输出：[1, 2, 3]
```

**示例2:**

```
 输入：[1, 1, 1, 1, 2]
 输出：[1, 2]
```

**提示：**

1. 链表长度在[0, 20000]范围内。
2. 链表元素在[0, 20000]范围内。

**进阶：**

如果不得使用临时缓冲区，该怎么解决？



**思路**

#### 哈希 O(n)

哈希表存储出现过的元素，如果当前节点出现过，就删掉

我们从链表的头节点`head` 开始进行遍历，遍历的指针记为`cur`。由于头节点一定不会被删除，因此我们可以枚举待移除节点的前驱节点`pre`，减少编写代码的复杂度。



```
# -*- coding: utf-8 -*-
# @Time     : 2020/7/16 22:27
# @Author   : affectalways
# @Site     : 
# @Contact  : affectalways@gmail.com
# @File     : 0201.py
# @Software : PyCharm 

# Definition for singly-linked list.
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution(object):
    def removeDuplicateNodes(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return head
        tmp = {head.val}
        pre = head
        cur = pre.next
        while cur:
            if cur.val in tmp:
                pre.next = cur.next
                cur = cur.next
            else:
                tmp.add(cur.val)
                pre = pre.next
                cur = cur.next
        return head


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
    head = create_link([1, 2, 3, 3, 2, 1])
    # traversal_link(head)
    solution = Solution()
    result = solution.removeDuplicateNodes(head)
    traversal_link(result)

```

#### 双指针

固定p指针，右侧q指针扫描，然后移动p，指针q再次扫描
时间复杂度 O(n^2)

```
# Definition for singly-linked list.
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution(object):
    def removeDuplicateNodes(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        p = head
        while p:
            q = p
            while q.next:
                if q.next.val == p.val:
                    q.next = q.next.next
                else:
                    q = q.next
            p = p.next

        return head


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
    head = create_link([1, 2, 3, 3, 2, 1])
    # traversal_link(head)
    solution = Solution()
    result = solution.removeDuplicateNodes(head)
    traversal_link(result)

```


