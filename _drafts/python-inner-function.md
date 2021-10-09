---
layout: post
title: 关于Python内部函数操作外部函数成员的问题
categories: Python
description: 内部函数作用域的理解
keywords: Python, inner function, scope
---

## 问题描述

在用 Python 做与二叉树遍历相关的 LeetCode 题目时，标准写法是借助一个内部函数来实现递归，这里以[前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)为例，解法如下：

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        res = []
        def traversal(root):
            if not root:
                return
            res.append(root.val)
            traversal(root.left)
            traversal(root.right)
        traversal(root)
        return res
```

可以看到，这里我们在外部函数中设立了一个 `res` 对象用来存储遍历的结果，在内部函数中可以直接对其进行操作。在做过这道题之后，我误认为这是遍历二叉树的一种模板写法，直到我遇到了[538-把BST转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)这道题，才发现在解题时还要考虑到 Python 的语言特性。

如果按照模板写法，对二叉树进行逆中序遍历，则代码如下：

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def convertBST(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        pre = 0
        def traversal(curr):
            if not curr:
                return None
            traversal(curr.right)
            curr.val += pre
            pre = curr.val
            traversal(curr.left)
        traversal(root)
        return root
```

我们的本意是在外部函数用一个`pre`来记录上一个节点的值，每次在内部函数中操作并不断更新`pre`。然而这段代码看似没有问题，在运行的时候会报错：

> UnboundLocalError: local variable 'pre' referenced before assignment

## 解释

问题出在这个`pre`上，解释器认为它在内部函数中未经声明就直接进行了操作。初次看到这个问题的时候我感到很疑惑，因为在二叉树中序遍历的题解中我的`res`也是在外部函数声明的，内部函数却可以直接运行。为此我查阅了一些资料，得到的解释如下：

1. 内部函数中使用基本类型会覆盖掉外部函数中的同名scope
2. 若外部函数中声明的类型是容器，那么无事发生，因为容器对象创建在堆上


## 解决办法

1. `nonlocal` 关键字声明
2. 将基本类型封装成容器，比如，将`pre`变成list(pre)，然后操作列表中的第一个元素即可
3. 将`pre`设置成类的属性，self.pre