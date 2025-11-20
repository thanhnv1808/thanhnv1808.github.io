---
title: "LeetCode 88: Merge Sorted Array - Three Solutions Explained"
author: thanhnv1808
date: 2025-11-20 11:50:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, typescript, array, two-pointers, merge, in-place]
description: Learn three different approaches to merge two sorted arrays in-place, from O(n log n) to optimal O(m+n) time and O(1) space solutions.
comments: true
---

## Problem Statement

You are given two integer arrays `nums1` and `nums2`, sorted in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.

Merge `nums2` into `nums1` as one sorted array.

The final sorted array should not be returned by the function, but instead be stored inside the array `nums1`. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`.

**Link**: [LeetCode 88 - Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/description/)

### Examples

**Example 1:**
```plaintext
Input: nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
Output: [1,2,2,3,5,6]
```
{: .nolineno }

Explanation: The arrays we are merging are `[1,2,3]` and `[2,5,6]`. The result of the merge is `[1,2,2,3,5,6]` with the underlined elements coming from `nums1`.

**Example 2:**
```plaintext
Input: nums1 = [1], m = 1, nums2 = [], n = 0
Output: [1]
```
{: .nolineno }

Explanation: The arrays we are merging are `[1]` and `[]`. The result of the merge is `[1]`.

**Example 3:**
```plaintext
Input: nums1 = [0], m = 0, nums2 = [1], n = 1
Output: [1]
```
{: .nolineno }

Explanation: The arrays we are merging are `[]` and `[1]`. The result of the merge is `[1]`. Note that because `m = 0`, there are no elements in `nums1`. The `0` is only there to ensure the merge result can fit in `nums1`.

### Constraints

- `nums1.length == m + n`
- `nums2.length == n`
- `0 <= m, n <= 200`
- `1 <= m + n <= 200`
- `-10⁹ <= nums1[i], nums2[j] <= 10⁹`

> Follow up: Can you come up with an algorithm that runs in O(m + n) time?
{: .prompt-info }

---

## Solution Approaches

Let's explore three different ways to solve this problem, progressing from straightforward to optimal.

---

## Solution 1: Copy and Sort (O(n log n))

### Approach

The simplest approach is to copy all elements from `nums2` into the empty slots of `nums1`, then sort the entire array.

**Algorithm:**
1. Copy all elements from `nums2` to the end of `nums1` (starting from index `m`)
2. Sort the entire `nums1` array

### TypeScript Implementation

```typescript
function merge(nums1: number[], m: number, nums2: number[], n: number): void {
    // Copy nums2 into nums1 starting from index m
    for (let i = 0; i < n; i++) {
        nums1[m + i] = nums2[i];
    }
    
    // Sort the entire array
    nums1.sort((a, b) => a - b);
}
```
{: file="solution1.ts" }

### Complexity Analysis

- **Time Complexity**: O((m + n) log(m + n)) - Due to sorting
- **Space Complexity**: O(1) - Only uses constant extra space (sorting may use O(log n) stack space)

### Example Walkthrough

For `nums1 = [1,2,3,0,0,0]`, `m = 3`, `nums2 = [2,5,6]`, `n = 3`:
1. After copying: `nums1 = [1,2,3,2,5,6]`
2. After sorting: `nums1 = [1,2,2,3,5,6]`

> This solution is simple but not optimal. We can do better by leveraging the fact that both arrays are already sorted!
{: .prompt-warning }

---

## Solution 2: Merge Using Extra Array (O(m + n))

### Approach

Since both arrays are already sorted, we can merge them efficiently using a two-pointer technique with an auxiliary array.

**Algorithm:**
1. Create a new array to store the merged result
2. Use two pointers `i` and `j` to traverse `nums1` and `nums2` respectively
3. Compare elements at both pointers and copy the smaller one to the result
4. Copy remaining elements from either array
5. Copy the merged result back to `nums1`

### TypeScript Implementation

```typescript
function merge(nums1: number[], m: number, nums2: number[], n: number): void {
    // Create a temporary array to store merged result
    const merged: number[] = new Array(m + n);
    
    let i = 0; // Pointer for nums1
    let j = 0; // Pointer for nums2
    let k = 0; // Pointer for merged array
    
    // Merge both arrays by comparing elements
    while (i < m && j < n) {
        if (nums1[i] <= nums2[j]) {
            merged[k++] = nums1[i++];
        } else {
            merged[k++] = nums2[j++];
        }
    }
    
    // Copy remaining elements from nums1
    while (i < m) {
        merged[k++] = nums1[i++];
    }
    
    // Copy remaining elements from nums2
    while (j < n) {
        merged[k++] = nums2[j++];
    }
    
    // Copy merged result back to nums1
    for (let idx = 0; idx < m + n; idx++) {
        nums1[idx] = merged[idx];
    }
}
```
{: file="solution2.ts" }

### Complexity Analysis

- **Time Complexity**: O(m + n) - We traverse both arrays once
- **Space Complexity**: O(m + n) - We use an extra array of size `m + n`

### Example Walkthrough

For `nums1 = [1,2,3,0,0,0]`, `m = 3`, `nums2 = [2,5,6]`, `n = 3`:

| Step | i | j | nums1[i] | nums2[j] | Comparison | Action | merged |
|------|---|---|----------|----------|------------|--------|--------|
| 1 | 0 | 0 | 1 | 2 | 1 < 2 | Copy 1 | `[1]` |
| 2 | 1 | 0 | 2 | 2 | 2 == 2 | Copy 2 | `[1,2]` |
| 3 | 2 | 1 | 3 | 5 | 3 < 5 | Copy 3 | `[1,2,3]` |
| 4 | 3 | 1 | - | 5 | - | Copy 5 | `[1,2,3,5]` |
| 5 | 3 | 2 | - | 6 | - | Copy 6 | `[1,2,3,5,6]` |

Final: `nums1 = [1,2,2,3,5,6]`

> This solution is better in terms of time complexity, but uses O(m + n) extra space. Can we do it in-place?
{: .prompt-tip }

---

## Solution 3: Merge from the End (Optimal O(m + n))

### Approach

The key insight is to merge **backwards** from the end of `nums1`. Since `nums1` has extra space at the end, we can fill it from right to left without overwriting unprocessed elements.

**Algorithm:**
1. Initialize three pointers:
   - `i = m - 1`: Last index of `nums1`
   - `j = n - 1`: Last index of `nums2`
   - `k = m + n - 1`: Last index of the merged array in `nums1`
2. Merge in reverse order while `j >= 0`:
   - If `i >= 0` and `nums1[i] > nums2[j]`: Place `nums1[i]` at position `k` and decrement `i`
   - Otherwise: Place `nums2[j]` at position `k` and decrement `j`
   - Decrement `k`
3. Note: We don't need to handle remaining elements from `nums1` separately because they're already in the correct position. The loop continues until all elements from `nums2` are merged.

### Visual Walkthrough

**Example 1:** For `nums1 = [1,2,3,0,0,0]`, `m = 3`, `nums2 = [2,5,6]`, `n = 3`:

| Step | i | j | k | nums1[i] | nums2[j] | Comparison | Action | nums1 |
|------|---|---|---|----------|----------|------------|--------|-------|
| Initial | 2 | 2 | 5 | 3 | 6 | 3 < 6 | `nums1[5] = 6`, j--, k-- | `[1,2,3,0,0,6]` |
| 1 | 2 | 1 | 4 | 3 | 5 | 3 < 5 | `nums1[4] = 5`, j--, k-- | `[1,2,3,0,5,6]` |
| 2 | 2 | 0 | 3 | 3 | 2 | 3 > 2 | `nums1[3] = 3`, i--, k-- | `[1,2,3,3,5,6]` |
| 3 | 1 | 0 | 2 | 2 | 2 | 2 == 2 | `nums1[2] = 2`, j--, k-- | `[1,2,2,3,5,6]` |
| 4 | 1 | -1 | 1 | 2 | - | - | Loop exits (j < 0) | `[1,2,2,3,5,6]` |

**Example 2 (Edge Case):** For `nums1 = [0]`, `m = 0`, `nums2 = [1]`, `n = 1`:

| Step | i | j | k | nums1[i] | nums2[j] | Condition Check | Action | nums1 |
|------|---|---|---|----------|----------|-----------------|--------|-------|
| Initial | -1 | 0 | 0 | N/A | 1 | `i >= 0`? No | `nums1[0] = 1` | `[1]` |
| 1 | -1 | -1 | -1 | - | - | `j >= 0`? No | Loop exits | `[1]` |

**Example 3:** For `nums1 = [2,0]`, `m = 1`, `nums2 = [1]`, `n = 1`:

| Step | i | j | k | nums1[i] | nums2[j] | Comparison | Action | nums1 |
|------|---|---|---|----------|----------|------------|--------|-------|
| Initial | 0 | 0 | 1 | 2 | 1 | 2 > 1 | `nums1[1] = 2`, i--, k-- | `[2,2]` |
| 1 | -1 | 0 | 0 | - | 1 | - | `nums1[0] = 1`, j--, k-- | `[1,2]` |
| 2 | -1 | -1 | -1 | - | - | - | Loop exits | `[1,2]` |

### TypeScript Implementation

```typescript
/**
 * Do not return anything, modify nums1 in-place instead.
 */
