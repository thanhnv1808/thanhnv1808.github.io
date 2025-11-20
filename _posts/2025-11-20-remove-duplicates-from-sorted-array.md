---
title: "LeetCode 26: Remove Duplicates from Sorted Array - Two Pointer Solution"
author: thanhnv1808
date: 2025-11-20 11:10:00 +0700
categories: [Algorithms, LeetCode, Array]
tags: [leetcode, algorithms, typescript, array, two-pointers, in-place]
description: Learn how to remove duplicates from a sorted array in-place using the two-pointer technique, achieving O(n) time and O(1) space complexity.
comments: true
---

## Problem Statement

Given an integer array `nums` sorted in **non-decreasing order**, remove the duplicates **in-place** such that each unique element appears only **once**. The relative order of the elements should be kept the **same**.

Consider the number of unique elements in `nums` to be `k`. After removing duplicates, return the number of unique elements `k`.

The first `k` elements of `nums` should contain the unique numbers in sorted order. The remaining elements beyond index `k - 1` can be ignored.

**Link**: [LeetCode 26 - Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)

### Custom Judge

The judge will test your solution with the following code:

```typescript
let nums = [...]; // Input array
let expectedNums = [...]; // The expected answer with correct length

let k = removeDuplicates(nums); // Calls your implementation

assert k === expectedNums.length;
for (let i = 0; i < k; i++) {
    assert nums[i] === expectedNums[i];
}
```

If all assertions pass, then your solution will be accepted.

### Examples

**Example 1:**
```plaintext
Input: nums = [1,1,2]
Output: 2, nums = [1,2,_]
```
{: .nolineno }

Explanation: Your function should return `k = 2`, with the first two elements of `nums` being `1` and `2` respectively. It does not matter what you leave beyond the returned `k` (hence they are underscores).

**Example 2:**
```plaintext
Input: nums = [0,0,1,1,1,2,2,3,3,4]
Output: 5, nums = [0,1,2,3,4,_,_,_,_,_]
```
{: .nolineno }

Explanation: Your function should return `k = 5`, with the first five elements of `nums` being `0, 1, 2, 3,` and `4` respectively. It does not matter what you leave beyond the returned `k` (hence they are underscores).

### Constraints

- `1 <= nums.length <= 3 * 10⁴`
- `-100 <= nums[i] <= 100`
- `nums` is sorted in **non-decreasing order**

> The key challenge here is modifying the array **in-place** without using extra space for another array!
{: .prompt-info }

---

## Solution Approach: Two Pointers

### Intuition

Since the array is already sorted, all duplicate elements will be adjacent to each other. We can use two pointers to solve this problem efficiently:

- **Pointer `j`**: Traverses through the entire array to find unique elements
- **Pointer `i`**: Tracks the position where the next unique element should be placed

### Algorithm

1. Initialize both pointers `i` and `j` starting at index `1` (since the first element is always unique)
2. For each position `j`:
   - If `nums[j] != nums[j-1]`, it means we found a new unique element
   - Place this unique element at position `i`: `nums[i] = nums[j]`
   - Increment `i` to prepare for the next unique element
3. Increment `j` to continue scanning
4. Return `i`, which represents the count of unique elements

### Visual Walkthrough

For `nums = [0,0,1,1,1,2,2,3,3,4]`:

