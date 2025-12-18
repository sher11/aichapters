---
layout: chapter
title: Binary Search & Divide and Conquer
course_id: coding-patterns
chapter_number: 3
---

**One-liner:** Reduce search space by half each iteration for O(log n) efficiency.

## Binary Search Pattern

### Classic Binary Search
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2  # Avoid overflow
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```
**Time:** O(log n) | **Space:** O(1)

### When to Use
- Sorted array/space
- Search for element or insertion point
- Minimize/maximize with constraint
- Keywords: "sorted", "find in O(log n)"

## Binary Search Variants

### 1. Find First/Last Occurrence
```python
def find_first(arr, target):
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            result = mid
            right = mid - 1  # Continue left
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return result
```

### 2. Search in Rotated Array
```python
def search_rotated(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            return mid
        
        # Determine which half is sorted
        if arr[left] <= arr[mid]:  # Left sorted
            if arr[left] <= target < arr[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # Right sorted
            if arr[mid] < target <= arr[right]:
                left = mid + 1
            else:
                right = mid - 1
    
    return -1
```

### 3. Search in 2D Matrix
```python
def search_matrix(matrix, target):
    if not matrix: return False
    m, n = len(matrix), len(matrix[0])
    left, right = 0, m * n - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        mid_val = matrix[mid // n][mid % n]
        
        if mid_val == target:
            return True
        elif mid_val < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return False
```

## High-Frequency Problems

1. **Binary Search** - Classic O(log n) search
2. **First Bad Version** - Find first true in boolean array
3. **Search in Rotated Sorted Array** - Modified binary search
4. **Find Peak Element** - Binary search on unsorted
5. **Search 2D Matrix** - Treat as 1D sorted array
6. **Sqrt(x)** - Binary search on answer space
7. **Koko Eating Bananas** - Binary search on answer
8. **Median of Two Sorted Arrays** - Partition with binary search

## Binary Search on Answer

When problem asks "minimize/maximize X":
1. Identify search space (min to max possible answer)
2. Binary search on answer
3. Use helper function to check if answer is feasible

```python
def min_capacity(weights, days):
    left, right = max(weights), sum(weights)
    
    while left < right:
        mid = left + (right - left) // 2
        if can_ship(weights, days, mid):
            right = mid  # Try smaller
        else:
            left = mid + 1  # Need larger
    
    return left

def can_ship(weights, days, capacity):
    current = day_count = 0
    for w in weights:
        if current + w > capacity:
            day_count += 1
            current = 0
        current += w
    return day_count + 1 <= days
```

## Divide and Conquer

### Merge Sort
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)
```
**Time:** O(n log n) | **Space:** O(n)

### Quick Sort
```python
def quick_sort(arr, low, high):
    if low < high:
        pivot = partition(arr, low, high)
        quick_sort(arr, low, pivot - 1)
        quick_sort(arr, pivot + 1, high)
```
**Time:** O(n log n) average, O(n²) worst | **Space:** O(log n)

### Quick Select (Kth Largest)
```python
def find_kth_largest(arr, k):
    k = len(arr) - k  # Convert to 0-indexed
    
    def select(left, right):
        pivot = arr[right]
        i = left
        for j in range(left, right):
            if arr[j] <= pivot:
                arr[i], arr[j] = arr[j], arr[i]
                i += 1
        arr[i], arr[right] = arr[right], arr[i]
        
        if i == k: return arr[i]
        elif i < k: return select(i + 1, right)
        else: return select(left, i - 1)
    
    return select(0, len(arr) - 1)
```
**Time:** O(n) average | **Space:** O(1)

## Gotchas & Mistakes

- **Overflow:** Use `mid = left + (right - left) // 2` not `(left + right) // 2`
- **Boundaries:** `left <= right` vs `left < right` affects result
- **Infinite loop:** Ensure left/right always progress
- **Not sorted:** Binary search requires sorted input (or rotated sorted)
- **Duplicates:** Handle first/last occurrence carefully

## Key Takeaways

- Binary search: O(log n) for sorted arrays
- Template: narrow [left, right] by half until found
- Variants: first/last, rotated, 2D, answer space
- Divide and conquer: split, solve, combine
- Merge sort: O(n log n) stable, Quick sort: O(n log n) average in-place
- For L6/E6: Discuss overflow, boundary conditions, when NOT to use
