# [剑指 Offer 55 - II. 平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/)

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

 

**示例 1:**

给定二叉树` [3,9,20,null,null,15,7]`

        3
       / \
      9  20
        /  \
       15   7
    返回 true 。


**示例 2:**

给定二叉树 `[1,2,2,3,3,null,null,4,4]`

           1
          / \
         2   2
        / \
        3   3
      / \
     4   4
     
    返回 false 。



**限制：**

- 0 <= 树的结点个数 <= 10000

**注意：**本题与主站 110 题相同：https://leetcode-cn.com/problems/balanced-binary-tree/



## 思路

**dfs获取所有路径，取最大路径长度、最小路径长度，两者相减，大于一，就不是平衡二叉树**

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:
    def isBalanced(self, root: TreeNode):
        # dfs
        if not root:
            return True

        stack = [(root, 0)]
        ans = []
        paths = []
        while stack:
            cur, depth = stack.pop()
            while len(paths) > depth:
                paths.pop()

            paths.append(cur.val)
            if cur.right:
                stack.append((cur.right, depth + 1))

            if cur.left:
                stack.append((cur.left, depth + 1))

            # 注意：只要有一个左右子节点为空，就把该路径长度添加进结果集
            if not cur.left or not cur.right:
                ans.append(len(paths[:]))
                # ans.append(paths[:])
        return True if max(ans) - min(ans) <= 1 else False
```

