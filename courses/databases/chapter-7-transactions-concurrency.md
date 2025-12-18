---
layout: chapter
title: Transactions & Concurrency
course_id: databases
chapter_number: 7
---

**One-liner:** ACID properties and isolation levels ensure data integrity in concurrent multi-user environments.

## ACID Properties

### Atomicity
**All or nothing** - transaction either completes fully or rolls back.

```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both succeed or both fail
```

**Failure scenarios:**
- System crash: undo log restores state
- Constraint violation: automatic rollback
- Explicit ROLLBACK

### Consistency
**Valid state to valid state** - constraints preserved.

```sql
-- CHECK constraint enforced
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL CHECK (balance >= 0)
);

-- Transaction fails if balance goes negative
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
-- Error: CHECK constraint violated
```

### Isolation
**Concurrent transactions don't interfere** - appears serial.

**Isolation levels** (strictest to loosest):
1. SERIALIZABLE
2. REPEATABLE READ
3. READ COMMITTED
4. READ UNCOMMITTED

### Durability
**Committed data persists** - survives crashes.

- Write-ahead log (WAL)
- Data flushed to disk before commit returns
- Recovery replays log after crash

## Isolation Levels

### Concurrency Problems

**1. Dirty Read**
Read uncommitted data from another transaction.

```
T1: UPDATE accounts SET balance = 500 WHERE id = 1;
T2: SELECT balance FROM accounts WHERE id = 1;  -- Reads 500
T1: ROLLBACK;  -- T2 read invalid data!
```

**2. Non-Repeatable Read**
Same query returns different results within transaction.

```
T1: SELECT balance FROM accounts WHERE id = 1;  -- Returns 1000
T2: UPDATE accounts SET balance = 500 WHERE id = 1;
T2: COMMIT;
T1: SELECT balance FROM accounts WHERE id = 1;  -- Returns 500
```

**3. Phantom Read**
New rows appear in query result.

```
T1: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 5
T2: INSERT INTO accounts VALUES (6, 200);
T2: COMMIT;
T1: SELECT COUNT(*) FROM accounts WHERE balance > 100;  -- Returns 6
```

### Isolation Level Guarantees

| Level | Dirty Read | Non-Repeatable | Phantom |
|-------|------------|----------------|---------|
| READ UNCOMMITTED | Yes | Yes | Yes |
| READ COMMITTED | No | Yes | Yes |
| REPEATABLE READ | No | No | Yes |
| SERIALIZABLE | No | No | No |

### Choosing Isolation Level

```sql
-- PostgreSQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- MySQL
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**Trade-offs:**
- **Higher isolation:** More consistency, less concurrency
- **Lower isolation:** More concurrency, less consistency

**Defaults:**
- PostgreSQL: READ COMMITTED
- MySQL: REPEATABLE READ
- Oracle: READ COMMITTED

## Locking Mechanisms

### Pessimistic Locking
Acquire lock before reading/writing.

```sql
-- Exclusive lock (write)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = 500 WHERE id = 1;
COMMIT;

-- Shared lock (read)
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
-- Others can read but not write
```

**Pros:** Prevents conflicts
**Cons:** Reduced concurrency, deadlock risk

### Optimistic Locking
Check version before commit.

```sql
-- Add version column
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    balance DECIMAL,
    version INT DEFAULT 0
);

-- Read with version
SELECT balance, version FROM accounts WHERE id = 1;
-- version = 5, balance = 1000

-- Update with version check
UPDATE accounts 
SET balance = 900, version = version + 1
WHERE id = 1 AND version = 5;

-- If 0 rows affected, conflict occurred (retry)
```

**Pros:** High concurrency
**Cons:** Retry logic needed, conflicts on high contention

## Deadlocks

### What is Deadlock?

```
T1: LOCK accounts WHERE id = 1;
T2: LOCK accounts WHERE id = 2;
T1: LOCK accounts WHERE id = 2;  -- Waits for T2
T2: LOCK accounts WHERE id = 1;  -- Waits for T1 (DEADLOCK!)
```

### Prevention Strategies

**1. Lock Ordering**
Always acquire locks in same order.

```sql
-- Always lock lower ID first
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
```

**2. Timeout**
Abort transaction after waiting too long.

```sql
SET lock_timeout = '5s';
```

**3. Deadlock Detection**
Database detects cycle, aborts one transaction.

**4. Reduce Lock Duration**
Keep transactions short.

## Multi-Version Concurrency Control (MVCC)

**How it works:**
- Each transaction sees snapshot of data
- Writers don't block readers
- Readers don't block writers

```
T1 (started at time 10): SELECT balance FROM accounts WHERE id = 1;
T2 (time 11): UPDATE accounts SET balance = 500 WHERE id = 1;
T2 (time 12): COMMIT;
T1 (time 13): SELECT balance FROM accounts WHERE id = 1;
-- T1 still sees old value (REPEATABLE READ)
```

**Benefits:**
- High concurrency
- No read locks
- Consistent snapshots

**Cost:**
- Vacuum needed (cleanup old versions)
- More storage

**Databases:** PostgreSQL, MySQL InnoDB, Oracle

## Distributed Transactions

### Two-Phase Commit (2PC)

**Phase 1: Prepare**
- Coordinator asks all nodes: "Can you commit?"
- Nodes write to log, reply YES or NO

**Phase 2: Commit**
- If all YES: Coordinator tells all to COMMIT
- If any NO: Coordinator tells all to ROLLBACK

**Problems:**
- Blocking: Nodes wait for coordinator
- Single point of failure
- High latency (2 network round-trips)

### Saga Pattern (Alternative)

**Choreography:**
- Each service completes local transaction
- On failure, compensating transactions undo changes

```
Order Service: Create order → Success
Payment Service: Charge card → Failure
Order Service: Cancel order (compensating transaction)
```

**Pros:** No coordinator, higher availability
**Cons:** Eventual consistency, complex error handling

## Concurrency Patterns (L6/E6)

### Read-Modify-Write

**Problem:** Lost updates

```
T1: SELECT balance FROM accounts WHERE id = 1;  -- 1000
T2: SELECT balance FROM accounts WHERE id = 1;  -- 1000
T1: UPDATE accounts SET balance = 900 WHERE id = 1;  -- -100
T2: UPDATE accounts SET balance = 800 WHERE id = 1;  -- -200 (LOST T1!)
```

**Solution 1: Pessimistic lock**
```sql
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
```

**Solution 2: Optimistic lock (version)**
```sql
UPDATE accounts SET balance = 900, version = version + 1
WHERE id = 1 AND version = 5;
```

**Solution 3: Atomic update**
```sql
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```

### Hot Row Contention

**Problem:** Many transactions update same row (e.g., like counter)

**Solutions:**
1. **Sharding:** Split counter across multiple rows
2. **Buffering:** Batch updates
3. **Redis:** In-memory increments, periodic sync to DB
4. **Optimistic concurrency:** Accept conflicts, retry

## Key Takeaways

- ACID: Atomicity (all/nothing), Consistency (valid state), Isolation (no interference), Durability (persists)
- Isolation levels: SERIALIZABLE (strictest) to READ UNCOMMITTED (loosest)
- Higher isolation = more consistency, less concurrency
- Pessimistic (lock first) vs optimistic (check version) locking
- Deadlocks: prevent with lock ordering, timeout, or detection
- MVCC: readers don't block writers (PostgreSQL, MySQL)
- 2PC for distributed transactions (slow), Saga for eventual consistency
- For L6/E6: Discuss isolation level trade-offs, deadlock scenarios, distributed transactions
- Production: Balance consistency needs vs performance
