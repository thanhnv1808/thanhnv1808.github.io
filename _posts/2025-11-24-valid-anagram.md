---
title: "LeetCode 242: Valid Anagram - Three Efficient Solutions"
author: thanhnv1808
date: 2025-11-24 17:00:00 +0700
categories: [Algorithms, LeetCode]
tags: [leetcode, algorithms, string, hash-map, sorting]
description: Learn multiple approaches to check if two strings are anagrams, from sorting comparison to optimized character counting techniques.
comments: true
---

## Problem Statement

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise.

An **Anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

**Link**: [LeetCode 242 - Valid Anagram](https://leetcode.com/problems/valid-anagram/description/)

### Examples

**Example 1:**
```plaintext
Input: s = "anagram", t = "nagaram"
Output: true
```
{: .nolineno }

**Example 2:**
```plaintext
Input: s = "rat", t = "car"
Output: false
```
{: .nolineno }

### Constraints

- `1 <= s.length, t.length <= 5 * 10^4`
- `s` and `t` consist of lowercase English letters.

---

## Solution 1: Sorting

The simplest approach is to sort both strings and compare them. If they are anagrams, their sorted versions will be identical.

### Approach

1. Convert both strings to character arrays
2. Sort both arrays
3. Convert back to strings and compare
4. Return `true` if they match, `false` otherwise

### Complexity Analysis

- **Time Complexity**: O(n log n) - Sorting takes O(n log n) time for each string, where n is the length of the strings
- **Space Complexity**: O(n) - We need to store the character arrays for both strings

### TypeScript Implementation

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) {
        return false;
    }
    
    const sChars = s.split("").sort();
    const tChars = t.split("").sort();
    
    return sChars.join("") === tChars.join("");
}
```

### Why This Works

- **Anagrams have the same characters** - When sorted, anagrams produce identical sequences
- **Simple and intuitive** - Easy to understand and implement
- **Works for any character set** - Not limited to lowercase letters

### Example Trace

For `s = "anagram"`, `t = "nagaram"`:

```
Step 1: Convert to arrays
  sChars = ['a', 'n', 'a', 'g', 'r', 'a', 'm']
  tChars = ['n', 'a', 'g', 'a', 'r', 'a', 'm']

Step 2: Sort both arrays
  sChars = ['a', 'a', 'a', 'g', 'm', 'n', 'r']
  tChars = ['a', 'a', 'a', 'g', 'm', 'n', 'r']

Step 3: Compare
  sChars.join("") = "aaagmnr"
  tChars.join("") = "aaagmnr"
  "aaagmnr" === "aaagmnr"? Yes ✓
  
Return: true
```

---

## Solution 2: Character Counting with Map

This approach uses a hash map to count the frequency of each character in both strings, then compares the counts.

### Approach

1. If the strings have different lengths, return `false` immediately
2. Create a map to count characters in the first string
3. Decrement counts for characters in the second string
4. Check if all counts are zero
5. If any count is non-zero, return `false`; otherwise return `true`

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through both strings once
- **Space Complexity**: O(k) where k is the number of unique characters (at most 26 for lowercase English letters)

### TypeScript Implementation

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) {
        return false;
    }
    
    const charCount = new Map<string, number>();
    
    // Count characters in s
    for (const char of s) {
        charCount.set(char, (charCount.get(char) || 0) + 1);
    }
    
    // Decrement counts for characters in t
    for (const char of t) {
        const count = charCount.get(char) || 0;
        if (count === 0) {
            return false; // Character not in s or already used up
        }
        charCount.set(char, count - 1);
    }
    
    // Verify all counts are zero
    for (const count of charCount.values()) {
        if (count !== 0) {
            return false;
        }
    }
    
    return true;
}
```

### Alternative: Two Maps Approach

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) {
        return false;
    }
    
    const sCount = new Map<string, number>();
    const tCount = new Map<string, number>();
    
    for (let i = 0; i < s.length; i++) {
        sCount.set(s[i], (sCount.get(s[i]) || 0) + 1);
        tCount.set(t[i], (tCount.get(t[i]) || 0) + 1);
    }
    
    if (sCount.size !== tCount.size) {
        return false;
    }
    
    for (const [char, count] of sCount) {
        if (tCount.get(char) !== count) {
            return false;
        }
    }
    
    return true;
}
```

### Why This Works

- **Character frequency must match** - Anagrams have the same count for each character
- **Early termination** - We can return `false` as soon as we find a mismatch
- **Space efficient** - Only stores unique characters, not all characters

### Example Trace

For `s = "anagram"`, `t = "nagaram"`:

```
Step 1: Count characters in s
  charCount = {'a': 3, 'n': 1, 'g': 1, 'r': 1, 'm': 1}

