---
layout: researcher
---

> 孔子曰：温故而知新

# Traversal

## Iteratively breadth first search
use `queue`.

## Iteratively depth first search
use `stack`.

First time when a node is pushed into `stack`, is when its parent was visited and it's pushed into `stack` as the child.

First time when a node is popped out from `stack`, is the time to visit the ***subtree*** represented by this node.

Hence, after a node is popped firstly, the order re-push itself and its children will determine the final order of visiting.


## in-order
visit left child first, then node itself, then right child. [online judge](https://leetcode.com/problems/binary-tree-inorder-traversal/)

pay attention to the order to push current node and its children, which determines the order of traversal
```python
def inorderTraversal(root: TreeNode) -> List[int]:
        if root is None:
            return []
        stack = [(root, 0)]
        res = []
        while len(stack) > 0:
            node, cnt = stack.pop()
            if cnt == 0:
                if node.right:
                    stack.append((node.right, 0))
                stack.append((node, 1))
                if node.left:
                    stack.append((node.left, 0))

            elif cnt== 1:
                res.append(node.val)
        return res
```

## post-order
visit left child first, then right child, finally node itself. [online judge](https://leetcode.com/problems/binary-tree-postorder-traversal/)

pay attention to the order to push current node and its children, which determines the order of traversal
```python
def postorderTraversal(root: TreeNode) -> List[int]:
    if root is None:
        return []

    res = []
    stack = [(root, 0)]
    while stack:
        node, cnt = stack.pop()
        if cnt == 0:
            stack.append((node, 1))
            if node.right:
                stack.append((node.right, 0))
            if node.left:
                stack.append((node.left, 0))
        elif cnt == 1:
            res.append(node.val)
    return res

```

## pre-order
visit root first, then left child, then right child. [online judge](https://leetcode.com/problems/binary-tree-preorder-traversal/)

It's the simplest one because the node will be visited when it's firstly poped. Actually we still can image that it's popped imediately after it's pushed in the third order.

```python
def preorderTraversal(root: TreeNode) -> List[int]:
        if root is None:
            return []

        res = []
        stack = [root]
        while len(stack) > 0:
            curr = stack.pop()
            res.append(curr.val)
            if curr.right:
                stack.append(curr.right)
            if curr.left:
                stack.append(curr.left)
        return res
```

