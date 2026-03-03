---
layout: post
title: "160. 相交链表 题解"
date: 2026-03-03
categories: [LeetCode, 算法]
tags: [链表, 双指针, 哈希表]
---

## 题目

[原题链接](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

给你两个单链表的头节点 `headA` 和 `headB`，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 `null`。

**要求：** O(n) 时间复杂度，O(1) 空间复杂度。

## 解题思路

### 解法一：双指针（推荐）⭐

最优雅的解法：
- pA 从 headA 走，走完后切换到 headB 继续
- pB 从 headB 走，走完后切换到 headA 继续
- 两指针必在相交点相遇（或同时到 null）

**原理：** `a + c + b = b + c + a`（a/b 是独有部分，c 是共享部分）

```typescript
function getIntersectionNode(headA: ListNode | null, headB: ListNode | null): ListNode | null {
    if (!headA || !headB) return null;
    
    let pA = headA;
    let pB = headB;
    
    while (pA !== pB) {
        pA = pA ? pA.next : headB;
        pB = pB ? pB.next : headA;
    }
    
    return pA; // 相交点或 null
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)
- 空间复杂度：O(1)

---

### 解法二：长度差法

先计算两个链表的长度，让较长的链表先走差值步，然后同步前进比较。

```typescript
function getIntersectionNode(headA: ListNode | null, headB: ListNode | null): ListNode | null {
    // 计算长度
    const getLen = (head: ListNode | null): number => {
        let len = 0;
        while (head) { len++; head = head.next; }
        return len;
    };
    
    let lenA = getLen(headA), lenB = getLen(headB);
    
    // 对齐起点
    while (lenA > lenB) { headA = headA!.next; lenA--; }
    while (lenB > lenA) { headB = headB!.next; lenB--; }
    
    // 同时前进
    while (headA !== headB) {
        headA = headA!.next;
        headB = headB!.next;
    }
    
    return headA;
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)
- 空间复杂度：O(1)

---

### 解法三：哈希表

用 Set 记录链表 A 的所有节点，然后遍历链表 B 查找第一个出现的节点。

```typescript
function getIntersectionNode(headA: ListNode | null, headB: ListNode | null): ListNode | null {
    const visited = new Set<ListNode>();
    
    while (headA) {
        visited.add(headA);
        headA = headA.next;
    }
    
    while (headB) {
        if (visited.has(headB)) return headB;
        headB = headB.next;
    }
    
    return null;
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)
- 空间复杂度：O(m) 或 O(n)

---

## 总结

推荐使用 **双指针法**，代码简洁、空间最优、思路巧妙！

核心思想是让两个指针走相同的总路程，这样它们一定会在相交点相遇。
