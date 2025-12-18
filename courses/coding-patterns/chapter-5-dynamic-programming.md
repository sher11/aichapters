---
layout: chapter
title: Dynamic Programming
course_id: coding-patterns
chapter_number: 5
---

**One-liner:** Solve problems by breaking into overlapping subproblems and storing results to avoid recomputation.

## When to Use Dynamic Programming

1. **Optimal substructure:** Optimal solution contains optimal solutions to subproblems
2. **Overlapping subproblems:** Same subproblems solved repeatedly
3. **Keywords:** "minimum/maximum", "count ways", "longest/shortest"

## DP Approaches

### 1. Memoization (Top-Down)
```python
def fib(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```
**Pros:** Intuitive recursion | **Cons:** Stack overflow risk

### 2. Tabulation (Bottom-Up)
```python
def fib(n):
    if n <= 1: return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```
**Pros:** No recursion, better space optimization | **Cons:** Less intuitive

### 3. Space-Optimized
```python
def fib(n):
    if n <= 1: return n
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    return curr
```
**Benefit:** O(1) space when only need previous states

## Common DP Patterns

### 1. 1D DP (Linear)
**Problem:** Climbing stairs, house robber, decode ways

```python
def climb_stairs(n):
    if n <= 2: return n
    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

### 2. 2D DP (Grid)
**Problem:** Unique paths, longest common subsequence, edit distance

```python
def unique_paths(m, n):
    dp = [[1] * n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]
```

### 3. Knapsack Pattern
**Problem:** 0/1 knapsack, subset sum, partition equal subset

```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(
                    dp[i-1][w],  # Don't take
                    dp[i-1][w - weights[i-1]] + values[i-1]  # Take
                )
            else:
                dp[i][w] = dp[i-1][w]
    
    return dp[n][capacity]
```

### 4. LCS/Edit Distance Pattern
```python
def longest_common_subsequence(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]
```

## High-Frequency Problems

1. **Climbing Stairs** - Fibonacci variant
2. **House Robber** - Max non-adjacent sum
3. **Coin Change** - Unbounded knapsack
4. **Longest Increasing Subsequence** - O(n log n) with binary search
5. **Longest Common Subsequence** - 2D DP
6. **Edit Distance** - 2D DP (insert/delete/replace)
7. **Unique Paths** - 2D grid DP
8. **Word Break** - 1D DP with substring check
9. **Maximum Subarray** - Kadane's algorithm
10. **Best Time to Buy/Sell Stock** - State machine DP

## Complexity Analysis

| Pattern | Time | Space | Space-Optimized |
|---------|------|-------|-----------------|
| 1D Linear | O(n) | O(n) | O(1) |
| 2D Grid | O(mn) | O(mn) | O(min(m,n)) |
| Knapsack | O(nW) | O(nW) | O(W) |
| LCS | O(mn) | O(mn) | O(min(m,n)) |

## Optimization Techniques

### Space Optimization
- Often only need previous row/column: O(n) space
- Rolling array technique
- Single variable for simple cases (Fibonacci)

### Time Optimization
- Monotonic stack/deque for certain problems
- Binary search within DP (LIS: O(n log n))

## Gotchas & Mistakes

- **Base cases:** Initialize dp[0], dp[1] correctly
- **Index confusion:** dp[i] often represents i+1 elements (handle 0-indexing)
- **Loop direction:** Some problems need reverse iteration
- **State definition:** Clearly define what dp[i] represents
- **Space optimization:** Only works when current state depends on limited previous states

## Key Takeaways

- DP = Recursion + Memoization (or bottom-up tabulation)
- Identify: optimal substructure + overlapping subproblems
- Common patterns: 1D linear, 2D grid, knapsack, LCS/edit distance
- Memoization (top-down) vs tabulation (bottom-up)
- Space optimize: often O(n) or O(1) instead of O(n²)
- For L6/E6: Discuss state definition, optimization trade-offs, when NOT DP
- Master Kadane's, knapsack, and LCS - foundational patterns
