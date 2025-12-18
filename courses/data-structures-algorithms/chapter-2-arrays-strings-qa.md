---
layout: chapter
title: Arrays & Strings - Q&A
course_id: data-structures-algorithms
chapter_number: 2
---

**Quick Revision:** Test your understanding of arrays and strings with these focused questions.

## Complexity Questions

**Q1:** What is the time complexity of accessing the i-th element in an array?
**A:** O(1) - Direct memory offset calculation: `base_address + i * element_size`

**Q2:** Why is inserting at the beginning of a dynamic array O(n)?
**A:** Must shift all n elements one position right to make room.

**Q3:** String concatenation in a loop: `for s in arr: result += s`. Complexity?
**A:** O(n²) - Each concatenation creates new string, copying all previous characters.

**Q4:** With prefix sum array built, what's the time to answer range sum queries?
**A:** O(1) per query using `prefix[j] - prefix[i-1]`, O(n) to build initially.

## Pattern Recognition

**Q5:** Given "find two numbers that sum to target", what pattern and why?
**A:** Hash map pattern. Store complements as you iterate: O(n) time vs O(n²) brute force.

**Q6:** "Longest substring without repeating characters" - which pattern?
**A:** Sliding window + hash set. Expand window right, contract when duplicate found.

**Q7:** When should you use two pointers instead of two nested loops?
**A:** When array is sorted OR you're working from both ends toward center (e.g., palindrome, container with water).

**Q8:** "Maximum sum subarray" - what algorithm and complexity?
**A:** Kadane's algorithm. Track current_sum and max_sum. O(n) time, O(1) space.

## Trade-off Questions

**Q9:** Array vs Dynamic Array: when to use which?
**A:**
- **Array:** Fixed size known, memory constrained, no insertions needed
- **Dynamic Array:** Size unknown, need flexibility, can tolerate amortized resizing cost

**Q10:** Why is array access faster than linked list in practice, even though both are O(1)?
**A:** Cache locality. Arrays store elements contiguously; CPU cache prefetches nearby memory. Linked lists scatter nodes in memory causing cache misses.

**Q11:** String vs StringBuilder: when does it matter?
**A:** Matters for multiple modifications. 10+ concatenations or any loop: use StringBuilder. Single concat: string is fine.

## Problem-Solving Questions

**Q12:** Two Sum: Why use hash map instead of sorting + two pointers?
**A:** Hash map preserves original indices (often required) and is O(n). Sorting is O(n log n) and loses indices.

**Q13:** "Product of array except self" without division - how?
**A:** Two passes: prefix products left-to-right, then suffix products right-to-left. O(n) time, O(1) extra space.

**Q14:** Find duplicates in array of integers [1..n]. O(1) space without modifying array?
**A:** Trick question - impossible by pigeonhole principle. Either use O(n) space OR modify array (mark visited by negating values at indices).

**Q15:** Detect palindrome: why expand from center instead of two pointers from ends?
**A:** Both work for single palindrome check. Expand from center works better for "find longest palindrome" by trying all centers (n possibilities).

## Edge Cases & Gotchas

**Q16:** What edge cases should you always test with arrays?
**A:**
- Empty array `[]`
- Single element `[x]`
- All duplicates `[5,5,5]`
- Already sorted / reverse sorted
- Negative numbers (if applicable)
- Integer overflow on sums

**Q17:** Common off-by-one errors with arrays?
**A:**
- Loop: `for i in range(len(arr))` vs `range(len(arr)-1)`
- Slice: `arr[0:n]` includes n elements, not n+1
- Prefix sum: `prefix[i]` = sum of first i elements (or i+1 with dummy 0)

**Q18:** Why might `arr[-1]` cause issues in a multilingual codebase?
**A:** Python allows negative indexing (last element), but Java/C++ throw errors. Clarify language or avoid negative indices.

## System Design Context (L6/E6)

**Q19:** Why prefer arrays for high-performance systems?
**A:** CPU cache optimization. Sequential access patterns prefetch cache lines. Critical for low-latency systems (trading, gaming, databases).

**Q20:** When would you NOT use a dynamic array in production?
**A:**
- Real-time systems: resizing causes unpredictable latency spikes
- Memory-constrained environments: wasted capacity (often 2x growth)
- Solution: Preallocate fixed size or use deque/circular buffer

**Q21:** Concurrent access to arrays - what's the issue?
**A:** Arrays aren't thread-safe. Reads usually OK, but writes need synchronization. Options: CopyOnWriteArrayList (read-heavy), external locks, or immutable arrays.

## Algorithm Debugging

**Q22:** Your sliding window solution times out. What to check?
**A:**
1. Are you resetting window position correctly? (common bug: not moving left pointer)
2. Is inner loop necessary? (should be O(n), not O(n²))
3. Hash operations O(1)? (verify you're not using list.contains())

**Q23:** Two pointers not finding solution in sorted array. What went wrong?
**A:** Check if array is actually sorted. Verify pointer movement logic (when to move left vs right). Ensure you're not skipping the answer (>= vs >).

## Quick Recall Drills

**Q24:** Time to search unsorted array? *O(n)*

**Q25:** Space for Kadane's algorithm? *O(1)*

**Q26:** KMP string matching complexity? *O(m+n)*

**Q27:** Rabin-Karp best vs worst case? *O(n) average, O(mn) worst*

**Q28:** Why array[26] for lowercase letter count? *O(1) space for fixed alphabet*

**Q29:** Sliding window typical space complexity? *O(k) where k = window size*

**Q30:** Prefix sum query time after preprocessing? *O(1)*

## Key Insights to Remember

- 🎯 Array access is O(1), but insertion/deletion is O(n) due to shifting
- 🎯 Two pointers and sliding window reduce O(n²) to O(n) for many problems
- 🎯 Hash maps complement arrays: trade O(n) space for O(1) lookup
- 🎯 String immutability matters: use StringBuilder for repeated modifications
- 🎯 Cache locality makes arrays faster in practice than complexity suggests
- 🎯 For L6/E6: always discuss production implications (concurrency, memory, latency)

## Revision Checklist

Before moving on, ensure you can:
- [ ] Explain all O(n²) → O(n) optimizations (two pointers, sliding window, hash map)
- [ ] Code Kadane's algorithm from memory
- [ ] Identify which pattern fits a problem in <30 seconds
- [ ] Discuss array vs dynamic array trade-offs with system examples
- [ ] List 5+ edge cases without looking
- [ ] Explain cache locality impact on real systems
