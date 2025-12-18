---
layout: chapter
title: Databases & Caching - Q&A
course_id: system-design
chapter_number: 4
---

**Quick Revision:** Master database and caching concepts.

## Database Selection

**Q1:** When to use SQL vs NoSQL?
**A:** SQL: Structured data, ACID, complex queries. NoSQL: Massive scale, flexible schema, high writes.

**Q2:** Key-value vs Document vs Column-family stores?
**A:** Key-value (Redis): Fastest, simple. Document (Mongo): Flexible JSON. Column (Cassandra): Write-heavy, time-series.

## Indexing

**Q3:** Why not index every column?
**A:** Indexes slow writes (must update index) and use storage. Only index frequent query patterns.

**Q4:** Composite index (a, b) - can you query on b alone efficiently?
**A:** No. Leftmost prefix rule - can query (a) or (a, b), not (b) alone.

## Scaling Reads

**Q5:** 10M reads/sec, 10K writes/sec - architecture?
**A:** Master-replica: 1 master for writes, 100+ read replicas. Async replication (eventual consistency OK).

**Q6:** What is replication lag and when does it matter?
**A:** Delay between master write and replica update. Matters for read-your-writes (user sees own data).

## Caching

**Q7:** Cache-aside vs write-through - when each?
**A:** Cache-aside: Read-heavy, lazy load. Write-through: Need consistency, can afford write latency.

**Q8:** What is cache stampede and how to prevent?
**A:** Many requests hit DB when cache expires. Solution: Lock during refresh, only 1 query to DB.

**Q9:** LRU vs LFU eviction?
**A:** LRU: Recency-based, good general use. LFU: Frequency-based, good for stable patterns.

## Key Insights

- SQL and NoSQL coexist (polyglot persistence)
- Index on queries, not columns (write cost matters)
- Read replicas scale reads 100x; sharding scales writes
- Cache at multiple layers: CDN, app, database
- Cache invalidation is hardest problem - use TTL or explicit invalidation
- For L6/E6: Discuss replication lag, cache consistency, shard key hotspots
