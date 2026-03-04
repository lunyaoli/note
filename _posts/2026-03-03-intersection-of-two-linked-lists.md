---
layout: post
title: "160. 相交链表 题解"
date: 2026-03-03
categories: [LeetCode, 算法]
tags: [链表, 双指针, 哈希表, 简单]
---

## 题目描述

给你两个单链表的头节点 `headA` 和 `headB`，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 `null`。

图示两个链表在节点 `c1` 开始相交：

```
A:       a1 → a2
                ↘
                 c1 → c2 → c3
                ↗
B: b1 → b2 → b3
```

**题目保证：** 整个链式结构中不存在环。

**进阶要求：** O(n) 时间复杂度，O(1) 空间复杂度。

**原题链接：** [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

---

## 解题思路

### 解法一：双指针法（最优解）⭐

**思路：**
- 指针 `pA` 从 `headA` 出发，走完后切换到 `headB` 继续走
- 指针 `pB` 从 `headB` 出发，走完后切换到 `headA` 继续走
- 两指针必在相交点相遇，或同时走到 `null`

**核心原理与可行性分析：**

设：
- `a` = 链表 A 独有部分的长度
- `b` = 链表 B 独有部分的长度  
- `c` = 相交部分的长度

```
链表A长度 = a + c
链表B长度 = b + c
```

当两指针走完各自链表后切换到对方链表：
- `pA` 走了 `a + c + b` 步
- `pB` 走了 `b + c + a` 步

由于 `a + c + b = b + c + a`，两指针**一定同时到达相交点**！

若不相交，则 `c = 0`，两指针同时走到 `null`，也满足条件。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }
        
        ListNode pA = headA;
        ListNode pB = headB;
        
        while (pA != pB) {
            pA = (pA == null) ? headB : pA.next;
            pB = (pB == null) ? headA : pB.next;
        }
        
        return pA; // 相交点或 null
    }
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)，m、n 分别为两链表长度
- 空间复杂度：O(1)，只使用两个指针

---

### 解法二：长度差法

**思路：**
1. 先分别计算两个链表的长度
2. 让较长的链表先走"长度差"步，使两链表剩余长度相同
3. 然后同步前进，第一个相遇的节点即为相交点

**核心原理与可行性分析：**

假设链表 A 较长，长度差 `diff = lenA - lenB`。
让 A 先走 `diff` 步后，A 和 B 的剩余长度相同。
此时同步前进，若相交则必在相交点相遇；若不相交则同时走到 `null`。

此方法通过**对齐起跑线**，将问题转化为两个等长链表的比较问题。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        // 计算长度
        int lenA = getLength(headA);
        int lenB = getLength(headB);
        
        // 对齐起点
        while (lenA > lenB) {
            headA = headA.next;
            lenA--;
        }
        while (lenB > lenA) {
            headB = headB.next;
            lenB--;
        }
        
        // 同步前进寻找相交点
        while (headA != headB) {
            headA = headA.next;
            headB = headB.next;
        }
        
        return headA;
    }
    
    private int getLength(ListNode head) {
        int len = 0;
        while (head != null) {
            len++;
            head = head.next;
        }
        return len;
    }
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)
- 空间复杂度：O(1)

---

### 解法三：哈希表法

**思路：**
1. 遍历链表 A，将所有节点存入 HashSet
2. 遍历链表 B，检查每个节点是否在 Set 中
3. 第一个存在的节点即为相交点

**核心原理与可行性分析：**

相交意味着**节点引用相同**（不仅仅是 val 相同）。
利用 HashSet 的 O(1) 查找特性，快速判断节点是否访问过。
由于题目保证无环，故第一个重复出现的节点一定是相交点。

```java
import java.util.HashSet;
import java.util.Set;

public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        Set<ListNode> visited = new HashSet<>();
        
        // 记录链表 A 的所有节点
        ListNode temp = headA;
        while (temp != null) {
            visited.add(temp);
            temp = temp.next;
        }
        
        // 遍历链表 B，查找第一个出现的节点
        temp = headB;
        while (temp != null) {
            if (visited.contains(temp)) {
                return temp;
            }
            temp = temp.next;
        }
        
        return null;
    }
}
```

**复杂度分析：**
- 时间复杂度：O(m + n)
- 空间复杂度：O(m) 或 O(n)，需要存储其中一个链表的所有节点

---

## 总结

| 解法 | 时间复杂度 | 空间复杂度 | 优点 | 缺点 |
|------|-----------|-----------|------|------|
| 双指针法 | O(m+n) | O(1) | 代码简洁，空间最优 | 理解有一定门槛 |
| 长度差法 | O(m+n) | O(1) | 思路直观 | 需要遍历两次 |
| 哈希表法 | O(m+n) | O(m) | 思路简单 | 空间复杂度高 |

**推荐：** 双指针法，代码最优雅，空间最优。

**关键思想：** 让两个指针走相同的总路程 `a + c + b = b + c + a`，必在相交点相遇。

---

## 相关扩展题目

1. **141. 环形链表** - 判断链表是否有环（快慢指针）
2. **142. 环形链表 II** - 找到环的入口节点
3. **19. 删除链表的倒数第 N 个结点** - 双指针应用
4. **206. 反转链表** - 链表基础操作
5. **21. 合并两个有序链表** - 双指针合并
