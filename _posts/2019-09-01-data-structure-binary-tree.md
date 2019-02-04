---
layout: researcher
---

> 孔子曰：温故而知新

# Traversal
When do it iteratively, use a stack and just know the node at top is the one will be visit, eight to get its children or its value.

The only difference are counting the ***time*** visited the node and the ***order*** to push itself and its children into stack.

## pre-order
visit root first, then left child, then right child. [online judge](https://leetcode.com/problems/binary-tree-preorder-traversal/)


#### Recursive solution is trivial
```python
def preorderTraversal(root: TreeNode) -> List[int]:
    if root is None:
        return []
    else:
        return [root.val] + preorderTraversal(root.left) + preorderTraversal(root.right)

```

#### iterative solution is simple as well
> the node at top of stack represent the whole sub-tree to be visited.

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

## in-order
visit left child first, then node itself, then right child. [online judge](https://leetcode.com/problems/binary-tree-inorder-traversal/)

```python

rst = []
def _trav(node):
    if node.left:
        _trav(node.left)
    rst.append(node.val)
    if node.right:
        _trav(node.right)

```

#### iterative solution is simple as well
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
```python
rst = []

def trav(node):
    if node.left:
        trav(node.left)
    if node.right:
        trav(node.right)
    rst.append(node.val)
```

#### iterative solution is simple as well
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
