---
layout: chapter
title: System Design Fundamentals - Q&A
course_id: system-design
chapter_number: 2
---

**Quick Revision:** Test your system design fundamentals.

## CAP Theorem

**Q1:** What does CAP stand for and what's the real choice?
**A:** Consistency, Availability, Partition tolerance. Since partitions happen, choose CP (consistency) or AP (availability).

**Q2:** Banking system - CP or AP and why?
**A:** CP. Must guarantee consistency for account balances. Brief unavailability acceptable.

**Q3:** Social media feed - CP or AP and why?
**A:** AP. Always available, eventual consistency fine for likes/comments.

## Scalability

**Q4:** Vertical vs horizontal scaling trade-offs?
**A:** Vertical: Simple, expensive, limited. Horizontal: Unlimited, complex (sharding, consistency), cost-effective.

**Q5:** When is vertical scaling sufficient?
**A:** Small to medium scale (< 100K users), need ACID guarantees, simpler operations.

## Consistency

**Q6:** Strong vs eventual consistency - when each?
**A:** Strong: Banking, inventory (correctness critical). Eventual: Social feeds, DNS (speed/availability critical).

**Q7:** Read-your-writes consistency use case?
**A:** User sees own updates immediately (profile, comments) while others eventual.

## Performance

**Q8:** Why is caching so effective?
**A:** Memory 1000x faster than SSD, 100,000x faster than HDD. Latency: 100ns vs 150μs vs 10ms.

**Q9:** Calculate: 1M QPS, 5KB response, what bandwidth?
**A:** 1M × 5KB = 5GB/sec = 40Gbps.

## Key Insights

- CAP forces CP or AP choice in distributed systems
- Horizontal scaling is only path to infinite scale
- Cache aggressively - memory is 1000x faster than disk
- Always quantify: QPS, storage, bandwidth
- Discuss trade-offs: consistency vs availability, latency vs throughput
- For L6/E6: Justify every design choice with numbers and trade-offs
