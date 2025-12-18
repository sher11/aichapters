---
layout: chapter
title: SQL Fundamentals - Q&A
course_id: databases
chapter_number: 2
---

**Quick Revision:** Test your SQL knowledge.

## Query Basics

**Q1:** Difference between WHERE and HAVING?
**A:** WHERE filters rows before grouping. HAVING filters groups after GROUP BY.

**Q2:** COUNT(*) vs COUNT(column)?
**A:** COUNT(*) counts all rows including NULLs. COUNT(column) excludes NULLs.

**Q3:** INNER JOIN vs LEFT JOIN?
**A:** INNER: only matching rows. LEFT: all left rows + matching right (NULL if no match).

## Advanced Queries

**Q4:** When to use window functions vs GROUP BY?
**A:** Window functions keep all rows (e.g., running total). GROUP BY collapses rows into groups.

**Q5:** What is a CTE and why use it?
**A:** Common Table Expression - temporary named result set. Improves readability, enables recursion.

**Q6:** ROW_NUMBER vs RANK vs DENSE_RANK?
**A:** ROW_NUMBER: unique sequential. RANK: gaps for ties. DENSE_RANK: no gaps.

## Schema Design

**Q7:** What is 3NF and why use it?
**A:** Third Normal Form - no transitive dependencies. Prevents update anomalies, reduces redundancy.

**Q8:** When to denormalize?
**A:** Read-heavy workloads, expensive joins, acceptable data duplication (analytics, reporting).

**Q9:** Composite primary key vs single surrogate key?
**A:** Composite: natural relationship (student_id, course_id). Surrogate: simpler joins, arbitrary ID.

## Performance

**Q10:** What is the N+1 query problem?
**A:** Executing 1 query + N queries in loop. Solution: Use JOIN to fetch related data in single query.

**Q11:** Why avoid SELECT *?
**A:** Fetches unnecessary data, breaks apps if schema changes, prevents covering indexes.

**Q12:** How does NULL affect queries?
**A:** NULL != NULL, use IS NULL. COUNT(col) excludes NULLs. JOIN on NULL doesn't match.

## Key Insights

- WHERE filters rows, HAVING filters groups
- Window functions keep all rows, GROUP BY collapses
- Normalize for transactional, denormalize for analytics
- Index on query patterns (WHERE, JOIN, ORDER BY)
- N+1 queries: use JOINs, not loops
- For L6/E6: Discuss execution plans, denormalization trade-offs