Step 2: Process characters in t
  'n': count = 1, decrement → {'a': 3, 'n': 0, 'g': 1, 'r': 1, 'm': 1}
  'a': count = 3, decrement → {'a': 2, 'n': 0, 'g': 1, 'r': 1, 'm': 1}
  'g': count = 1, decrement → {'a': 2, 'n': 0, 'g': 0, 'r': 1, 'm': 1}
  'a': count = 2, decrement → {'a': 1, 'n': 0, 'g': 0, 'r': 1, 'm': 1}
  'r': count = 1, decrement → {'a': 1, 'n': 0, 'g': 0, 'r': 0, 'm': 1}
  'a': count = 1, decrement → {'a': 0, 'n': 0, 'g': 0, 'r': 0, 'm': 1}
  'm': count = 1, decrement → {'a': 0, 'n': 0, 'g': 0, 'r': 0, 'm': 0}

Step 3: Check all counts are zero
  All counts are 0 ✓
  
Return: true
```

---

## Solution 3: Optimized Character Counting with Array (Optimal)

Since the problem states that strings consist of lowercase English letters only, we can use a fixed-size array instead of a map for better performance.

### Approach

1. If the strings have different lengths, return `false` immediately
2. Create an array of size 26 (for 'a' to 'z')
3. Count characters in the first string (increment)
4. Count characters in the second string (decrement)
5. Check if all array values are zero
6. If any value is non-zero, return `false`; otherwise return `true`

### Complexity Analysis

- **Time Complexity**: O(n) - We iterate through both strings once
- **Space Complexity**: O(1) - We use a fixed-size array of 26 elements, regardless of input size

### TypeScript Implementation

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) {
        return false;
    }
    
    const charCount = new Array(26).fill(0);
    
    // Count characters in s (increment)
    for (const char of s) {
        const index = char.charCodeAt(0) - 'a'.charCodeAt(0);
        charCount[index]++;
    }
    
    // Count characters in t (decrement)
    for (const char of t) {
        const index = char.charCodeAt(0) - 'a'.charCodeAt(0);
        charCount[index]--;
        
        // Early termination: if count becomes negative, t has more of this char
        if (charCount[index] < 0) {
            return false;
        }
    }
    
    return true;
}
```

### Alternative: Single Pass with Early Check

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) {
        return false;
    }
    
    const charCount = new Array(26).fill(0);
    
    for (let i = 0; i < s.length; i++) {
        charCount[s.charCodeAt(i) - 97]++; // 'a' = 97
        charCount[t.charCodeAt(i) - 97]--;
    }
    
    return charCount.every(count => count === 0);
}
```

### Why This Works

- **Fixed space** - Array of 26 elements is constant space O(1)
- **Direct indexing** - No hash computation needed, just simple arithmetic
- **Faster than Map** - Array access is faster than hash map operations
- **Early termination** - Can detect mismatches during the decrement phase

### Example Trace

For `s = "anagram"`, `t = "nagaram"`:

```
Step 1: Initialize array
  charCount = [0, 0, 0, ..., 0] (26 zeros)

Step 2: Count characters in s
  'a' (index 0): charCount[0] = 1
  'n' (index 13): charCount[13] = 1
  'a' (index 0): charCount[0] = 2
  'g' (index 6): charCount[6] = 1
  'r' (index 17): charCount[17] = 1
  'a' (index 0): charCount[0] = 3
  'm' (index 12): charCount[12] = 1
  Result: [3, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1, ...]

Step 3: Decrement for characters in t
  'n' (index 13): charCount[13] = 0
  'a' (index 0): charCount[0] = 2
  'g' (index 6): charCount[6] = 0
  'a' (index 0): charCount[0] = 1
  'r' (index 17): charCount[17] = 0
  'a' (index 0): charCount[0] = 0
  'm' (index 12): charCount[12] = 0
  Result: All zeros ✓
  
Return: true
```

---

## Comparison of Solutions

| Solution | Time Complexity | Space Complexity | Best For |
|----------|----------------|------------------|----------|
| Sorting | O(n log n) | O(n) | Simple implementation, any character set |
| Map Counting | O(n) | O(k) where k ≤ 26 | General purpose, works with any characters |
| Array Counting | O(n) | O(1) | Optimal for lowercase English letters only |

> 💡 **Recommendation**: Use the **Array Counting approach** for optimal performance when dealing with lowercase English letters. It provides O(1) space complexity and is faster than using a Map. Use the Map approach if you need to handle any character set.
{: .prompt-tip }

---

## Key Takeaways

1. **Anagrams have identical character frequencies** - The core insight is that anagrams contain the same characters with the same counts
2. **Sorting is simple but slower** - O(n log n) time complexity makes it less efficient for large inputs
3. **Character counting is optimal** - O(n) time with O(1) space when using a fixed-size array
4. **Early termination matters** - Check length mismatch and negative counts early to avoid unnecessary processing
5. **Choose the right data structure** - Arrays are faster than Maps when you have a fixed, small character set
6. **Edge cases** - Always check if strings have the same length first

---

## Related Problems

- [LeetCode 49: Group Anagrams](https://leetcode.com/problems/group-anagrams/) - Group strings that are anagrams of each other
- [LeetCode 438: Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) - Find all anagram substrings
- [LeetCode 383: Ransom Note](https://leetcode.com/problems/ransom-note/) - Similar character counting problem

---

## Practice

Try solving this problem on LeetCode: [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/)

