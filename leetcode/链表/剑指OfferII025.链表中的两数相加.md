# [剑指 Offer II 025. 链表中的两数相加](https://leetcode.cn/problems/lMSNwu/)

给定两个 **非空链表** `l1`和 `l2` 来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。

可以假设除了数字 0 之外，这两个数字都不会以零开头。

 

**示例1：**

![img](https://pic.leetcode-cn.com/1626420025-fZfzMX-image.png)

```
输入：l1 = [7,2,4,3], l2 = [5,6,4]
输出：[7,8,0,7]
```

**示例2：**

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[8,0,7]
```

**示例3：**

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

 

**提示：**

- 链表的长度范围为` [1, 100]`
- `0 <= node.val <= 9`
- 输入数据保证链表代表的数字无前导 0

 

**进阶：**如果输入链表不能修改该如何处理？换句话说，不能对列表中的节点进行翻转。

 

注意：本题与主站 445 题相同：https://leetcode-cn.com/problems/add-two-numbers-ii/



## 思路

```
栈
```



```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        # 栈
        s1 = list()
        s2 = list()
        while l1:
            s1.append(l1.val)
            l1 = l1.next

        while l2:
            s2.append(l2.val)
            l2 = l2.next

        carry = 0
        res = []
        while s1 or s2 or carry != 0:
            a = 0 if not s1 else s1.pop()
            b = 0 if not s2 else s2.pop()
            val = a + b + carry
            carry = val // 10
            val = val % 10
            node = ListNode(val)
            res.append(node)

        dummy = ListNode()
        cur = dummy
        for item in res[::-1]:
            cur.next = item
            cur = cur.next

        return dummy.next
```

