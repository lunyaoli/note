---
title: "LeetCode 1758. 生成交替二进制字符串的最少操作数"
date: 2026-03-05 18:31:00 +0800
categories: [LeetCode]
tags: [算法, 力扣, 贪心, 字符串]
math: false
mermaid: false
---

## 题目链接

[1758. 生成交替二进制字符串的最少操作数](https://leetcode.cn/problems/minimum-changes-to-make-alternating-binary-string/description/?envType=daily-question&envId=2026-03-05)

## 题目描述

给你一个仅由字符 `'0'` 和 `'1'` 组成的字符串 `s`。一次操作中，你可以将一个字符 `'0'` 变成 `'1'`，或者将一个字符 `'1'` 变成 `'0'`。

返回使 `s` 变成**交替字符串**所需的最少操作数。

**交替字符串**的定义：相邻字符之间不同的字符串。例如，字符串 `"010"` 和 `"1010"` 是交替字符串，但字符串 `"0100"` 不是。

### 示例 1

```
输入：s = "0100"
输出：1
解释：将最后一个字符 '0' 变成 '1'，得到 "0101"，是交替字符串。
```

### 示例 2

```
输入：s = "10"
输出：0
解释：s 已经是交替字符串。
```

### 示例 3

```
输入：s = "1111"
输出：2
解释：需要 2 次操作才能得到交替字符串。
```

### 提示

- `1 <= s.length <= 10^4`
- `s[i]` 为 `'0'` 或 `'1'`

## 解题思路

### 关键洞察

交替字符串只有**两种可能**的形式：

1. 以 `'0'` 开头：`01010101...`
2. 以 `'1'` 开头：`10101010...`

因此，我们只需要分别计算字符串变成这两种形式需要的操作数，然后取最小值即可。

### 实现方法

遍历字符串，对于每个位置 `i`：

- **形式 1（0101...）**：偶数位置（索引 0, 2, 4...）应为 `'0'`，奇数位置应为 `'1'`
- **形式 2（1010...）**：偶数位置应为 `'1'`，奇数位置应为 `'0'`

统计与每种形式的差异个数，取较小值。

### 优化思路

观察到两种形式**完全相反**，如果与形式 1 有 `diff` 个位置不同，那么与形式 2 一定有 `n - diff` 个位置不同。

所以只需统计与其中一种形式的差异，答案就是 `min(diff, n - diff)`。

## 代码实现

### 方法一：分别统计两种形式

```typescript
function minOperations(s: string): number {
    let count0 = 0; // 变成 0101... 形式的操作数
    let count1 = 0; // 变成 1010... 形式的操作数
    
    for (let i = 0; i < s.length; i++) {
        const expected0 = i % 2 === 0 ? '0' : '1'; // 0101形式第i位期望值
        const expected1 = i % 2 === 0 ? '1' : '0'; // 1010形式第i位期望值
        
        if (s[i] !== expected0) count0++;
        if (s[i] !== expected1) count1++;
    }
    
    return Math.min(count0, count1);
}
```

### 方法二：优化版本

```typescript
function minOperations(s: string): number {
    let diff = 0; // 与 0101... 形式的差异个数
    
    for (let i = 0; i < s.length; i++) {
        // 0101形式: 偶数位应为0，奇数位应为1
        if (s[i] !== (i % 2 === 0 ? '0' : '1')) {
            diff++;
        }
    }
    
    // 与 1010... 形式的差异 = n - diff
    return Math.min(diff, s.length - diff);
}
```

### Python 版本

```python
class Solution:
    def minOperations(self, s: str) -> int:
        diff = 0
        for i, c in enumerate(s):
            expected = '0' if i % 2 == 0 else '1'
            if c != expected:
                diff += 1
        return min(diff, len(s) - diff)
```

## 复杂度分析

- **时间复杂度**：O(n)，其中 n 是字符串长度，只需一次遍历
- **空间复杂度**：O(1)，只使用常量空间

## 总结

这道题的关键在于认识到交替字符串只有两种可能形式，从而将问题转化为简单的计数问题。通过观察两种形式的互补关系，可以进一步优化只需统计一种形式的差异即可。
