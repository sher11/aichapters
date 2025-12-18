---
layout: chapter
title: NoSQL Databases - Q&A
course_id: databases
chapter_number: 4
---

**Quick Revision:** Test your NoSQL understanding.

## Database Selection

**Q1:** When to use NoSQL over SQL?
**A:** Need horizontal scale (billions of records), flexible schema, simple access patterns, eventual consistency acceptable.

**Q2:** Key-value vs Document vs Column-family?
**A:** Key-value: Fastest, caching. Document: Flexible, nested data. Column-family: Write-heavy, time-series.

**Q3:** When to use Graph database?
**A:** Relationship-heavy queries (social networks, recommendations, fraud detection). When traversals are core feature.

## CAP Theorem

**Q4:** What is CAP theorem?
**A:** Consistency, Availability, Partition tolerance. Network partitions happen, so choose CP or AP.

**Q5:** Is Cassandra CP or AP and what does it mean?
**A:** AP - prioritizes availability and partition tolerance. Eventually consistent by default.

**Q6:** MongoDB CP or AP?
**A:** CP by default - can sacrifice availability for consistency during partition.

## Data Modeling

**Q7:** Embedding vs referencing in MongoDB?
**A:** Embed: Single query, atomic, data duplication. Reference: No duplication, multiple queries, no joins.

**Q8:** When to embed documents?
**A:** Data always accessed together, bounded size, one-to-few relationship.

**Q9:** What makes a good shard key?
**A:** High cardinality, even distribution, supports common queries. Avoid hotspots.

## Consistency

**Q10:** What is eventual consistency?
**A:** System becomes consistent over time. Reads may return stale data temporarily. Acceptable for feeds, caching.

**Q11:** How to achieve strong consistency in Cassandra?
**A:** R + W > N (read + write replicas > total replicas). E.g., QUORUM reads and writes.

**Q12:** Why does NoSQL sacrifice ACID?
**A:** Distributed transactions are expensive (2PC). Trade consistency for availability and performance at scale.

## Key Insights

- NoSQL trades ACID for scale and flexibility
- Four types serve different access patterns
- CAP forces CP or AP choice in distributed systems
- Embedding (fast) vs referencing (normalized)
- Shard key selection prevents hot partitions
- Eventual consistency fine for many use cases
- For L6/E6: Discuss polyglot persistence, when SQL is better
