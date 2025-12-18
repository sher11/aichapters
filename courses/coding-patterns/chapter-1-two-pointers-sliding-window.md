---
layout: chapter
title: Two Pointers & Sliding Window
course_id: coding-patterns
chapter_number: 1
---

**One-liner:** Optimize O(n²) brute force to O(n) by maintaining pointers/window boundaries intelligently.

## Two Pointers Pattern

### When to Use
- Array/string traversal with pair relationships
- Sorted input (often)
- Need to optimize from O(n²) nested loops

### Variants

#### 1. Opposite Ends (Converging)
```python
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        curr_sum = arr[left] + arr[right]
        if curr_sum == target:
            return [left, right]
        elif curr_sum < target:
            left += 1
        else:
            right -= 1
    return None
```
**Time:** O(n) | **Space:** O(1)
**Use:** Pair sum, palindrome check, container with most water

#### 2. Same Direction (Fast & Slow)
```python
def remove_duplicates(arr):
    slow = 0
    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    return slow + 1
```
**Time:** O(n) | **Space:** O(1)
**Use:** Remove duplicates, partition, linked list cycle

## Sliding Window Pattern

### When to Use
- Subarray/substring problems
- Contiguous sequence with constraint
- Keywords: "longest", "shortest", "maximum", "minimum" substring/subarray

### Fixed Window Size
```python
def max_sum_subarray(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]  # Slide: add right, remove left
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```
**Time:** O(n) | **Space:** O(1)

### Dynamic Window Size
```python
def longest_substring_no_repeats(s):
    char_set = set()
    left = max_len = 0
    
    for right in range(len(s)):
        while s[right] in char_set:  # Contract window
            char_set.remove(s[left])
            left += 1
        char_set.add(s[right])  # Expand window
        max_len = max(max_len, right - left + 1)
    
    return max_len
```
**Time:** O(n) | **Space:** O(min(n, alphabet_size))

## Common Problems (High-Frequency)

### Two Pointers
1. **Two Sum II (sorted)** - Opposite ends, O(n)
2. **3Sum** - Sort + two pointers, O(n²)
3. **Container With Most Water** - Opposite ends, O(n)
4. **Valid Palindrome** - Opposite ends, O(n)
5. **Remove Duplicates** - Fast/slow, O(n)
6. **Trapping Rain Water** - Two pointers with max heights

### Sliding Window
1. **Maximum Sum Subarray of Size K** - Fixed window
2. **Longest Substring Without Repeating** - Dynamic + set
3. **Minimum Window Substring** - Dynamic + hash map
4. **Longest Substring with K Distinct Chars** - Dynamic + hash map
5. **Permutation in String** - Fixed window + frequency
6. **Sliding Window Maximum** - Deque (monotonic queue)

## Pattern Recognition

### Two Pointers Clues
- "Sorted array" + "find pair/triplet"
- "Remove duplicates/elements in-place"
- "Palindrome" check
- "Partition" array

### Sliding Window Clues
- "Longest/shortest substring/subarray"
- "Maximum/minimum of all subarrays of size K"
- "Substring with constraint" (distinct chars, anagram, etc.)

## Optimization: O(n²) → O(n)

### Before (Brute Force)
```python
# Check all subarrays - O(n²)
for i in range(n):
    for j in range(i, n):
        # Process subarray [i:j]
```

### After (Sliding Window)
```python
# Maintain window - O(n)
left = 0
for right in range(n):
    # Add arr[right] to window
    while condition_violated:
        # Remove arr[left] from window
        left += 1
    # Update result
```

## Trade-offs

| Factor | Two Pointers | Sliding Window |
|--------|--------------|----------------|
| Input | Often sorted | Any order |
| Window | No window concept | Fixed or dynamic window |
| Space | O(1) typical | O(k) for window state |
| Use | Pairs, partitions | Subarrays, substrings |

## Gotchas & Common Mistakes

- **Sorted assumption:** Two pointers (opposite ends) requires sorted input
- **Infinite loops:** Ensure pointers always progress in sliding window
- **Boundary:** Check `left < right` or `left <= right` carefully
- **Off-by-one:** Window size is `right - left + 1`, not `right - left`
- **State management:** Track window state (sum, freq) correctly when sliding

## Edge Cases

- Empty array/string
- Single element
- All same elements
- No valid window exists
- Entire array is the answer

## Complexity Analysis

### Two Pointers
- **Time:** O(n) single pass, or O(n²) for 3Sum variant
- **Space:** O(1) if in-place, O(n) if creating result

### Sliding Window
- **Time:** O(n) for right pointer, O(n) for left pointer = O(n) total
- **Space:** O(k) for window state (set/map)

## Advanced Variants

### Monotonic Deque (Sliding Window Maximum)
```python
from collections import deque

def max_sliding_window(arr, k):
    dq = deque()  # Store indices
    result = []
    
    for i in range(len(arr)):
        # Remove out-of-window
        if dq and dq[0] < i - k + 1:
            dq.popleft()
        
        # Maintain decreasing order
        while dq and arr[dq[-1]] < arr[i]:
            dq.pop()
        
        dq.append(i)
        
        if i >= k - 1:
            result.append(arr[dq[0]])
    
    return result
```
**Time:** O(n) | **Space:** O(k)

## Memory Tricks

**Mnemonic - "SLOW":**
- **S**orted → Opposite end pointers
- **L**ongest/Shortest substring → Sliding window
- **O**(1) space → Fast/slow pointers
- **W**indow size varies → Use hash map/set

## Quick Self-Test

1. Two Sum in sorted array - which pattern? *Two pointers (opposite ends)*
2. Longest substring K distinct chars? *Sliding window (dynamic)*
3. Remove duplicates in-place? *Two pointers (fast/slow)*
4. Max sum subarray size K? *Sliding window (fixed)*
5. 3Sum time complexity? *O(n²) with sort + two pointers*

## Key Takeaways

- Two pointers optimize pairs/triplets from O(n²) to O(n)
- Opposite ends for sorted arrays; fast/slow for in-place modifications
- Sliding window for subarray/substring with constraints
- Fixed window: add right, remove left; dynamic: expand/contract based on condition
- Both patterns typically O(n) time, O(1) or O(k) space
- For L6/E6: Discuss why O(n) works (amortized), edge cases, constraint handling
- Master these: appear in 40%+ of array/string interview problems
