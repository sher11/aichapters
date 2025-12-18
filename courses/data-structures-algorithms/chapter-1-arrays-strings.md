---
layout: chapter
title: Arrays & Strings
course_id: data-structures-algorithms
chapter_number: 1
---

**One-liner:** Contiguous memory structures with O(1) access, fundamental for many interview patterns.

## Core Operations

| Operation | Array | Dynamic Array | String |
|-----------|-------|---------------|--------|
| Access | O(1) | O(1) | O(1) |
| Search | O(n) | O(n) | O(n) |
| Insert End | N/A | O(1) amortized | N/A (immutable) |
| Insert Middle | N/A | O(n) | N/A (immutable) |
| Delete | N/A | O(n) | N/A (immutable) |

## Key Patterns for Interviews

### 1. Two Pointers
- **When:** Sorted array, palindrome, pair sum problems
- **Example:** Remove duplicates, container with most water
- **Time:** O(n) | **Space:** O(1)

### 2. Sliding Window
- **When:** Subarray/substring with constraint
- **Example:** Longest substring without repeating chars, max sum subarray
- **Time:** O(n) | **Space:** O(k) where k is window size

### 3. Prefix Sum
- **When:** Range queries, subarray sum problems
- **Example:** Subarray sum equals K, range sum query
- **Time:** O(n) preprocessing, O(1) query | **Space:** O(n)

### 4. Kadane's Algorithm
- **When:** Maximum subarray sum
- **Time:** O(n) | **Space:** O(1)
```python
max_sum = current_sum = arr[0]
for num in arr[1:]:
    current_sum = max(num, current_sum + num)
    max_sum = max(max_sum, current_sum)
```

## String-Specific Techniques

### Character Frequency
- Use hash map or array[26] for lowercase letters
- Common in anagram, permutation problems
- **Space:** O(1) for fixed alphabet, O(k) for k unique chars

### String Matching
- **Naive:** O(mn) where m=pattern, n=text
- **KMP:** O(m+n) with O(m) space for failure function
- **Rabin-Karp:** O(n) average, O(mn) worst with rolling hash

### StringBuilder Pattern
- Mutable string operations avoid O(n²) concatenation
- **Java:** StringBuilder | **Python:** list + join() | **C++:** ostringstream

## Common Interview Questions (High-Frequency)

1. **Two Sum** - Hash map for O(n)
2. **Best Time to Buy/Sell Stock** - Track min price
3. **Product of Array Except Self** - Prefix/suffix products
4. **Longest Substring Without Repeating Characters** - Sliding window + set
5. **Valid Anagram** - Character frequency
6. **Group Anagrams** - Sort or count as key
7. **Longest Palindromic Substring** - Expand around center O(n²)
8. **3Sum** - Sort + two pointers O(n²)

## Trade-offs & When to Use

| Scenario | Use Array | Use Dynamic Array |
|----------|-----------|-------------------|
| Fixed size known | ✓ Better memory | |
| Unknown size | | ✓ Flexibility |
| Frequent insertions | | ✓ (use ArrayList/vector) |
| Memory constrained | ✓ No overhead | ❌ Amortized resizing cost |

## Gotchas & Common Mistakes

- **Off-by-one errors:** Watch loop boundaries and index calculations
- **Integer overflow:** Sum of large numbers, use long or check bounds
- **Mutating while iterating:** Create copy or iterate backwards for deletions
- **String immutability:** In Java/Python, string concat is O(n²), use StringBuilder
- **Empty array:** Always check `arr.length == 0` before accessing
- **Negative indices:** Python allows, other languages don't

## Edge Cases to Consider

- Empty input: `[]` or `""`
- Single element: `[x]` or `"a"`
- All same elements: `[1,1,1,1]`
- Sorted vs unsorted
- Negative numbers
- Duplicates

## Real-World System Considerations (L6/E6)

- **Cache locality:** Arrays have better cache performance than linked structures
- **Memory allocation:** Dynamic arrays may cause memory spikes on resize
- **Concurrent access:** Arrays need external synchronization, consider CopyOnWriteArrayList
- **Large datasets:** Consider if entire array fits in memory, streaming approaches

## Memory Tricks

**Mnemonic - "SWAP":**
- **S**liding window for subarrays
- **W**indow + hash for substrings
- **A**rray[26] for character counts
- **P**refix sum for range queries

## Quick Self-Test

1. What's time complexity of inserting at index 0 in dynamic array? *O(n)*
2. Why use two pointers instead of nested loops? *Reduces O(n²) to O(n)*
3. When is prefix sum array useful? *Multiple range queries O(1) each*
4. String concatenation in loop - complexity? *O(n²) without StringBuilder*

## Key Takeaways

- ✓ Arrays provide O(1) access but fixed size; dynamic arrays trade memory for flexibility
- ✓ Two pointers and sliding window are most common optimization patterns
- ✓ Hash maps complement arrays for O(1) lookup (e.g., Two Sum)
- ✓ Strings are often immutable - use StringBuilder to avoid O(n²) operations
- ✓ For L6/E6: discuss cache locality, memory allocation, and concurrency implications
- ✓ Always clarify: sorted? duplicates? negative numbers? empty input?
