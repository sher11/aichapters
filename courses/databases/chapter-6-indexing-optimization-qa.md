---
layout: chapter
title: Indexing & Query Optimization - Q&A
course_id: databases
chapter_number: 6
---

**Quick Revision:** Master indexing and query optimization.

## Index Fundamentals

**Q1:** B-tree vs Hash index?
**A:** B-tree: O(log n), supports range queries. Hash: O(1) equality, no range/order.

**Q2:** What is leftmost prefix rule?
**A:** Composite index (a, b, c) can be used for (a), (a,b), (a,b,c) but not (b) or (c) alone.

**Q3:** What is a covering index?
**A:** Index includes all columns in SELECT - no table lookup needed. Fastest query type.

**Q4:** When NOT to create an index?
**A:** Small tables, high write/low read, low cardinality columns, rarely queried columns.

## Query Optimization

**Q5:** Why avoid SELECT *?
**A:** Fetches unnecessary data, prevents covering indexes, breaks code on schema changes.

**Q6:** Why is WHERE YEAR(date) = 2024 slow?
**A:** Function on indexed column prevents index use. Use range: WHERE date >= '2024-01-01'.

**Q7:** EXISTS vs COUNT(*) > 0?
**A:** EXISTS stops at first match. COUNT scans all rows. EXISTS is faster.

**Q8:** UNION vs UNION ALL?
**A:** UNION removes duplicates (expensive). UNION ALL keeps all (fast).

## Performance Issues

**Q9:** What is N+1 query problem and solution?
**A:** 1 query + N queries in loop. Solution: Use JOIN to fetch related data in single query.

**Q10:** How to diagnose slow query?
**A:** Use EXPLAIN ANALYZE. Look for Seq Scan (bad), Index Scan (good), Index Only Scan (best).

**Q11:** Too many indexes - what's the problem?
**A:** Storage overhead, write penalty (each write updates all indexes). Balance read speed vs write cost.

**Q12:** Index hit ratio - what is it?
**A:** Percentage of queries using indexes vs full scans. High ratio (>95%) is good.

## Key Insights

- B-tree default for most use cases (search, range, order)
- Leftmost prefix rule for composite indexes
- Covering indexes fastest (no table lookup)
- Index WHERE, JOIN, ORDER BY columns
- Don't over-index: write penalty and storage cost
- EXPLAIN shows execution plan - use it
- N+1 queries: use joins, not loops
- For L6/E6: Balance read performance vs write cost vs storage