function merge(nums1: number[], m: number, nums2: number[], n: number): void {
    let k = m + n - 1, i = m - 1, j = n - 1;
    
    while (j >= 0) {
        if (i >= 0 && nums1[i] > nums2[j]) {
            nums1[k] = nums1[i];
            i--;
        } else {
            nums1[k] = nums2[j];
            j--;
        }
        k--;
    }
}
```
{: file="solution3.ts" }

### Complexity Analysis

- **Time Complexity**: O(m + n) - We traverse both arrays once
- **Space Complexity**: O(1) - We only use three extra variables

> This is the optimal solution! We achieve O(m + n) time and O(1) space complexity by merging backwards.
{: .prompt-tip }

### Why This Works

The key insight is that `nums1` has enough space at the end to accommodate all elements from `nums2`. By merging from right to left:

1. We only need to loop while `j >= 0` (elements in `nums2` remain). Once all elements from `nums2` are merged, we're done.
2. We check `i >= 0` inside the loop to see if we still have elements in `nums1` to compare. If `i < 0`, we simply copy remaining elements from `nums2`.
3. We never overwrite unprocessed elements from `nums1` (since we start from the end)
4. We can place elements directly in their final positions
5. We don't need any extra space, and we don't need a separate loop to copy remaining elements

---

## Comparison Summary

## Comparison Summary

| Solution | Time Complexity | Space Complexity | In-Place | Best For |
|----------|----------------|------------------|----------|----------|
| Copy and Sort | O((m+n) log(m+n)) | O(1) | Yes | Quick implementation, small arrays |
| Extra Array | O(m + n) | O(m + n) | No | When extra space is acceptable |
| Merge from End | O(m + n) | O(1) | Yes | **Optimal - interviews and production** |

---

## Key Takeaways

1. **Merging backwards** is the key to solving this in-place with O(1) space
2. Since `nums1` has extra space at the end, we can fill it from right to left
3. We never overwrite unprocessed elements because we start from the end
4. The two-pointer technique works perfectly when merging sorted arrays
5. Always consider the direction of iteration when modifying arrays in-place

---

## Conclusion

**Solution 3** (merge from the end) is the most efficient approach, achieving optimal O(m + n) time complexity and O(1) space complexity. It's the preferred solution for interviews and production code.

The key insight is recognizing that we can merge backwards without overwriting unprocessed elements, eliminating the need for extra space. This pattern is commonly used in array manipulation problems where you need to modify arrays in-place.

Mastering this backward merging technique will help you solve many similar LeetCode problems efficiently!

Happy coding! 🚀

