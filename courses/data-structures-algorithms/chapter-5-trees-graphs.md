---
layout: chapter
title: Trees & Graphs
course_id: data-structures-algorithms
chapter_number: 5
---

**One-liner:** Hierarchical and network structures essential for 50%+ of FAANG technical interviews, master traversals and graph algorithms.

## Tree Types & Complexities

| Tree Type | Search | Insert | Delete | Notes |
|-----------|--------|--------|--------|-------|
| Binary Search Tree | O(h) | O(h) | O(h) | h = height, O(n) worst |
| AVL Tree | O(log n) | O(log n) | O(log n) | Balanced, strict |
| Red-Black Tree | O(log n) | O(log n) | O(log n) | Java TreeMap |
| Trie | O(m) | O(m) | O(m) | m = key length |

## Graph Algorithms

### DFS vs BFS
- **DFS:** Less memory, find any path, cycle detection
- **BFS:** Shortest path guarantee, level-order

### Union-Find
- **Time:** O(α(n)) ≈ O(1) amortized with path compression
- **Use:** Connected components, cycle detection, MST

### Topological Sort
- **When:** DAG (Directed Acyclic Graph)
- **Time:** O(V + E)

## High-Frequency Problems

1. **Validate BST** - Recursive with range bounds
2. **Number of Islands** - DFS/BFS on grid
3. **Course Schedule** - Topological sort
4. **Clone Graph** - DFS + hash map
5. **Word Ladder** - BFS shortest transformation

## Key Takeaways

- Master DFS and BFS traversals - used in 50%+ of problems
- BST validation requires range checking, not just local comparison
- Union-find is fastest for connectivity problems
- BFS finds shortest paths in unweighted graphs
- Topological sort works only on DAGs
- For L6/E6: discuss B-trees for databases, graph sharding, bloom filters
