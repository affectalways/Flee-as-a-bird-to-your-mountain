# [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

 

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
  [20,9],
  [15,7]
]
```

**提示：**

- 节点总数 <= 1000

  

## 思路

**层次遍历+奇偶行判断**

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
        level = 0
        while stack:
            path = []
            tmp = []
            while stack:
                cur = stack.pop(0)
                # ans.append(cur.val)
                tmp.append(cur.val)
                if cur.left:
                    path.append(cur.left)
                if cur.right:
                    path.append(cur.right)

            ans.append(tmp)
            # 奇偶行判断！！！
            if level % 2 == 1:
                stack.extend(path[:])
            else:
                stack.extend(path[::-1])

            level += 1

        return ans
```

