---
title: "LeetCode 1480: Running Sum of 1d Array - Simple In-Place Solution"
author: thanhnv1808
date: 2025-11-24 13:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, typescript, array]
description: Learn how to compute the running sum of an array efficiently using an in-place approach with O(n) time and O(1) space complexity.
comments: true
---

## Problem Statement

Given an array `nums`. We define a running sum of an array as `runningSum[i] = sum(nums[0]…nums[i])`.

Return the running sum of `nums`.

**Link**: [LeetCode 1480 - Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/description/)

### Examples

**Example 1:**
```plaintext
Input: nums = [1,2,3,4]
Output: [1,3,6,10]
Explanation: Running sum is obtained as follows: [1, 1+2, 1+2+3, 1+2+3+4].
```
{: .nolineno }

**Example 2:**
```plaintext
Input: nums = [1,1,1,1,1]
Output: [1,2,3,4,5]
Explanation: Running sum is obtained as follows: [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1].
```
{: .nolineno }

**Example 3:**
```plaintext
Input: nums = [3,1,2,10,1]
Output: [3,4,6,16,17]
```
{: .nolineno }

### Constraints

- `1 <= nums.length <= 1000`
- `-10^6 <= nums[i] <= 10^6`

---

## Solution: In-Place Modification

The key insight is that we can compute the running sum in-place by adding each element to the sum of all previous elements. Since we're processing the array from left to right, we can simply add the current element to the previous running sum.

### Approach

1. Start from index `1` (since `nums[0]` is already the first running sum)
2. For each element at index `i`, add the value at `nums[i-1]` (which contains the running sum up to `i-1`) to `nums[i]`
3. This modifies `nums[i]` to contain the running sum up to index `i`
4. Continue until we've processed all elements

### Visual Walkthrough

For `nums = [1,2,3,4]`:

| Step | i | nums[i] | nums[i-1] | Operation | nums after |
|------|---|---------|-----------|-----------|------------|
| Initial | - | - | - | - | `[1,2,3,4]` |
| 1 | 1 | 2 | 1 | `nums[1] = 2 + 1` | `[1,3,3,4]` |
| 2 | 2 | 3 | 3 | `nums[2] = 3 + 3` | `[1,3,6,4]` |
| 3 | 3 | 4 | 6 | `nums[3] = 4 + 6` | `[1,3,6,10]` |

### TypeScript Implementation

```typescript
function runningSum(nums: number[]): number[] {
    let i = 1;

    while (i < nums.length) {
        nums[i] = nums[i] + nums[i - 1];
        i++;
    }

    return nums;
}
```

### Alternative: Using For Loop

```typescript
function runningSum(nums: number[]): number[] {
    for (let i = 1; i < nums.length; i++) {
        nums[i] += nums[i - 1];
    }
    return nums;
}
```

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through the array once, starting from index 1
- **Space Complexity**: O(1) - We modify the array in-place without using any extra space (excluding the input array itself)

> 💡 **Note**: The problem allows us to modify the input array in-place. If we needed to preserve the original array, we would need O(n) extra space to create a new array.
{: .prompt-info }

### Why This Works

The algorithm works because:

1. **Base case**: `nums[0]` is already the running sum for index 0 (sum of elements from index 0 to 0)
2. **Inductive step**: At each index `i > 0`, `nums[i-1]` already contains the running sum up to index `i-1`. By adding `nums[i]` (the original value) to `nums[i-1]` (the running sum), we get the running sum up to index `i`
3. **In-place modification**: Since we process from left to right, we never overwrite values we haven't processed yet

### Example Trace

Let's trace through Example 3: `nums = [3,1,2,10,1]`

```
Initial: [3, 1, 2, 10, 1]
         ↑
         i=0 (already correct, runningSum[0] = 3)

Step 1:  [3, 1, 2, 10, 1]
            ↑
            i=1: nums[1] = 1 + 3 = 4
         Result: [3, 4, 2, 10, 1]

Step 2:  [3, 4, 2, 10, 1]
               ↑
               i=2: nums[2] = 2 + 4 = 6
         Result: [3, 4, 6, 10, 1]

Step 3:  [3, 4, 6, 10, 1]
                  ↑
                  i=3: nums[3] = 10 + 6 = 16
         Result: [3, 4, 6, 16, 1]

Step 4:  [3, 4, 6, 16, 1]
                     ↑
                     i=4: nums[4] = 1 + 16 = 17
         Result: [3, 4, 6, 16, 17] ✓
```

---

## Key Takeaways

1. **In-place modification** is efficient when the problem allows it - we save O(n) space
2. **Processing order matters** - we must process from left to right to ensure previous running sums are available
3. **Simple problems can teach important patterns** - this problem demonstrates the prefix sum pattern, which is useful in many array problems
4. **Time vs Space trade-off** - if we needed to preserve the original array, we'd need O(n) extra space

---

## Related Problems

- [LeetCode 303: Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)
- [LeetCode 304: Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)
- [LeetCode 560: Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

---

## Practice

Try solving this problem on LeetCode: [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/)

