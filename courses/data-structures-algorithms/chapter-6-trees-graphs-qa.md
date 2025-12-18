---
layout: chapter
title: Trees & Graphs - Q&A
course_id: data-structures-algorithms
chapter_number: 6
---

**Quick Revision:** Test your trees and graphs knowledge.

## Complexity Questions

**Q1:** In-order traversal of BST gives what order?
**A:** Sorted ascending order. Left subtree < Root < Right subtree.

**Q2:** DFS vs BFS space complexity?
**A:** DFS: O(h) for stack depth. BFS: O(w) for queue width. Worst case both O(n).

**Q3:** Union-find with path compression - time per operation?
**A:** O(α(n)) where α is inverse Ackermann, effectively O(1).

**Q4:** Dijkstra's algorithm time complexity?
**A:** O((V + E) log V) with min heap.

## Pattern Recognition

**Q5:** "Find shortest path in unweighted graph" - which algorithm?
**A:** BFS. Guarantees shortest path for unweighted edges.

**Q6:** "Detect cycle in directed graph" - approach?
**A:** DFS with three states: unvisited, in-progress, done. Cycle if you revisit in-progress node.

**Q7:** "All tasks with dependencies, find valid order" - what algorithm?
**A:** Topological sort using DFS or Kahn's algorithm (BFS + in-degree).

**Q8:** "Number of connected components in undirected graph"?
**A:** DFS/BFS from each unvisited node, count starts. Or use union-find.

## Common Mistakes

**Q9:** What's wrong with this BST validation?
```
def isValid(node):
    return node.left.val < node.val < node.right.val
```
**A:** Only checks immediate children. Must verify entire left subtree < root < entire right subtree using range bounds.

**Q10:** BFS shortest path - when does it fail?
**A:** Weighted edges. Use Dijkstra for positive weights, Bellman-Ford for negative weights.

## Key Insights

- DFS: O(h) space, good for connectivity and cycles
- BFS: O(w) space, shortest path guarantee
- Union-find: Dynamic connectivity in near O(1)
- Topological sort only works on DAGs
- BST operations are O(h), balanced trees guarantee O(log n)
- For L6: discuss B-trees for databases, sharding for social graphs
