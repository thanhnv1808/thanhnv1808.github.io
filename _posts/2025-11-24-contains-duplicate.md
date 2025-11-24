---
title: "LeetCode 217: Contains Duplicate - Multiple Efficient Solutions"
author: thanhnv1808
date: 2025-11-24 15:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, array, hash-set, sorting]
description: Learn multiple approaches to detect duplicates in an array, from hash set optimization to sorting techniques, with detailed complexity analysis.
comments: true
---

## Problem Statement

Given an integer array `nums`, return `true` if any value appears at least twice in the array, and return `false` if every element is distinct.

**Link**: [LeetCode 217 - Contains Duplicate](https://leetcode.com/problems/contains-duplicate/description/)

### Examples

**Example 1:**
```plaintext
Input: nums = [1,2,3,1]
Output: true
Explanation: The element 1 occurs at the indices 0 and 3.
```
{: .nolineno }

**Example 2:**
```plaintext
Input: nums = [1,2,3,4]
Output: false
Explanation: All elements are distinct.
```
{: .nolineno }

**Example 3:**
```plaintext
Input: nums = [1,1,1,3,3,4,3,2,4,2]
Output: true
```
{: .nolineno }

### Constraints

- `1 <= nums.length <= 10^5`
- `-10^9 <= nums[i] <= 10^9`

---

## Solution 1: Hash Set (Optimal)

The most efficient approach uses a hash set to track seen elements. As we iterate through the array, if we encounter an element that already exists in the set, we've found a duplicate.

### Approach

1. Create an empty hash set to store seen elements
2. Iterate through each element in the array
3. For each element:
   - If it exists in the set, return `true` (duplicate found)
   - Otherwise, add it to the set
4. If we finish the iteration without finding duplicates, return `false`

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through the array once, and hash set operations (insertion and lookup) are O(1) on average
- **Space Complexity**: O(n) - In the worst case, we store all n elements in the hash set

### TypeScript Implementation

```typescript
function containsDuplicate(nums: number[]): boolean {
    const seen = new Set<number>();
    
    for (const num of nums) {
        if (seen.has(num)) {
            return true;
        }
        seen.add(num);
    }
    
    return false;
}
```

### Alternative: Using Array Length Comparison

```typescript
function containsDuplicate(nums: number[]): boolean {
    const uniqueSet = new Set(nums);
    return uniqueSet.size !== nums.length;
}
```

This approach creates a set from the entire array and compares sizes. It's more concise but less efficient for early termination when duplicates are found early in the array.

### Why This Works

- **Hash sets provide O(1) average lookup time** - This makes checking for duplicates very fast
- **Early termination** - We can return `true` as soon as we find the first duplicate, without processing the rest of the array
- **Space-time trade-off** - We use O(n) extra space to achieve O(n) time complexity

### Example Trace

For `nums = [1,2,3,1]`:

```
Step 1: num = 1
  seen = {}
  seen.has(1)? No
  seen.add(1)
  seen = {1}

Step 2: num = 2
  seen = {1}
  seen.has(2)? No
  seen.add(2)
  seen = {1, 2}

Step 3: num = 3
  seen = {1, 2}
  seen.has(3)? No
  seen.add(3)
  seen = {1, 2, 3}

Step 4: num = 1
  seen = {1, 2, 3}
  seen.has(1)? Yes ✓ (duplicate found!)
  Return: true
```

---

## Solution 2: Sorting

This approach sorts the array first, then checks adjacent elements for duplicates.

### Approach

1. Sort the array in ascending order
2. Iterate through the array from index 0 to `n-2`
3. Compare each element with its next neighbor
4. If any adjacent pair is equal, return `true`
5. If no duplicates are found after checking all pairs, return `false`

### Complexity Analysis

- **Time Complexity**: O(n log n) - Sorting takes O(n log n) time, and the subsequent iteration is O(n)
- **Space Complexity**: O(1) - We only use a constant amount of extra space (assuming in-place sorting)

> ⚠️ **Note**: If the problem requires preserving the original array, we'd need O(n) extra space to create a copy before sorting.
{: .prompt-warning }

### TypeScript Implementation

```typescript
function containsDuplicate(nums: number[]): boolean {
    const n = nums.length;
    if (n === 0 || n === 1) return false;

    nums.sort((a, b) => a - b);
    
    for (let i = 0; i < n - 1; i++) {
        if (nums[i] === nums[i + 1]) {
            return true;
        }
    }
    
    return false;
}
```

### Why This Works

- **After sorting, duplicates are adjacent** - If there are any duplicates, they will be next to each other after sorting
- **Single pass comparison** - We only need to check adjacent elements, not all pairs
- **Space efficient** - If we can modify the input array, this uses O(1) extra space

### Example Trace

For `nums = [1,2,3,1]`:

```
Step 1: Sort the array
  Before: [1, 2, 3, 1]
  After:  [1, 1, 2, 3]

Step 2: Compare adjacent elements
  i=0: nums[0] = 1, nums[1] = 1
       1 === 1? Yes ✓ (duplicate found!)
  Return: true
```

---

## Solution 3: Brute Force (Nested Loops)

This is the most straightforward approach but also the least efficient. We compare every pair of elements.

### Approach

1. Use two nested loops
2. For each element at index `i`, compare it with all elements at indices `j > i`
3. If any pair is equal, return `true`
4. If no duplicates are found after checking all pairs, return `false`

### Complexity Analysis

- **Time Complexity**: O(n²) - We have nested loops, so we check n(n-1)/2 pairs in the worst case
- **Space Complexity**: O(1) - We only use a constant amount of extra space

### TypeScript Implementation

```typescript
function containsDuplicate(nums: number[]): boolean {
    const n = nums.length;
    
    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {
            if (nums[i] === nums[j]) {
                return true;
            }
        }
    }
    
    return false;
}
```

### Why This Works

- **Exhaustive comparison** - We check every possible pair of elements
- **No extra space** - We don't need any additional data structures
- **Simple logic** - Easy to understand and implement

### Example Trace

For `nums = [1,2,3,1]`:

```
i=0: Compare nums[0]=1 with nums[1]=2, nums[2]=3, nums[3]=1
  nums[0] === nums[3]? Yes ✓ (duplicate found!)
  Return: true
```

---

## Comparison of Solutions

| Solution | Time Complexity | Space Complexity | Best For |
|----------|----------------|------------------|----------|
| Hash Set | O(n) | O(n) | General use, optimal performance |
| Sorting | O(n log n) | O(1) | When space is limited and array can be modified |
| Brute Force | O(n²) | O(1) | Small arrays, educational purposes |

> 💡 **Recommendation**: Use the **Hash Set approach** for optimal performance. It's the fastest and most practical solution for this problem. The sorting approach is a good alternative when you need to minimize space usage and can modify the input array.
{: .prompt-tip }

---

## Key Takeaways

1. **Hash sets are powerful** - They provide O(1) average lookup time, making them ideal for duplicate detection
2. **Time vs Space trade-off** - Hash set uses O(n) space for O(n) time, while sorting uses O(1) space but O(n log n) time
3. **Early termination matters** - The hash set solution can return immediately when a duplicate is found
4. **Sorting changes the array** - Be careful if you need to preserve the original array order
5. **Brute force is simple but slow** - Only use nested loops for small arrays or when learning

---

## Related Problems

- [LeetCode 219: Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/) - Find duplicates within k distance
- [LeetCode 220: Contains Duplicate III](https://leetcode.com/problems/contains-duplicate-iii/) - Find duplicates within k distance and t value difference
- [LeetCode 287: Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) - Find the duplicate number in an array

---

## Practice

Try solving this problem on LeetCode: [217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)

