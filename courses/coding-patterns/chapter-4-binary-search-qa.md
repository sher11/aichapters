---
layout: chapter
title: Binary Search - Q&A
course_id: coding-patterns
chapter_number: 4
---

**Quick Revision:** Test your binary search mastery.

## Core Concepts

**Q1:** Binary search time and space complexity?
**A:** Time: O(log n), Space: O(1) iterative, O(log n) recursive (call stack).

**Q2:** Why `mid = left + (right - left) // 2` instead of `(left + right) // 2`?
**A:** Avoids integer overflow when left + right > MAX_INT.

**Q3:** `left <= right` vs `left < right` - when each?
**A:** `<=` for finding exact element. `<` for finding insertion point/boundary.

## Pattern Recognition

**Q4:** "Find minimum in rotated sorted array" - approach?
**A:** Modified binary search. Compare mid with right to determine which half is sorted.

**Q5:** "Find peak element" - does it require fully sorted array?
**A:** No. Binary search works: if arr[mid] < arr[mid+1], peak is on right; else left.

**Q6:** "Capacity to ship packages in D days" - what pattern?
**A:** Binary search on answer space. Search capacity from max(weights) to sum(weights).

## Common Mistakes

**Q7:** Binary search infinite loop - common cause?
**A:** Left or right not advancing. E.g., `left = mid` instead of `left = mid + 1`.

**Q8:** Why does binary search fail on unsorted array?
**A:** Relies on sorted property to eliminate half. Without it, can't determine which half has answer.

## Key Insights

- Binary search: O(log n) reduces search space by half
- Requires sorted input (or rotated sorted with modification)
- Template: narrow [left, right] until found or left > right
- Answer space: binary search on possible values, use helper to check feasibility
- Avoid overflow: `left + (right - left) // 2`
- For L6/E6: Discuss boundary conditions, when NOT applicable (unsorted)
