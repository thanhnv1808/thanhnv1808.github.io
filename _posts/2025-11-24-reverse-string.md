---
title: "LeetCode 344: Reverse String - Three Solutions Explained"
author: thanhnv1808
date: 2025-11-24 10:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, typescript, string, two-pointers, recursion, stack, in-place]
description: Learn three different approaches to reverse a string in-place, from two-pointer technique to recursion and stack-based solutions.
comments: true
---

## Problem Statement

Write a function that reverses a string. The input string is given as an array of characters `s`.

You must do this by modifying the input array **in-place** with **O(1) extra memory**.

**Link**: [LeetCode 344 - Reverse String](https://leetcode.com/problems/reverse-string/description/)

### Examples

**Example 1:**
```plaintext
Input: s = ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]
```
{: .nolineno }

**Example 2:**
```plaintext
Input: s = ["H","a","n","n","a","h"]
Output: ["h","a","n","n","a","H"]
```
{: .nolineno }

### Constraints

- `1 <= s.length <= 10^5`
- `s[i]` is a printable ascii character.

---

## Solution 1: Two-Pointer Technique

The most intuitive and efficient approach is to use two pointers - one starting from the left and one from the right, swapping characters until they meet in the middle.

### Approach

1. Initialize two pointers: `l` at the start (index 0) and `r` at the end (index `s.length - 1`)
2. While `l < r`:
   - Swap `s[l]` and `s[r]`
   - Move `l` forward and `r` backward
3. The string is reversed when the pointers meet

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through half of the array
- **Space Complexity**: O(1) - Only using a constant amount of extra space for the temporary variable

### Code

```typescript
/**
 * Do not return anything, modify s in-place instead.
 */
function reverseString(s: string[]): void {
    let l = 0, r = s.length - 1;
    
    while (l < r) {
        // Swap characters
        let temp = s[l];
        s[l] = s[r];
        s[r] = temp;
        
        l++;
        r--;
    }
}
```

### Explanation

The two-pointer technique is optimal for this problem because:
- It requires only O(1) extra space (just the temporary variable)
- It processes the array in a single pass
- It's easy to understand and implement

---

## Solution 2: Recursive Approach

We can solve this problem recursively by swapping the first and last characters, then recursively reversing the substring in between.

### Approach

1. Create a helper function that takes the array and two indices (left and right)
2. Base case: if `l >= r`, return (nothing to swap)
3. Recursive case:
   - Swap `s[l]` and `s[r]`
   - Recursively call with `l+1` and `r-1`

### Complexity Analysis

- **Time Complexity**: O(n) - We make n/2 recursive calls
- **Space Complexity**: O(n) - Due to the recursion stack (implicit extra memory)

> ⚠️ **Note**: While this solution is elegant, it uses O(n) space due to the recursion stack, which violates the O(1) space requirement. However, it's still a valid approach to understand recursion.

### Code

```typescript
/**
 * Do not return anything, modify s in-place instead.
 */
function reverseString(s: string[]): void {
    reverse(s, 0, s.length - 1);
}

function reverse(s: string[], l: number, r: number): void {
    if (l >= r) return;
    
    // Swap characters
    let temp = s[l];
    s[l] = s[r];
    s[r] = temp;
    
    // Recursively reverse the substring
    reverse(s, l + 1, r - 1);
}
```

### Explanation

The recursive approach:
- Breaks down the problem into smaller subproblems
- Each recursive call handles one pair of characters
- The base case stops when there's nothing left to swap

---

## Solution 3: Stack-Based Approach

We can use a stack to reverse the string by pushing all characters onto the stack, then popping them back into the array.

### Approach

1. Push all characters from the array onto a stack
2. Pop characters from the stack and assign them back to the array from the beginning

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through the array twice (push and pop)
- **Space Complexity**: O(n) - We need a stack to store all characters

> ⚠️ **Note**: This approach uses O(n) extra space for the stack, which violates the O(1) space requirement. It's included here for educational purposes to show alternative thinking.

### Code

```typescript
/**
 * Do not return anything, modify s in-place instead.
 */
function reverseString(s: string[]): void {
    const stack: string[] = [];
    
    // Push all characters onto stack
    for (let i = 0; i < s.length; i++) {
        stack.push(s[i]);
    }
    
    // Pop from stack and set to array
    for (let i = 0; i < s.length; i++) {
        s[i] = stack.pop()!;
    }
}
```

### Explanation

The stack-based approach:
- Uses the Last-In-First-Out (LIFO) property of stacks
- Naturally reverses the order of elements
- Requires extra space proportional to the input size

---

## Comparison of Solutions

| Solution | Time Complexity | Space Complexity | Notes |
|----------|----------------|------------------|-------|
| Two-Pointer | O(n) | O(1) | ✅ Optimal, meets all requirements |
| Recursion | O(n) | O(n) | ⚠️ Uses recursion stack |
| Stack | O(n) | O(n) | ⚠️ Requires extra array space |

---

## Key Takeaways

1. **Two-pointer technique** is the optimal solution for this problem, meeting both time and space requirements
2. **Recursion** provides an elegant solution but uses implicit stack space
3. **Stack-based approach** demonstrates alternative thinking but doesn't meet the space constraint
4. When space is a constraint, always prefer iterative solutions over recursive ones
5. The two-pointer technique is a powerful pattern for array manipulation problems

---

## Related Problems

- [LeetCode 541: Reverse String II](https://leetcode.com/problems/reverse-string-ii/)
- [LeetCode 151: Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)
- [LeetCode 186: Reverse Words in a String II](https://leetcode.com/problems/reverse-words-in-a-string-ii/)

---

## Practice

Try solving this problem on LeetCode: [344. Reverse String](https://leetcode.com/problems/reverse-string/)

