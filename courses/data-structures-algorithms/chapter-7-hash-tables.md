---
layout: chapter
title: Hash Tables & Sets
course_id: data-structures-algorithms
chapter_number: 7
---

**One-liner:** O(1) average lookup/insert/delete via hashing, critical for optimization and caching patterns.

## Core Operations

| Operation | Average | Worst | Notes |
|-----------|---------|-------|-------|
| Search | O(1) | O(n) | With good hash function |
| Insert | O(1) | O(n) | May trigger resize |
| Delete | O(1) | O(n) | Mark deleted or rehash |
| Space | O(n) | O(n) | Load factor affects memory |

## Hash Function Requirements

1. **Deterministic:** Same input → same hash
2. **Uniform distribution:** Minimize collisions
3. **Fast computation:** O(1) or O(k) for string length k
4. **Avalanche effect:** Small input change → large hash change

## Collision Resolution

### Chaining (Most Common)
- Each bucket is linked list (or tree in Java 8+)
- **Load factor α = n/m** where n=elements, m=buckets
- **Search:** O(1 + α) average

### Open Addressing
- **Linear probing:** Check next slot: (h + i) mod m
- **Quadratic probing:** (h + i²) mod m
- **Double hashing:** (h1 + i * h2) mod m
- **Issue:** Clustering degrades performance

## Common Interview Patterns

### 1. Two Sum Pattern
```python
def twoSum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
```
**Time:** O(n) | **Space:** O(n)

### 2. Frequency Counting
```python
from collections import Counter
freq = Counter(arr)  # Or manual dict
```
**Use:** Anagrams, top K elements, duplicates

### 3. Caching/Memoization
```python
cache = {}
def fib(n):
    if n in cache: return cache[n]
    if n <= 1: return n
    cache[n] = fib(n-1) + fib(n-2)
    return cache[n]
```
**Reduces:** Exponential to O(n) for DP problems

### 4. Seen/Visited Tracking
```python
visited = set()
for node in graph:
    if node not in visited:
        dfs(node, visited)
```
**Time:** O(1) per check vs O(n) for list

## High-Frequency Interview Problems

1. **Two Sum** - Hash map for complement
2. **Group Anagrams** - Sorted string or char count as key
3. **Longest Substring Without Repeating** - Sliding window + set
4. **Valid Anagram** - Frequency count comparison
5. **Top K Frequent Elements** - Counter + heap
6. **Subarray Sum Equals K** - Prefix sum + hash map
7. **LRU Cache** - Hash map + doubly linked list
8. **Design HashMap** - Array + chaining/probing

## Hash Set vs Hash Map

| Feature | Set | Map |
|---------|-----|-----|
| Stores | Keys only | Key-value pairs |
| Duplicates | No | Keys unique, values can repeat |
| Use | Membership testing | Lookup/count/cache |
| Example | visited set | word → count |

## Trade-offs

### Hash Table vs Array
- **Hash:** O(1) lookup, unordered, more memory
- **Array:** O(n) search, ordered, cache-friendly

### Hash Table vs Tree (BST/TreeMap)
- **Hash:** O(1) avg, no order, O(n) worst
- **Tree:** O(log n) guaranteed, sorted order, range queries

### When to Use Hash Table
- ✓ Need O(1) lookup (Two Sum, caching)
- ✓ Frequency counting, duplicates
- ✓ Seen/visited tracking
- ❌ Need sorted order (use TreeMap)
- ❌ Memory constrained (overhead ~2x)
- ❌ Small datasets (array faster due to cache)

## Gotchas & Common Mistakes

- **Hash of mutable objects:** Don't hash lists/dicts (use tuples in Python)
- **Load factor:** High → more collisions, low → wasted memory
- **Integer overflow:** Hash codes can overflow, use modulo
- **Equality:** Hash collision ≠ equality, must check equals()
- **Resize cost:** O(n) when load factor exceeds threshold (amortized O(1))
- **Determinism:** Same hash for equal objects (implement hashCode + equals)

## Edge Cases

- Empty map/set
- Single element
- All elements same hash (worst case O(n))
- All elements unique
- Load factor triggers resize mid-operation

## Real-World System Considerations (L6/E6)

### Database Indexing
- **Hash index:** O(1) equality lookups, no range queries
- **B-tree index:** O(log n) lookups, supports range
- **When hash:** Point queries on unique keys (user_id → data)

### Caching Systems
- **Redis/Memcached:** Hash table in memory for O(1) access
- **Eviction:** LRU (hash map + linked list), LFU, TTL
- **Sharding:** Consistent hashing to distribute keys across servers

### Distributed Systems
- **Consistent hashing:** Minimize rehashing when nodes added/removed
- **Hash ring:** Each node owns range of hash space
- **Replication:** Multiple nodes per key for fault tolerance

### Concurrent Access
- **Java ConcurrentHashMap:** Lock striping (segment-level locks)
- **Lock-free:** CAS operations for high concurrency
- **Trade-off:** Synchronization overhead vs correctness

## Advanced Concepts (L6/E6)

### Bloom Filters
- **Purpose:** Probabilistic set membership (false positives OK)
- **Space:** Much smaller than hash set
- **Use:** Web crawlers (billions of URLs), databases (skip disk reads)
- **Operations:** O(k) for k hash functions

### Consistent Hashing
- **Problem:** Adding/removing servers rehashes all keys
- **Solution:** Hash ring, each server owns arc, minimize movement
- **Use:** Load balancers, distributed caches, CDNs

### Cuckoo Hashing
- **Method:** Two hash functions, two tables
- **Guarantee:** O(1) worst-case lookup
- **Insert:** May evict and re-insert (cuckoo displacement)

## Memory Tricks

**Mnemonic - "FAST":**
- **F**requency counting (Counter/dict)
- **A**ccess O(1) (vs O(n) linear search)
- **S**een tracking (visited set)
- **T**wo Sum pattern (complement in hash)

## Quick Self-Test

1. Hash table average search time? *O(1)*
2. Hash table worst-case search? *O(n) with all collisions*
3. What makes good hash function? *Uniform distribution + deterministic*
4. Two Sum with hash map complexity? *O(n) time, O(n) space*
5. Hash table vs BST for sorted order? *BST - O(log n) ordered*

## Key Takeaways

- Hash tables provide O(1) average operations, O(n) worst case
- Critical for optimization: Two Sum, caching, frequency counting
- Trade space (O(n)) for time (O(1) vs O(n) or O(log n))
- Cannot hash mutable objects, must implement proper hashCode/equals
- Load factor balances speed vs memory (typical: 0.75)
- For L6/E6: discuss consistent hashing, bloom filters, concurrent access
- Real-world: caching (Redis), distributed systems (sharding), databases (hash indexes)
