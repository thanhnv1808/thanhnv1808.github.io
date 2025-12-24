---
title: "LeetCode 27: Remove Element - Two Pointers Technique"
author: thanhnv1808
date: 2025-11-25 09:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, array, two-pointers, in-place]
description: Learn how to remove all occurrences of a value from an array in-place using the two pointers technique, achieving O(n) time and O(1) space complexity.
comments: true
---

## Problem Statement

Given an integer array `nums` and an integer `val`, remove all occurrences of `val` in `nums` **in-place**. The order of the elements may be changed. Then return the number of elements in `nums` which are not equal to `val`.

Consider the number of elements in `nums` which are not equal to `val` be `k`, to get accepted, you need to do the following things:

- Change the array `nums` such that the first `k` elements of `nums` contain the elements which are not equal to `val`. The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

**Link**: [LeetCode 27 - Remove Element](https://leetcode.com/problems/remove-element/description/)

### Custom Judge

The judge will test your solution with the following code:

```plaintext
int[] nums = [...]; // Input array
int val = ...; // Value to remove
int[] expectedNums = [...]; // The expected answer with correct length.
                            // It is sorted with no values equaling val.

int k = removeElement(nums, val); // Calls your implementation

assert k == expectedNums.length;
sort(nums, 0, k); // Sort the first k elements of nums
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}
```
{: .nolineno }

If all assertions pass, then your solution will be accepted.

### Examples

**Example 1:**
```plaintext
Input: nums = [3,2,2,3], val = 3
Output: 2, nums = [2,2,_,_]
Explanation: Your function should return k = 2, with the first two elements of nums being 2.
It does not matter what you leave beyond the returned k (hence they are underscores).
```
{: .nolineno }

**Example 2:**
```plaintext
Input: nums = [0,1,2,2,3,0,4,2], val = 2
Output: 5, nums = [0,1,4,0,3,_,_,_]
Explanation: Your function should return k = 5, with the first five elements of nums containing 0, 0, 1, 3, and 4.
Note that the five elements can be returned in any order.
It does not matter what you leave beyond the returned k (hence they are underscores).
```
{: .nolineno }

### Constraints

- `0 <= nums.length <= 100`
- `0 <= nums[i] <= 50`
- `0 <= val <= 100`

---

## Solution 1: Create New Array (Not In-Place)

The simplest approach is to create a new array and copy only the elements that are not equal to `val`.

### Approach

1. Create a new array to store valid elements
2. Iterate through the original array
3. For each element, if it's not equal to `val`, add it to the new array
4. Copy the new array back to the original array (if required)
5. Return the length of the new array

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through the array once
- **Space Complexity**: O(n) - We create a new array to store valid elements

### TypeScript Implementation

```typescript
function removeElement(nums: number[], val: number): number {
    const result: number[] = [];
    
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] !== val) {
            result.push(nums[i]);
        }
    }
    
    // Copy back to original array
    for (let i = 0; i < result.length; i++) {
        nums[i] = result[i];
    }
    
    return result.length;
}
```

### Why This Works

- **Simple logic** - Easy to understand and implement
- **Preserves order** - Maintains the original order of valid elements
- **Straightforward** - No complex pointer manipulation

### Limitations

- **Not truly in-place** - Requires O(n) extra space
- **Two passes** - Needs to iterate twice (once to build, once to copy back)

> ⚠️ **Note**: While this approach works, it doesn't meet the in-place requirement efficiently. The problem allows changing the order, so we can optimize further.
{: .prompt-warning }

---

## Solution 2: Two Pointers (Optimal - In-Place)

This approach uses two pointers to swap elements in-place, achieving O(1) space complexity.

### Approach

1. Use two pointers: `i` starting from the left (0) and `j` starting from the right (length - 1)
2. While `i <= j`:
   - If `nums[i]` is not equal to `val`, increment `i` (keep this element)
   - If `nums[i]` equals `val`, swap it with `nums[j]` and decrement `j` (move the unwanted value to the end)
3. Return `i` (which represents the count of valid elements)

### Complexity Analysis

- **Time Complexity**: O(n) - In the worst case, we visit each element once
- **Space Complexity**: O(1) - We only use a constant amount of extra space for the pointers and temporary swap variable

### TypeScript Implementation

```typescript
function removeElement(nums: number[], val: number): number {
    let i = 0, j = nums.length - 1;

    while (i <= j) {
        if (nums[i] !== val) {
            i++;
        } else {
            // Swap nums[i] with nums[j]
            const temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
            
            j--;
        }
    }

    return i;
}
```

### Why This Works

- **In-place modification** - We modify the array without creating a new one
- **Single pass** - We process elements in one iteration
- **Efficient space usage** - Only O(1) extra space needed
- **Order flexibility** - The problem allows changing order, so swapping is acceptable

### Example Trace

For `nums = [3,2,2,3]`, `val = 3`:

```
Initial: i=0, j=3, nums = [3,2,2,3]

Step 1: i=0, nums[0]=3 == val
  Swap nums[0] with nums[3]
  nums = [3,2,2,3] → [3,2,2,3] (same after swap)
  j = 2
  nums = [3,2,2,3] (but j now points to index 2)

Step 2: i=0, nums[0]=3 == val
  Swap nums[0] with nums[2]
  nums = [2,2,3,3]
  j = 1

Step 3: i=0, nums[0]=2 ≠ val
  i = 1

Step 4: i=1, nums[1]=2 ≠ val
  i = 2

Now i=2, j=1, so i > j, loop ends
Return: 2
Final: nums = [2,2,3,3] (first 2 elements are valid)
```

For `nums = [0,1,2,2,3,0,4,2]`, `val = 2`:

```
Initial: i=0, j=7, nums = [0,1,2,2,3,0,4,2]

Step 1: i=0, nums[0]=0 ≠ val → i=1
Step 2: i=1, nums[1]=1 ≠ val → i=2
Step 3: i=2, nums[2]=2 == val
  Swap with nums[7]: [0,1,2,2,3,0,4,2] → [0,1,2,2,3,0,4,2]
  j=6
Step 4: i=2, nums[2]=2 == val
  Swap with nums[6]: [0,1,4,2,3,0,2,2]
  j=5
Step 5: i=2, nums[2]=4 ≠ val → i=3
Step 6: i=3, nums[3]=2 == val
  Swap with nums[5]: [0,1,4,0,3,2,2,2]
  j=4
Step 7: i=3, nums[3]=0 ≠ val → i=4
Step 8: i=4, nums[4]=3 ≠ val → i=5

Now i=5, j=4, so i > j, loop ends
Return: 5
Final: nums = [0,1,4,0,3,2,2,2] (first 5 elements are valid: 0,1,4,0,3)
```

---

## Solution 3: Two Pointers (Forward Only - Alternative)

An alternative approach that maintains relative order of valid elements by using a write pointer.

### Approach

1. Use a write pointer `k` starting at 0
2. Iterate through the array with a read pointer `i`
3. For each element:
   - If `nums[i] !== val`, copy it to `nums[k]` and increment `k`
   - If `nums[i] === val`, skip it (just increment `i`)
4. Return `k` (the number of valid elements)

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through the array once
- **Space Complexity**: O(1) - We only use a constant amount of extra space

### TypeScript Implementation

```typescript
function removeElement(nums: number[], val: number): number {
    let k = 0;
    
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] !== val) {
            nums[k] = nums[i];
            k++;
        }
    }
    
    return k;
}
```

### Why This Works

- **Preserves order** - Maintains the relative order of valid elements
- **Simple logic** - Easier to understand than the swap approach
- **In-place** - Modifies the array without extra space
- **Single pass** - Processes elements in one iteration

### Example Trace

For `nums = [3,2,2,3]`, `val = 3`:

```
Initial: k=0, nums = [3,2,2,3]

Step 1: i=0, nums[0]=3 == val → skip, k=0
Step 2: i=1, nums[1]=2 ≠ val
  nums[0] = 2, k=1
  nums = [2,2,2,3]
Step 3: i=2, nums[2]=2 ≠ val
  nums[1] = 2, k=2
  nums = [2,2,2,3]
Step 4: i=3, nums[3]=3 == val → skip, k=2

Return: 2
Final: nums = [2,2,2,3] (first 2 elements are valid)
```

> 💡 **Note**: This approach is simpler and preserves order, but Solution 2 (swap approach) is more efficient when the array has many elements to remove, as it doesn't need to shift elements.
{: .prompt-tip }

---

## Comparison of Solutions

| Solution | Time Complexity | Space Complexity | Preserves Order | Best For |
|----------|----------------|------------------|-----------------|----------|
| New Array | O(n) | O(n) | Yes | When order matters and space is not a concern |
| Two Pointers (Swap) | O(n) | O(1) | No | When order doesn't matter and we want minimal operations |
| Two Pointers (Forward) | O(n) | O(1) | Yes | When order matters and we want in-place solution |

> 💡 **Recommendation**: Use **Solution 3 (Forward Two Pointers)** for most cases as it's simple, preserves order, and is truly in-place. Use **Solution 2 (Swap Two Pointers)** when you have many elements to remove and order preservation is not required.
{: .prompt-tip }

---

## Key Takeaways

1. **Two pointers technique** - A powerful pattern for in-place array manipulation
2. **Order flexibility** - When the problem allows changing order, swapping can be more efficient
3. **Write pointer pattern** - Using a separate write pointer is cleaner when preserving order
4. **Space optimization** - We can achieve O(1) space by modifying the array in-place
5. **Single pass efficiency** - Both optimal solutions process the array in one iteration

---

## Related Problems

- [LeetCode 26: Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) - Remove duplicates in-place from sorted array
- [LeetCode 283: Move Zeroes](https://leetcode.com/problems/move-zeroes/) - Move all zeros to the end while maintaining relative order
- [LeetCode 80: Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) - Remove duplicates allowing at most two occurrences

---

## Practice

Try solving this problem on LeetCode: [27. Remove Element](https://leetcode.com/problems/remove-element/)

