---
layout: chapter
title: Two Pointers & Sliding Window - Q&A
course_id: coding-patterns
chapter_number: 2
---

**Quick Revision:** Master two pointers and sliding window patterns.

## Pattern Recognition

**Q1:** "Find pair in sorted array that sums to target" - which pattern?
**A:** Two pointers, opposite ends. O(n) time, O(1) space.

**Q2:** "Longest substring with at most K distinct characters" - pattern?
**A:** Sliding window, dynamic size + hash map to track frequency.

**Q3:** "Remove duplicates from sorted array in-place" - pattern?
**A:** Two pointers, fast/slow (same direction).

## Complexity

**Q4:** Why is sliding window O(n) when there are nested loops?
**A:** Each element added once (right pointer) and removed once (left pointer). Total: 2n operations = O(n).

**Q5:** 3Sum problem time complexity?
**A:** O(n²). Sort O(n log n), then O(n) outer loop × O(n) two pointers = O(n²).

## Implementation

**Q6:** Sliding window size K - how to calculate current size?
**A:** `right - left + 1` (not `right - left`).

**Q7:** Two pointers (opposite ends) - when to move left vs right?
**A:** If sum < target, move left (increase). If sum > target, move right (decrease).

## Common Mistakes

**Q8:** Why does two pointers fail on unsorted array for pair sum?
**A:** Opposite-end pointers rely on sorted order to decide direction. Unsorted breaks this logic.

**Q9:** Sliding window infinite loop - common cause?
**A:** Left pointer not advancing in contraction phase. Always ensure progress.

## Key Takeaways

- Two pointers: opposite ends (sorted pairs), fast/slow (in-place)
- Sliding window: fixed (add right, drop left), dynamic (expand/contract)
- Both optimize O(n²) to O(n)
- Window size: `right - left + 1`
- Sliding window is O(n) amortized (each element processed twice max)
- Master these - appear in 40%+ of array/string problems
