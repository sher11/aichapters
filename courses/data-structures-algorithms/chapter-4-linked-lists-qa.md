---
layout: chapter
title: Linked Lists - Q&A
course_id: data-structures-algorithms
chapter_number: 4
---

**Quick Revision:** Test your linked list mastery with these interview-focused questions.

## Complexity & Operations

**Q1:** Why is accessing the 100th element O(n) in a linked list but O(1) in an array?
**A:** Array: `address = base + 100 * element_size` (direct calculation). Linked list: must traverse 100 nodes via next pointers.

**Q2:** You have a node reference. Insertion time for singly vs doubly linked list?
**A:** Both O(1). Doubly: can insert before or after. Singly: only after (unless you have prev).

**Q3:** Delete a node given only its reference (not head). Possible?
**A:** Yes, if not tail: copy next node's value to current, delete next. O(1) time. (Trick: not true deletion, value swap)

**Q4:** Why is linked list insertion O(1) but array insertion O(n)?
**A:** Linked list: change pointers, no shifting. Array: must shift all elements after insertion point.

## Pattern Recognition

**Q5:** "Find the middle node" - what approach and why?
**A:** Fast/slow pointers. Fast moves 2x speed; when fast reaches end, slow is at middle. Single pass, O(1) space.

**Q6:** "Detect cycle in linked list" - how and what's the complexity?
**A:** Floyd's cycle detection (tortoise/hare). Slow +1, fast +2. If they meet, cycle exists. O(n) time, O(1) space.

**Q7:** "Remove nth node from end" without calculating length?
**A:** Runner technique: advance first pointer n steps, then move both until first reaches end. O(n) time, single pass.

**Q8:** "Palindrome linked list" - O(1) space approach?
**A:** Find middle (fast/slow), reverse second half, compare both halves. Time O(n), space O(1).

## Reversal Questions

**Q9:** Iterative reversal - what three pointers do you need?
**A:** prev, curr, next. Pattern: save next, reverse curr.next, advance all three.

**Q10:** Recursive reversal - why O(n) space?
**A:** Call stack depth is n (one frame per node). Each call waits for next to return before reversing.

**Q11:** Reverse only nodes m to n. How to handle?
**A:** Traverse to m-1, reverse m to n using standard reversal, reconnect at boundaries.

**Q12:** When is iterative reversal better than recursive?
**A:** Always for production: O(1) space vs O(n). Recursive risks stack overflow for long lists.

## Trade-offs & Design

**Q13:** Why don't databases use linked lists for primary storage?
**A:** Cache misses kill performance. Disk seeks to scattered nodes are expensive. Arrays/B-trees provide locality.

**Q14:** When would you use linked list over dynamic array?
**A:** Frequent insertions/deletions in middle (e.g., task scheduler, LRU cache). Array requires O(n) shifts.

**Q15:** Singly vs doubly linked - when to use each?
**A:**
- **Singly:** Memory constrained, only forward traversal needed (e.g., stack)
- **Doubly:** Need backward traversal, O(1) delete with node reference (e.g., LRU cache)

**Q16:** Skip list vs balanced tree - which and why?
**A:** Skip list for concurrent systems: easier lock-free implementation. Tree for single-threaded: better worst-case guarantees.

## Key Takeaways

- Two-pointer technique solves 70% of linked list problems
- Dummy head eliminates 90% of edge case complexity
- Iterative > recursive for production (space and stack overflow)
- Poor cache locality makes linked lists slower than arrays in practice
- Real-world usage: LRU cache, queues, undo systems, not general storage
- For L6/E6: discuss memory overhead, fragmentation, and concurrency
