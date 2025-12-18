---
layout: chapter
title: Linked Lists
course_id: data-structures-algorithms
chapter_number: 3
---

**One-liner:** Node-based structures with O(1) insertion/deletion but O(n) access, essential for pointer manipulation interviews.

## Types & Operations

| Operation | Singly | Doubly | Circular |
|-----------|--------|--------|----------|
| Access | O(n) | O(n) | O(n) |
| Search | O(n) | O(n) | O(n) |
| Insert at Head | O(1) | O(1) | O(1) |
| Insert at Tail | O(n) or O(1)* | O(1) | O(1) |
| Delete | O(n) | O(1)** | O(n) |

*With tail pointer | **With node reference

## Node Structures

```python
# Singly Linked
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Doubly Linked
class DNode:
    def __init__(self, val=0, prev=None, next=None):
        self.val = val
        self.prev = prev
        self.next = next
```

## Key Patterns for Interviews

### 1. Two Pointers (Fast & Slow)
- **When:** Cycle detection, find middle, nth from end
- **Floyd's Cycle:** Slow +1, fast +2 per iteration
- **Time:** O(n) | **Space:** O(1)

```python
# Find middle
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
return slow  # slow is at middle
```

### 2. Dummy Head
- **When:** Operations on head node, building result list
- **Why:** Avoids special-casing empty list or head changes

```python
dummy = ListNode(0)
dummy.next = head
# ... operate on dummy.next
return dummy.next
```

### 3. Reversal
- **Iterative:** O(n) time, O(1) space
- **Recursive:** O(n) time, O(n) space (call stack)

```python
# Iterative reversal
prev, curr = None, head
while curr:
    next_temp = curr.next
    curr.next = prev
    prev = curr
    curr = next_temp
return prev
```

### 4. Runner Technique
- **When:** k-distance problems, interleaving
- **Example:** Remove nth from end - runner k nodes ahead

## Common Interview Questions (High-Frequency)

1. **Reverse Linked List** - Iterative O(1) space vs recursive O(n)
2. **Detect Cycle** - Floyd's algorithm (fast/slow pointers)
3. **Merge Two Sorted Lists** - Two pointers, dummy head
4. **Remove Nth Node From End** - Two pointers k apart
5. **Intersection of Two Lists** - Align by length or cycle to other list
6. **Palindrome Linked List** - Find middle, reverse second half, compare
7. **Add Two Numbers** - Simulate digit addition with carry
8. **Copy List with Random Pointer** - Hash map or interweaving nodes

## Trade-offs: Linked List vs Array

| Factor | Linked List | Array |
|--------|-------------|-------|
| Access | O(n) ❌ | O(1) ✓ |
| Insert/Delete | O(1)* ✓ | O(n) ❌ |
| Memory | Extra pointers ❌ | Contiguous ✓ |
| Cache | Poor locality ❌ | Excellent ✓ |
| Size | Dynamic ✓ | Fixed/resize ❌ |

*Given node reference

## When to Use Linked Lists

- ✓ Frequent insertions/deletions (e.g., LRU cache)
- ✓ Unknown size, unbounded growth
- ✓ Implementing stacks, queues, deques
- ❌ Need random access (use array)
- ❌ Memory constrained (pointer overhead ~50% for small values)
- ❌ Performance critical (cache misses)

## Gotchas & Common Mistakes

- **Null pointer exceptions:** Always check `if node is None` before `node.next`
- **Losing references:** Save `node.next` before reassigning `node.next`
- **Cycle creation:** Be careful with circular references in reversal
- **Off-by-one:** "Remove nth from end" - clarify 1-indexed vs 0-indexed
- **Modifying during traversal:** Keep prev pointer when deleting
- **Memory leaks (C/C++):** Delete nodes explicitly, avoid dangling pointers

## Edge Cases

- Empty list: `head is None`
- Single node: `head.next is None`
- Two nodes: minimal non-trivial case
- Cycle exists: fast pointer catches slow
- All same values
- Odd vs even length (for middle finding)

## Real-World System Considerations (L6/E6)

### Memory Management
- **Java/Python:** GC handles cleanup, watch for memory leaks from retained references
- **C/C++:** Manual delete required, use smart pointers (shared_ptr/unique_ptr)
- **Overhead:** Each node costs extra memory for pointer(s) vs array

### Performance Implications
- **Cache misses:** Nodes scattered in heap → poor cache locality
- **Memory allocation:** Each node is separate allocation (slow compared to array)
- **Fragmentation:** Can cause memory fragmentation over time

### Production Use Cases
- **LRU Cache:** Doubly linked list + hash map for O(1) operations
- **Browser history:** Back/forward navigation (doubly linked)
- **Music playlist:** Next/previous with circular list
- **Undo/Redo:** Stack implemented with linked list
- **Task scheduler:** Job queue with priority insertions

## Advanced Patterns

### Skip List (L6/E6 Concept)
- Probabilistic data structure: linked list with multiple levels
- **Search:** O(log n) average (vs O(n) for regular list)
- **Use:** Alternative to balanced trees, easier to implement concurrently
- **Example:** Redis sorted sets use skip lists

### Lock-Free Linked Lists
- **Challenge:** Concurrent insert/delete with CAS (Compare-And-Swap)
- **ABA Problem:** Node A → B → A looks unchanged but isn't
- **Solution:** Versioning or hazard pointers

## Memory Tricks

**Mnemonic - "FROG":**
- **F**loyd's for cycles (fast/slow)
- **R**eversal with three pointers (prev/curr/next)
- **R**unner technique for k-distance
- **D**ummy head for edge cases

## Quick Self-Test

1. Why use slow/fast pointers instead of length calculation? *Single pass, O(1) space*
2. Reverse linked list space complexity? *Iterative: O(1), Recursive: O(n)*
3. Why dummy head? *Eliminates special case for head operations*
4. Detect cycle - what if no cycle? *Fast reaches None/null*
5. Linked list vs array for LRU cache? *Linked list: O(1) move to front*

## Key Takeaways

- ✓ Linked lists trade O(1) access for O(1) insert/delete (with reference)
- ✓ Two pointers (fast/slow) solve most cycle and middle-finding problems
- ✓ Dummy head simplifies code by eliminating head-change special cases
- ✓ Reversal is fundamental - know both iterative and recursive
- ✓ Poor cache locality makes arrays faster in practice for small-to-medium datasets
- ✓ For L6/E6: discuss when NOT to use (cache, memory overhead, allocation cost)
- ✓ Real-world usage: LRU cache, task queues, browser history, undo systems