| Step | i | j | nums[j] | nums[j-1] | Condition | Action | Array State |
|------|---|---|---------|-----------|-----------|--------|-------------|
| Initial | 1 | 1 | 0 | 0 | Equal | j++ | `[0,0,1,1,1,2,2,3,3,4]` |
| 1 | 1 | 2 | 1 | 0 | Not equal | `nums[1]=1`, i++ | `[0,1,1,1,1,2,2,3,3,4]` |
| 2 | 2 | 3 | 1 | 1 | Equal | j++ | `[0,1,1,1,1,2,2,3,3,4]` |
| 3 | 2 | 4 | 1 | 1 | Equal | j++ | `[0,1,1,1,1,2,2,3,3,4]` |
| 4 | 2 | 5 | 2 | 1 | Not equal | `nums[2]=2`, i++ | `[0,1,2,1,1,2,2,3,3,4]` |
| 5 | 3 | 6 | 2 | 2 | Equal | j++ | `[0,1,2,1,1,2,2,3,3,4]` |
| 6 | 3 | 7 | 3 | 2 | Not equal | `nums[3]=3`, i++ | `[0,1,2,3,1,2,2,3,3,4]` |
| 7 | 4 | 8 | 3 | 3 | Equal | j++ | `[0,1,2,3,1,2,2,3,3,4]` |
| 8 | 4 | 9 | 4 | 3 | Not equal | `nums[4]=4`, i++ | `[0,1,2,3,4,2,2,3,3,4]` |

Final result: `k = 5`, first 5 elements are `[0,1,2,3,4]`

### TypeScript Implementation

```typescript
function removeDuplicates(nums: number[]): number {
    // Edge case: array with 0 or 1 element
    if (nums.length <= 1) {
        return nums.length;
    }
    
    // Both pointers start at index 1
    let i = 1; // Position to place next unique element
    let j = 1; // Current position being scanned
    
    while (j < nums.length) {
        // If current element is different from previous, it's unique
        if (nums[j] !== nums[j - 1]) {
            nums[i] = nums[j]; // Place unique element at position i
            i++; // Move to next position for unique element
        }
        j++; // Always move j to scan next element
    }
    
    return i; // i represents the count of unique elements
}
```
{: file="solution.ts" }

### Complexity Analysis

- **Time Complexity**: O(n) - We traverse the array once with pointer `j`
- **Space Complexity**: O(1) - We only use two extra variables (`i` and `j`), no additional data structures

> This is the optimal solution! We achieve O(n) time and O(1) space complexity.
{: .prompt-tip }

---

## Alternative Solution: Using Set (Not Recommended)

While we could use a `Set` to track unique elements, this approach doesn't meet the **in-place** requirement and uses O(n) extra space:

```typescript
// ❌ Not recommended - uses O(n) extra space
function removeDuplicates(nums: number[]): number {
    const unique = new Set(nums);
    let i = 0;
    for (const num of unique) {
        nums[i++] = num;
    }
    return unique.size;
}
```
{: .nolineno }

**Why this is not ideal:**
- Uses O(n) extra space for the Set
- Doesn't fully utilize the fact that the array is already sorted
- May not be accepted by the judge if strict in-place requirements are enforced

---

## Edge Cases to Consider

```typescript
// Test cases
console.log(removeDuplicates([1, 1, 2]));                    // Output: 2
console.log(removeDuplicates([0, 0, 1, 1, 1, 2, 2, 3, 3, 4])); // Output: 5
console.log(removeDuplicates([1]));                           // Output: 1
console.log(removeDuplicates([1, 1, 1]));                     // Output: 1
console.log(removeDuplicates([1, 2, 3]));                     // Output: 3 (no duplicates)
console.log(removeDuplicates([-100, -100, 0, 0, 1, 1]));     // Output: 3
```
{: .nolineno }

> Always handle edge cases like empty arrays, single-element arrays, and arrays with all duplicates!
{: .prompt-warning }

---

## Key Takeaways

1. **Two-pointer technique** is perfect for in-place array modifications
2. Since the array is **sorted**, duplicates are always adjacent
3. Pointer `i` tracks where to place unique elements
4. Pointer `j` scans through the array to find unique elements
5. The condition `nums[j] !== nums[j - 1]` identifies unique elements
6. We achieve **optimal O(n) time and O(1) space** complexity

## Conclusion

The two-pointer approach is the most efficient solution for this problem. It leverages the sorted nature of the array to remove duplicates in a single pass while using only constant extra space.

This pattern is commonly used in array manipulation problems, especially when you need to modify arrays in-place. Mastering the two-pointer technique will help you solve many similar LeetCode problems!

Happy coding! 🚀

