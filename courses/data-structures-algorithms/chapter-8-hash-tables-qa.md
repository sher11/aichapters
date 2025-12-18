---
layout: chapter
title: Hash Tables & Sets - Q&A
course_id: data-structures-algorithms
chapter_number: 8
---

**Quick Revision:** Master hash tables for FAANG interviews.

## Core Concepts

**Q1:** Hash table average vs worst-case time complexity?
**A:** Average: O(1) for search/insert/delete. Worst: O(n) with all collisions.

**Q2:** What is load factor and typical value?
**A:** Load factor α = n/m (elements/buckets). Typical: 0.75. Too high → collisions, too low → wasted memory.

**Q3:** Why can't you hash mutable objects?
**A:** Hash must be deterministic. If object changes, hash changes, lookup fails.

## Problem Solving

**Q4:** "Find two numbers that sum to target" - O(n) solution?
**A:** Hash map. Store complements: `if target - num in seen`. O(n) time, O(n) space.

**Q5:** "Group Anagrams" - what should be the hash key?
**A:** Sorted string or character frequency tuple. Both give same hash for anagrams.

**Q6:** "Longest substring without repeating characters" - which structures?
**A:** Sliding window + hash set. Set tracks characters in current window.

## Trade-offs

**Q7:** Hash table vs BST - when to use which?
**A:** Hash: O(1) unordered, BST: O(log n) sorted order + range queries.

**Q8:** Hash table vs array for membership testing?
**A:** Hash: O(1) lookup, Array: O(n) search. Hash wins for large datasets.

## System Design (L6/E6)

**Q9:** What is consistent hashing and why use it?
**A:** Hash ring that minimizes rehashing when nodes added/removed. Use: load balancers, distributed caches.

**Q10:** How does Redis achieve O(1) operations?
**A:** In-memory hash table. No disk I/O for reads/writes.

**Q11:** LRU cache implementation - which data structures?
**A:** Hash map (O(1) lookup) + doubly linked list (O(1) move to front/evict tail).

## Key Takeaways

- Hash tables trade O(n) space for O(1) time
- Perfect for: Two Sum, frequency counting, caching, seen tracking
- Cannot hash mutable objects
- Load factor balances speed vs memory (0.75 typical)
- For L6/E6: consistent hashing, bloom filters, concurrent access
- Real-world: Redis, distributed caching, database hash indexes
