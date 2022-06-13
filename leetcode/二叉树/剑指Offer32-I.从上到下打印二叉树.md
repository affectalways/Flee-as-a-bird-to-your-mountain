# [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

 

例如:
给定二叉树:` [3,9,20,null,null,15,7]`,

       3
      / \
      9  20
        /  \
       15   7

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```

**提示：**

- 节点总数 <= 1000

  

**注意：**本题与主站 102 题相同：https://leetcode-cn.com/problems/binary-tree-level-order-traversal/



## 思路

**层次遍历**

```python
class TreeNode:
  def __init__(self, x):
    self.val = x
    self.left = None
    self.right = None


class Solution:
  def levelOrder(self, root: TreeNode):
    # bfs
    ans = []
    if not root:
      return ans

    stack = [root]
    while stack:
      path = []
      while stack:
        cur = stack.pop(0)
        ans.append(cur.val)
        if cur.left:
          path.append(cur.left)
        if cur.right:
          path.append(cur.right)

      stack.extend(path[:])

    return ans
```

