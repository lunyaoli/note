---
layout: post
title: "236. 二叉树的最近公共祖先 题解"
date: 2026-03-03
categories: [LeetCode, 算法]
tags: [二叉树, 递归, LCA, 中等]
---

## 题目描述

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为："对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。"

### 示例 1

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出：3
解释：节点 5 和节点 1 的最近公共祖先是节点 3 。
```

### 示例 2

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出：5
解释：节点 5 和节点 4 的最近公共祖先是节点 5 ，因为根据定义最近公共祖先节点可以为节点本身。
```

### 示例 3

```
输入：root = [1,2], p = 1, q = 2
输出：1
```

## 解题思路

这道题是二叉树中非常经典的问题。我们需要找到两个节点 `p` 和 `q` 的最近公共祖先（LCA）。

### 核心思想

使用**递归后序遍历**的思路：

1. 从根节点开始递归遍历
2. 如果当前节点等于 `p` 或 `q`，直接返回当前节点
3. 递归在左子树中查找，得到 `left`
4. 递归在右子树中查找，得到 `right`
5. **关键判断**：
   - 如果 `left` 和 `right` 都不为空，说明 `p` 和 `q` 分别在当前节点的左右两侧，**当前节点就是 LCA**
   - 如果只有 `left` 不为空，说明 `p` 和 `q` 都在左子树，返回 `left`
   - 如果只有 `right` 不为空，说明 `p` 和 `q` 都在右子树，返回 `right`

### 为什么可行？

- 如果 `p` 是 `q` 的祖先（或反之），那么在遇到 `p` 时就会直接返回，不会继续向下
- 如果 `p` 和 `q` 分布在某节点的两侧，该节点就是 LCA
- 后序遍历保证了我们自底向上查找，找到的第一个"分叉点"就是 LCA

## 代码实现

### TypeScript

```typescript
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     val: number
 *     left: TreeNode | null
 *     right: TreeNode | null
 *     constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.left = (left===undefined ? null : left)
 *         this.right = (right===undefined ? null : right)
 *     }
 * }
 */

function lowestCommonAncestor(
    root: TreeNode | null,
    p: TreeNode | null,
    q: TreeNode | null
): TreeNode | null {
    // 递归终止条件
    if (root === null || root === p || root === q) {
        return root;
    }
    
    // 在左右子树中查找
    const left = lowestCommonAncestor(root.left, p, q);
    const right = lowestCommonAncestor(root.right, p, q);
    
    // 如果左右都找到了，当前节点就是 LCA
    if (left !== null && right !== null) {
        return root;
    }
    
    // 返回不为空的那一边
    return left !== null ? left : right;
}
```

### Python

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        # 递归终止条件
        if not root or root == p or root == q:
            return root
        
        # 在左右子树中查找
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)
        
        # 如果左右都找到了，当前节点就是 LCA
        if left and right:
            return root
        
        # 返回不为空的那一边
        return left if left else right
```

### Go

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    // 递归终止条件
    if root == nil || root == p || root == q {
        return root
    }
    
    // 在左右子树中查找
    left := lowestCommonAncestor(root.Left, p, q)
    right := lowestCommonAncestor(root.Right, p, q)
    
    // 如果左右都找到了，当前节点就是 LCA
    if left != nil && right != nil {
        return root
    }
    
    // 返回不为空的那一边
    if left != nil {
        return left
    }
    return right
}
```

## 复杂度分析

- **时间复杂度**: O(n)，其中 n 是二叉树的节点数。最坏情况下需要遍历所有节点。
- **空间复杂度**: O(h)，其中 h 是二叉树的高度。递归调用栈的深度最大为树的高度。
  - 最坏情况（树退化为链表）: O(n)
  - 平均情况（平衡树）: O(log n)

## 扩展思考

### 相关题目

1. **235. 二叉搜索树的最近公共祖先** - 利用 BST 性质可优化
2. **1644. 二叉树的最近公共祖先 II** - 节点可能不存在于树中
3. **1650. 二叉树的最近公共祖先 III** - 节点有父指针

### 进阶解法

如果需要多次查询 LCA，可以预处理使用：
- **Tarjan 算法** (离线 LCA)
- **树上倍增** (在线 LCA，O(nlogn) 预处理，O(logn) 查询)

## 总结

这道题体现了递归思想的优雅：将复杂问题分解为子问题，通过后序遍历自底向上地找到"分叉点"。理解"左右子树都不为空意味着当前节点是 LCA"这一关键点，是解题的核心。

---

> 原题链接: [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
