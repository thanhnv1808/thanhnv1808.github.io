---
title: "LeetCode 14: Longest Common Prefix - Two Efficient Solutions"
author: thanhnv1808
date: 2025-11-24 14:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, string, prefix]
description: Learn two efficient approaches to find the longest common prefix among an array of strings, from horizontal scanning to vertical comparison.
comments: true
---

## Problem Statement

Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string `""`.

**Link**: [LeetCode 14 - Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/description/)

### Examples

**Example 1:**
```plaintext
Input: strs = ["flower","flow","flight"]
Output: "fl"
```
{: .nolineno }

**Example 2:**
```plaintext
Input: strs = ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```
{: .nolineno }

### Constraints

- `1 <= strs.length <= 200`
- `0 <= strs[i].length <= 200`
- `strs[i]` consists of only lowercase English letters if it is non-empty.

---

## Solution 1: Horizontal Scanning

The first approach uses horizontal scanning, where we start with the first string as the prefix and gradually reduce it by comparing with each subsequent string.

### Approach

1. Start with the first string as the initial prefix
2. For each remaining string in the array:
   - While the current string doesn't start with the prefix:
     - Remove the last character from the prefix
     - If the prefix becomes empty, return `""`
3. Return the final prefix

### Complexity Analysis

- **Time Complexity**: O(S) where S is the sum of all characters in all strings. In the worst case, we compare each character of each string.
- **Space Complexity**: O(1) - We only use a constant amount of extra space for the prefix variable.

### Code

```typescript
function longestCommonPrefix(strs: string[]): string {
    if (strs.length === 0) {
        return "";
    }
    
    let prefix = strs[0];
    
    for (let i = 1; i < strs.length; i++) {
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length - 1);
            if (prefix === "") {
                return "";
            }
        }
    }
    
    return prefix;
}
```

### Explanation

The horizontal scanning approach:
- Uses the first string as a baseline prefix
- Iteratively shortens the prefix when it doesn't match the beginning of other strings
- Early returns when the prefix becomes empty
- Simple and intuitive to understand

### Example Trace

For `strs = ["flower","flow","flight"]`:

```
Initial: prefix = "flower"

Compare with "flow":
  "flow".startsWith("flower")? No
  prefix = "flowe"
  "flow".startsWith("flowe")? No
  prefix = "flow"
  "flow".startsWith("flow")? Yes ✓

Compare with "flight":
  "flight".startsWith("flow")? No
  prefix = "flo"
  "flight".startsWith("flo")? No
  prefix = "fl"
  "flight".startsWith("fl")? Yes ✓

Result: "fl"
```

---

## Solution 2: Vertical Scanning

The second approach uses vertical scanning, where we compare characters at the same position across all strings.

### Approach

1. Use the first string as a reference
2. For each character position `j` in the first string:
   - Get the character at position `j` from the first string
   - Compare it with the character at position `j` in all other strings
   - If any string is shorter than `j` or has a different character, return the prefix up to position `j`
3. If all characters match, return the entire first string

### Complexity Analysis

- **Time Complexity**: O(S) where S is the sum of all characters. In the best case, we only check the minimum length string.
- **Space Complexity**: O(1) - We only use a constant amount of extra space.

### Code

```typescript
function longestCommonPrefix(strs: string[]): string {
    if (strs.length === 0) {
        return "";
    }
    
    for (let j = 0; j < strs[0].length; j++) {
        const char = strs[0][j];
        
        for (let i = 1; i < strs.length; i++) {
            if (j >= strs[i].length || strs[i][j] !== char) {
                return strs[0].substring(0, j);
            }
        }
    }
    
    return strs[0];
}
```

### Explanation

The vertical scanning approach:
- Compares characters at the same index across all strings
- Stops as soon as it finds a mismatch or reaches the end of any string
- More efficient in cases where the common prefix is short
- Processes characters column by column

### Example Trace

For `strs = ["flower","flow","flight"]`:

```
Position 0: Compare 'f' across all strings
  strs[0][0] = 'f'
  strs[1][0] = 'f' ✓
  strs[2][0] = 'f' ✓

Position 1: Compare 'l' across all strings
  strs[0][1] = 'l'
  strs[1][1] = 'l' ✓
  strs[2][1] = 'l' ✓

Position 2: Compare 'o' and 'i'
  strs[0][2] = 'o'
  strs[1][2] = 'o' ✓
  strs[2][2] = 'i' ✗ (mismatch!)

Return: strs[0].substring(0, 2) = "fl"
```

---

## Comparison of Solutions

| Solution | Time Complexity | Space Complexity | Best Case | Worst Case |
|----------|----------------|------------------|-----------|------------|
| Horizontal Scanning | O(S) | O(1) | When prefix is long | When prefix is short |
| Vertical Scanning | O(S) | O(1) | When prefix is short | When prefix is long |

> 💡 **Note**: Both solutions have the same time complexity O(S), but vertical scanning can be more efficient when the common prefix is short, as it stops early. Horizontal scanning might be more intuitive for beginners.
{: .prompt-info }

---

## Key Takeaways

1. **Horizontal scanning** is intuitive - start with one string and reduce it based on comparisons
2. **Vertical scanning** can be more efficient - compare character by character and stop at the first mismatch
3. **Early termination** is important - both solutions stop as soon as they determine the result
4. **Edge cases matter** - handle empty arrays and strings of different lengths
5. **String methods** like `startsWith()` and `substring()` are useful for prefix problems

---

## Related Problems

- [LeetCode 1392: Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/)
- [LeetCode 1044: Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/)
- [LeetCode 792: Number of Matching Subsequences](https://leetcode.com/problems/number-of-matching-subsequences/)

---

## Practice

Try solving this problem on LeetCode: [14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

