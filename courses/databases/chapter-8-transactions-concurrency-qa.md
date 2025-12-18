---
layout: chapter
title: Transactions & Concurrency - Q&A
course_id: databases
chapter_number: 8
---

**Quick Revision:** Test your transactions and concurrency knowledge.

## ACID Properties

**Q1:** What does ACID stand for?
**A:** Atomicity (all or nothing), Consistency (valid states), Isolation (no interference), Durability (persists).

**Q2:** How is atomicity enforced?
**A:** Undo log. On failure, database replays log to rollback partial changes.

**Q3:** How is durability guaranteed?
**A:** Write-ahead log (WAL). Changes logged to disk before commit returns. Recovery replays log.

## Isolation Levels

**Q4:** Four isolation levels from strictest to loosest?
**A:** SERIALIZABLE, REPEATABLE READ, READ COMMITTED, READ UNCOMMITTED.

**Q5:** What is dirty read and which level allows it?
**A:** Reading uncommitted data from another transaction. Only READ UNCOMMITTED allows.

**Q6:** Difference between non-repeatable read and phantom read?
**A:** Non-repeatable: same row changes. Phantom: new rows appear in result set.

**Q7:** Which isolation level to use for financial transactions?
**A:** SERIALIZABLE or REPEATABLE READ. Must prevent dirty reads and lost updates.

## Locking

**Q8:** Pessimistic vs optimistic locking?
**A:** Pessimistic: lock before read/write (FOR UPDATE). Optimistic: check version before commit.

**Q9:** When to use optimistic over pessimistic?
**A:** Low contention, high concurrency needs. Pessimistic better for high contention.

**Q10:** What is deadlock?
**A:** Circular wait: T1 waits for T2, T2 waits for T1. Neither can proceed.

**Q11:** How to prevent deadlocks?
**A:** Lock ordering (always same order), timeouts, keep transactions short, deadlock detection.

## Advanced Concepts

**Q12:** What is MVCC?
**A:** Multi-Version Concurrency Control. Readers see snapshot, writers don't block readers. Used in PostgreSQL, MySQL.

**Q13:** What is two-phase commit (2PC)?
**A:** Distributed transaction protocol. Phase 1: prepare (vote). Phase 2: commit/abort. Slow but consistent.

**Q14:** Saga pattern vs 2PC?
**A:** Saga: local transactions + compensating actions. Eventual consistency, higher availability. 2PC: strong consistency, blocking.

**Q15:** Lost update problem - solutions?
**A:** (1) Pessimistic lock (FOR UPDATE), (2) Optimistic lock (version), (3) Atomic operations (SET x = x + 1).

## Key Insights

- ACID guarantees data integrity in concurrent systems
- Higher isolation = more consistency, less concurrency
- Pessimistic (lock) vs optimistic (version) locking
- Deadlocks: prevent with lock ordering, timeout, detection
- MVCC: readers don't block writers (PostgreSQL, MySQL)
- 2PC for strong consistency (slow), Saga for eventual consistency
- For L6/E6: Discuss trade-offs, when to lower isolation, hot row strategies
