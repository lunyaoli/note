---
layout: post
title: "236. 二叉树的最近公共祖先 题解"
date: 2026-03-03
categories: [LeetCode, 算法]
tags: [二叉树, 递归, LCA, 中等]
---

## 题目描述

给定一个二叉树，找到该树中两个指定节点的最近公共祖先。

**最近公共祖先的定义：** 对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。

**示例 1：**

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出: 3
解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
```

**示例 2：**

```
输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出: 5
解释: 节点 5 和节点 4 的最近公共祖先是节点 5，因为根据定义最近公共祖先节点可以为节点本身。
```

**原题链接：** [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

---

## 解题思路

### 解法一：递归后序遍历（推荐）⭐

**思路：**
1. 从根节点开始递归遍历
2. 如果当前节点等于 `p` 或 `q`，直接返回当前节点
3. 递归在左、右子树中查找
4. 根据左右子树的返回结果判断 LCA 位置

**核心原理与可行性分析：**

后序遍历保证了**自底向上**的查找顺序，这是关键！

判断逻辑：
- **情况1：** `left` 和 `right` 都非空 → 说明 `p` 和 `q` 分别在当前节点的左右两侧 → **当前节点就是 LCA**
- **情况2：** 只有 `left` 非空 → `p` 和 `q` 都在左子树 → 返回 `left`（继续向上传递）
- **情况3：** 只有 `right` 非空 → `p` 和 `q` 都在右子树 → 返回 `right`（继续向上传递）
- **情况4：** 都为空 → 当前子树不包含 `p` 或 `q`，返回 `null`

**特殊情况处理：** 如果 `p` 是 `q` 的祖先（或反之），在遇到 `p` 时直接返回，不会继续向下遍历，符合"节点可以是自己的祖先"的定义。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        // 递归终止条件
        if (root == null || root == p || root == q) {
            return root;
        }
        
        // 在左右子树中查找
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        
        // 如果左右都找到了，当前节点就是 LCA
        if (left != null && right != null) {
            return root;
        }
        
        // 返回不为空的那一边
        return left != null ? left : right;
    }
}
```

**复杂度分析：**
- 时间复杂度：O(n)，最坏情况需要遍历所有节点
- 空间复杂度：O(h)，h 为树的高度（递归栈深度）
  - 最坏情况（链表）：O(n)
  - 平均情况（平衡树）：O(log n)

---

### 解法二：存储父节点路径法

**思路：**
1. 从根节点开始遍历，用 HashMap 存储每个节点的父节点
2. 从 `p` 开始向上追溯，记录所有祖先节点到 Set
3. 从 `q` 开始向上追溯，第一个出现在 Set 中的节点即为 LCA

**核心原理与可行性分析：**

LCA 是 `p` 和 `q` 向上追溯路径的**第一个公共节点**。

通过哈希表建立"子节点 → 父节点"的映射，将树转化为可向上追溯的结构。
先收集 `p` 的所有祖先，再从 `q` 向上查找，第一个交集点即为 LCA。

这类似于"相交链表"问题的思路：两条路径在某点汇合。

```java
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        // 存储每个节点的父节点
        Map<TreeNode, TreeNode> parentMap = new HashMap<>();
        dfs(root, null, parentMap);
        
        // 收集 p 的所有祖先节点
        Set<TreeNode> ancestors = new HashSet<>();
        TreeNode curr = p;
        while (curr != null) {
            ancestors.add(curr);
            curr = parentMap.get(curr);
        }
        
        // 从 q 向上查找第一个公共祖先
        curr = q;
        while (!ancestors.contains(curr)) {
            curr = parentMap.get(curr);
        }
        
        return curr;
    }
    
    private void dfs(TreeNode node, TreeNode parent, Map<TreeNode, TreeNode> parentMap) {
        if (node == null) {
            return;
        }
        parentMap.put(node, parent);
        dfs(node.left, node, parentMap);
        dfs(node.right, node, parentMap);
    }
}
```

**复杂度分析：**
- 时间复杂度：O(n)
- 空间复杂度：O(n)，需要存储所有节点的父节点映射

---

### 解法三：迭代法（栈模拟递归）

**思路：**
使用栈模拟后序遍历，同时记录访问状态，避免使用递归栈。

**核心原理与可行性分析：**

递归本质上使用系统调用栈，迭代法使用显式栈替代。
后序遍历的顺序是"左 → 右 → 根"，需要标记节点是否已访问过左右子树。

```java
import java.util.Stack;

class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        Stack<TreeNode> stack = new Stack<>();
        Stack<Integer> statusStack = new Stack<>(); // 0:未访问, 1:已访问左, 2:已访问左右
        
        TreeNode result = null;
        boolean foundP = false, foundQ = false;
        
        if (root != null) {
            stack.push(root);
            statusStack.push(0);
        }
        
        while (!stack.isEmpty()) {
            TreeNode node = stack.peek();
            int status = statusStack.pop();
            
            if (status == 0) {
                // 检查当前节点
                if (node == p) foundP = true;
                if (node == q) foundQ = true;
                if (foundP && foundQ) {
                    result = node;
                    break;
                }
                statusStack.push(1);
                if (node.left != null) {
                    stack.push(node.left);
                    statusStack.push(0);
                }
            } else if (status == 1) {
                statusStack.push(2);
                if (node.right != null) {
                    stack.push(node.right);
                    statusStack.push(0);
                }
            } else {
                stack.pop();
            }
        }
        
        return result;
    }
}
```

**复杂度分析：**
- 时间复杂度：O(n)
- 空间复杂度：O(n)

---

## 总结

| 解法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| 递归后序遍历 | O(n) | O(h) | 代码简洁，思路清晰 | 依赖递归栈 |
| 存储父节点路径 | O(n) | O(n) | 思路直观，易于理解 | 空间开销大 |
| 迭代法 | O(n) | O(n) | 避免递归栈溢出 | 代码复杂 |

**推荐：** 递归后序遍历，代码最优雅，面试首选。

**关键思想：**
- 后序遍历保证自底向上查找
- "左右都非空 = 当前是 LCA" 是核心判断
- 利用递归返回值向上传递信息

---

## 相关扩展题目

1. **235. 二叉搜索树的最近公共祖先** - 利用 BST 性质可优化，无需遍历整棵树
2. **1644. 二叉树的最近公共祖先 II** - 节点可能不存在于树中，需要额外判断
3. **1676. 二叉树的最近公共祖先 IV** - 查找多个节点的 LCA
4. **1650. 二叉树的最近公共祖先 III** - 每个节点有父指针，可从下往上找
5. **1123. 最深叶节点的最近公共祖先** - LCA 变体应用

**进阶知识：**
- 对于多次 LCA 查询，可预处理使用 **Tarjan 算法**（离线）或 **树上倍增**（在线，O(nlogn) 预处理，O(logn) 查询）
