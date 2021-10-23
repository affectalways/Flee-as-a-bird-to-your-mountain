# [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

 

为了让您更好地理解问题，以下面的二叉搜索树为例：

![](https://assets.leetcode.com/uploads/2018/10/12/bstdlloriginalbst.png)

我们希望将这个二叉搜索树转化为双向循环链表。链表中的每个节点都有一个前驱和后继指针。对于双向循环链表，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。

下图展示了上面的二叉搜索树转化成的链表。“head” 表示指向链表中有最小元素的节点。

 ![](https://assets.leetcode.com/uploads/2018/10/12/bstdllreturndll.png)



 

特别地，我们希望可以就地完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。

 

注意：本题与主站 426 题相同：https://leetcode-cn.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/

注意：此题对比原题有改动。



## 解题思路

**二叉搜索树的中序遍历：递增序列**

- 用两个队列实现，一个队列p用于保存遍历的节点，另一个队列q保存中序遍历的递增序列
- p队列，先进先出，如果正在遍历的节点有左子节点，就把正在遍历的节点append进p队列，并把左子节点也append进p队列，并把p的左子节点置为None
- 如果正在遍历的左子节点为空，就把正在遍历的节点append进q队列
- 如果正在遍历的右子节点不为空，就把右子节点append进p队列
- 直至p为空结束遍历
- 此时q是一个递增序列
- 然后构造新节点，创建新的双向链表就可以了
- 考虑特殊情况，根节点为空

**以下代码可以优化**

```
class Solution:
    def treeToDoublyList(self, root):
        if not root:
            return None
        # 非递归实现中序遍历
        stack = [root]
        new_stack = []
        while stack:
            cur = stack.pop()
            if cur.left:
                stack.append(cur)
                l = cur.left
                cur.left = None
                stack.append(l)
                continue

            new_stack.append(cur.val)
            if cur.right:
                stack.append(cur.right)

        head = Node(-1)
        cur = head
        for val in new_stack:
            new_node = Node(val)
            cur.right = new_node
            new_node.left = cur
            cur = cur.right

        cur.right = head.right
        head.right.left = cur

        return head.right
```

