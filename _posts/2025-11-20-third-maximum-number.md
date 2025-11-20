---
title: "LeetCode 414: Third Maximum Number - Multiple Solutions"
author: thanhnv1808
date: 2025-11-20 09:30:00 +0700
categories: [Algorithms, LeetCode, Array]
tags: [leetcode, algorithms, typescript, array, sorting]
description: Three different approaches to solve LeetCode's Third Maximum Number problem, from O(n log n) to optimal O(n) solutions with TypeScript implementations.
comments: true
---

## Problem Statement

Given an integer array `nums`, return the **third distinct maximum number** in this array. If the third maximum does not exist, return the maximum number.

**Link**: [LeetCode 414 - Third Maximum Number](https://leetcode.com/problems/third-maximum-number/description/)

### Examples

**Example 1:**
```plaintext
Input: nums = [3,2,1]
Output: 1
```
{: .nolineno }

Explanation: The first distinct maximum is 3, the second is 2, and the third is 1.

**Example 2:**
```plaintext
Input: nums = [1,2]
Output: 2
```
{: .nolineno }

Explanation: The first distinct maximum is 2, the second is 1. The third maximum does not exist, so return the maximum (2).

**Example 3:**
```plaintext
Input: nums = [2,2,3,1]
Output: 1
```
{: .nolineno }

Explanation: The first distinct maximum is 3, the second is 2, and the third is 1. (Both 2's are counted together since they have the same value.)

### Constraints

- `1 <= nums.length <= 10⁴`
- `-2³¹ <= nums[i] <= 2³¹ - 1`

> Follow up: Can you find an O(n) solution?
{: .prompt-info }

## Solution Approaches

Let's explore three different ways to solve this problem, progressing from straightforward to optimal.

---

## Solution 1: Sort and Find (O(n log n))

### Approach

The simplest approach is to sort the array in descending order, then iterate through to find the third distinct maximum.

**Algorithm:**
1. Sort the array in descending order
2. Iterate through the sorted array
3. Count distinct maximum values
4. Return the third maximum if it exists, otherwise return the first maximum

### TypeScript Implementation

```typescript
function thirdMax(nums: number[]): number {
    // Sort in descending order
    nums.sort((a, b) => b - a);
    
    let count = 1;
    let prev = nums[0];
    
    for (let i = 1; i < nums.length; i++) {
        // Only count distinct values
        if (nums[i] !== prev) {
            count++;
            prev = nums[i];
            
            // Found the third maximum
            if (count === 3) {
                return nums[i];
            }
        }
    }
    
    // Third maximum doesn't exist, return the first (maximum)
    return nums[0];
}
```
{: file="solution1.ts" }

### Complexity Analysis

- **Time Complexity**: O(n log n) due to sorting
- **Space Complexity**: O(1) if sorting in-place (excluding sort's internal stack space)

### Example Walkthrough

For `nums = [8, 4, 6, 2, 7, 5]`:
1. After sorting: `[8, 7, 6, 5, 4, 2]`
2. First distinct: 8
3. Second distinct: 7
4. Third distinct: 6
5. Return 6

---

## Solution 2: Three Separate Passes (O(n))

### Approach

Instead of sorting, we can make three passes through the array to find the first, second, and third maximum values separately.

**Algorithm:**
1. First pass: Find the maximum value
2. Second pass: Find the second maximum (less than first maximum)
3. Third pass: Find the third maximum (less than second maximum)
4. Return third maximum if found, otherwise return first maximum

### TypeScript Implementation

```typescript
function thirdMax(nums: number[]): number {
    // First pass: find first maximum
    let first = Math.max(...nums);
    
    // Second pass: find second maximum
    let second = -Infinity;
    for (let num of nums) {
        if (num < first && num > second) {
            second = num;
        }
    }
    
    // Third pass: find third maximum
    let third = -Infinity;
    for (let num of nums) {
        if (num < second && num > third) {
            third = num;
        }
    }
    
    // If third maximum exists, return it; otherwise return first
    return third !== -Infinity ? third : first;
}
```
{: file="solution2.ts" }

### Complexity Analysis

- **Time Complexity**: O(n) - three separate passes through the array
- **Space Complexity**: O(1)

### Example Walkthrough

For `nums = [8, 4, 6, 2, 7, 5]`:
1. First pass: first = 8
2. Second pass: second = 7 (max value < 8)
3. Third pass: third = 6 (max value < 7)
4. Return 6

> This approach is better than sorting for large arrays!
{: .prompt-tip }

---

## Solution 3: Single Pass with Three Variables (Optimal O(n))

### Approach

The most efficient solution uses a single pass through the array while maintaining three variables to track the top three distinct maximum values.

**Algorithm:**
1. Initialize three variables: `first`, `second`, `third` as `null`
2. For each number in the array:
   - If it equals any of the three maximums, skip it (avoid duplicates)
   - If it's greater than `first`, shift values: `third = second`, `second = first`, `first = num`
   - Else if greater than `second`, shift: `third = second`, `second = num`
   - Else if greater than `third`, update: `third = num`
3. Return `third` if it exists, otherwise return `first`

### TypeScript Implementation

```typescript
function thirdMax(nums: number[]): number {
    let first: number | null = null;
    let second: number | null = null;
    let third: number | null = null;
    
    for (let num of nums) {
        // Skip if num is already one of the top 3
        if (num === first || num === second || num === third) {
            continue;
        }
        
        // Update the top 3 maximums
        if (first === null || num > first) {
            third = second;
            second = first;
            first = num;
        } else if (second === null || num > second) {
            third = second;
            second = num;
        } else if (third === null || num > third) {
            third = num;
        }
    }
    
    // Return third if it exists, otherwise return first
    return third !== null ? third : first!;
}
```
{: file="solution3.ts" }

### Complexity Analysis

- **Time Complexity**: O(n) - single pass through the array
- **Space Complexity**: O(1) - only three variables used

### Example Walkthrough

For `nums = [8, 4, 6, 2, 7, 5]`:

| num | first | second | third | Action |
|-----|-------|--------|-------|--------|
| 8   | 8     | null   | null  | first = 8 |
| 4   | 8     | 4      | null  | second = 4 |
| 6   | 8     | 6      | 4     | second = 6, third = 4 |
| 2   | 8     | 6      | 4     | no change (2 < third) |
| 7   | 8     | 7      | 6     | second = 7, third = 6 |
| 5   | 8     | 7      | 6     | no change (5 < third) |

Return: `third = 6`

> This is the optimal solution - single pass with constant space!
{: .prompt-tip }

---

## Edge Cases to Consider

```typescript
// Test cases
console.log(thirdMax([3, 2, 1]));           // Output: 1
console.log(thirdMax([1, 2]));              // Output: 2
console.log(thirdMax([2, 2, 3, 1]));        // Output: 1
console.log(thirdMax([1, 1, 1]));           // Output: 1
console.log(thirdMax([5, 2, 2]));           // Output: 5
console.log(thirdMax([1, 2, -2147483648])); // Output: -2147483648
```
{: .nolineno }

> Be careful with the edge case where the third maximum could be a very small number like -2³¹!
{: .prompt-warning }

---

## Comparison Summary

| Solution | Time Complexity | Space Complexity | Passes | Best For |
|----------|----------------|------------------|---------|----------|
| Sort and Find | O(n log n) | O(1) | 1 + sort | Small arrays, simplicity |
| Three Passes | O(n) | O(1) | 3 | Medium arrays, clarity |
| Single Pass | O(n) | O(1) | 1 | Large arrays, optimal |

## Conclusion

While all three solutions work correctly, **Solution 3** is the most efficient with O(n) time complexity and a single pass through the array. It's the preferred solution for interviews and production code.

The key insight is maintaining three variables to track the top three distinct values while iterating through the array just once, handling duplicates by skipping them during iteration.

Happy coding! 🚀

