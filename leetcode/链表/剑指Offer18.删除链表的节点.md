# [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

返回删除后的链表的头节点。

**注意：**此题对比原题有改动

**示例 1:**

```
输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

**示例 2:**

```
输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

**说明：**

- 题目保证链表中节点的值互不相同
- 若使用 C 或 C++ 语言，你不需要 `free` 或 `delete` 被删除的节点



## 解题思路

```
哑节点+遍历+双指针
```

删除值为 val 的节点可分为两步：定位节点、修改引用。

定位节点： 遍历链表，直到 head.val == val 时跳出，即可定位目标节点。
修改引用： 设节点 cur 的前驱节点为 pre ，后继节点为 cur.next ；则执行 pre.next = cur.next ，即可实现删除 cur 节点。

![](https://pic.leetcode-cn.com/0091d27673ec013c5557c7f9e7c731d3437f0ce655439269a6e24ce501235e4b-Picture0.png)

**算法流程：**

```
1.特例处理： 当应删除头节点 head 时，直接返回 head.next 即可。
2.初始化： pre = head , cur = head.next 。
3.定位节点： 当 cur 为空 或 cur 节点值等于 val 时跳出。
4.保存当前节点索引，即 pre = cur 。
5.遍历下一节点，即 cur = cur.next 。
6.删除节点： 若 cur 指向某节点，则执行 pre.next = cur.next 。（若 cur 指向 nullnull ，代表链表中不包含值为 val 的节点。
7.返回值： 返回链表头部节点 head 即可。


```

```python
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution(object):
    def deleteNode(self, head, val):
        """
        :type head: ListNode
        :type val: int
        :rtype: ListNode
        """
        if head.val == val:
            return head.next
        pre, cur = head, head.next
        while cur and cur.val != val:
            pre, cur = cur, cur.next
        if cur:
            pre.next = cur.next
        return head

```

