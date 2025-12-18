---
layout: chapter
title: Dynamic Programming - Q&A
course_id: coding-patterns
chapter_number: 6
---

**Quick Revision:** Master dynamic programming for FAANG interviews.

## Core Concepts

**Q1:** What are the two requirements for DP?
**A:** (1) Optimal substructure - optimal solution contains optimal subproblems. (2) Overlapping subproblems - same problems solved repeatedly.

**Q2:** Memoization vs tabulation?
**A:** Memoization: top-down recursion with cache. Tabulation: bottom-up iteration filling table.

**Q3:** Why tabulation over memoization?
**A:** No stack overflow, easier space optimization, better cache locality. Memoization more intuitive.

## Pattern Recognition

**Q4:** "Minimum cost to climb stairs" - which DP pattern?
**A:** 1D linear DP. dp[i] = min(dp[i-1], dp[i-2]) + cost[i].

**Q5:** "Longest common subsequence" - pattern and complexity?
**A:** 2D DP grid. O(mn) time, space-optimized to O(min(m,n)).

**Q6:** "Coin change - minimum coins" - which pattern?
**A:** Unbounded knapsack. For each coin, try taking unlimited times.

## Optimization

**Q7:** Fibonacci with O(1) space - how?
**A:** Only need last 2 values: prev, curr = curr, prev + curr.

**Q8:** 2D DP to 1D space - when possible?
**A:** When current row only depends on previous row. Use rolling array or single array.

**Q9:** Longest Increasing Subsequence - O(n log n) approach?
**A:** Binary search + patience sorting. Maintain tails array and binary search for position.

## Common Mistakes

**Q10:** Why does greedy fail for coin change but DP works?
**A:** Greedy may not find optimal (e.g., coins=[1,3,4], amount=6: greedy gives 4+1+1=3, optimal is 3+3=2).

**Q11:** DP array initialization - common mistake?
**A:** Not setting base cases correctly. E.g., dp[0] = 0 for sum problems, dp[0] = 1 for count problems.

## Key Insights

- DP = optimal substructure + overlapping subproblems
- Common patterns: 1D linear, 2D grid, knapsack, LCS
- Memoization (recursive) vs tabulation (iterative)
- Space optimize: often O(n) or O(1) instead of O(n²)
- Define state clearly: what does dp[i] represent?
- For L6/E6: Discuss optimization, state definition, when NOT to use DP
