---
layout: chapter
title: Indexing & Query Optimization
course_id: databases
chapter_number: 5
---

**One-liner:** Indexes dramatically speed reads at the cost of write performance and storage - critical for production databases.

## Index Types

### B-Tree Index (Default)

**Structure:** Balanced tree, sorted order

```sql
CREATE INDEX idx_users_email ON users(email);
```

**Operations:**
- **Search:** O(log n)
- **Range queries:** Efficient (sorted)
- **Insert/Update/Delete:** O(log n)

**Use:** Primary keys, foreign keys, WHERE clauses, ORDER BY

### Hash Index

**Structure:** Hash table

```sql
CREATE INDEX idx_users_id USING HASH ON users(id);
```

**Operations:**
- **Exact match:** O(1) average
- **Range queries:** Not supported
- **Order:** No ordering

**Use:** Equality comparisons only (e.g., user_id = 123)

### Composite (Multi-Column) Index

```sql
CREATE INDEX idx_users_country_city ON users(country, city);
```

**Leftmost Prefix Rule:**
- Can use for: (country), (country, city)
- Cannot use for: (city) alone

**Order matters:**
```sql
-- Good: uses index
WHERE country = 'US' AND city = 'NYC'
WHERE country = 'US'

-- Bad: doesn't use index
WHERE city = 'NYC'
```

### Covering Index

Index includes all columns needed by query - no table lookup required.

```sql
-- Index covers all selected columns
CREATE INDEX idx_cover ON users(country, city, name);

-- Query uses covering index (fast!)
SELECT name FROM users WHERE country = 'US' AND city = 'NYC';
```

### Partial (Filtered) Index

Index only subset of rows.

```sql
CREATE INDEX idx_active_users ON users(email) 
WHERE status = 'active';
```

**Benefit:** Smaller index, faster queries on filtered subset

### Full-Text Index

For text search.

```sql
CREATE FULLTEXT INDEX idx_content ON articles(title, body);

SELECT * FROM articles WHERE MATCH(title, body) AGAINST('database optimization');
```

## When to Index

### Index These
- **Primary keys** (automatic)
- **Foreign keys** (join columns)
- **WHERE clause columns** (frequent filters)
- **ORDER BY columns** (sorting)
- **GROUP BY columns** (grouping)

### Don't Index These
- **Small tables** (< 1000 rows) - full scan is faster
- **High write, low read columns**
- **Low cardinality** (few unique values, e.g., boolean)
- **Columns rarely queried**

## Index Cost

### Storage Overhead
- Each index duplicates data
- B-tree: ~50-100% of table size
- 10 indexes = 10x storage

### Write Penalty
- **INSERT:** Update all indexes
- **UPDATE:** Update affected indexes
- **DELETE:** Update all indexes

**Rule:** Index for queries, not just because you can.

## Query Optimization Techniques

### 1. EXPLAIN / EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'alice@ex.com';
```

**Look for:**
- **Seq Scan:** Full table scan (bad for large tables)
- **Index Scan:** Using index (good)
- **Index Only Scan:** Covering index (best)
- **Nested Loop:** Join method
- **Rows:** Estimated vs actual

### 2. Avoid SELECT *

```sql
-- Bad: fetches all columns
SELECT * FROM users WHERE id = 123;

-- Good: only needed columns (may use covering index)
SELECT name, email FROM users WHERE id = 123;
```

### 3. Limit Result Sets

```sql
-- Bad: returns millions of rows
SELECT * FROM orders;

-- Good: paginate
SELECT * FROM orders ORDER BY created_at DESC LIMIT 100 OFFSET 0;
```

### 4. Use EXISTS Instead of COUNT

```sql
-- Bad: counts all rows
SELECT CASE WHEN COUNT(*) > 0 THEN 1 ELSE 0 END
FROM orders WHERE user_id = 123;

-- Good: stops at first match
SELECT CASE WHEN EXISTS(
    SELECT 1 FROM orders WHERE user_id = 123
) THEN 1 ELSE 0 END;
```

### 5. Avoid Functions on Indexed Columns

```sql
-- Bad: function prevents index use
WHERE YEAR(created_at) = 2024;

-- Good: range comparison uses index
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### 6. Use UNION ALL Instead of UNION

```sql
-- Bad: UNION removes duplicates (expensive)
SELECT id FROM users WHERE country = 'US'
UNION
SELECT id FROM users WHERE country = 'UK';

-- Good: UNION ALL keeps duplicates (fast)
SELECT id FROM users WHERE country = 'US'
UNION ALL
SELECT id FROM users WHERE country = 'UK';
```

## Join Optimization

### Join Order Matters

```sql
-- Good: filter before join
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.country = 'US';

-- Better: explicit subquery (optimizer may do this)
SELECT u.name, o.total
FROM (SELECT * FROM users WHERE country = 'US') u
JOIN orders o ON u.id = o.user_id;
```

### Use Appropriate Join Type

- **INNER JOIN:** Only matching rows (fastest)
- **LEFT JOIN:** Keep all left rows (slower)
- **RIGHT/FULL JOIN:** Rare, consider restructuring

### Index Join Columns

```sql
-- Both columns should be indexed
CREATE INDEX idx_users_id ON users(id);
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

## Database-Specific Optimizations

### PostgreSQL

**Vacuum:** Reclaim space from deleted rows
```sql
VACUUM ANALYZE users;
```

**Statistics:** Help query planner
```sql
ANALYZE users;
```

### MySQL

**Query Cache:** Cache result sets (deprecated in 8.0)

**InnoDB:** Uses clustered index (data stored with primary key)

## Common Performance Issues

### N+1 Queries

**Problem:**
```python
# 1 query + N queries
users = db.query("SELECT * FROM users")
for user in users:
    orders = db.query(f"SELECT * FROM orders WHERE user_id = {user.id}")
```

**Solution:**
```python
# Single query with join
results = db.query("""
    SELECT u.*, o.*
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
""")
```

### Missing Indexes

**Symptom:** Slow queries, high CPU
**Diagnosis:** EXPLAIN shows Seq Scan
**Fix:** Add index on WHERE/JOIN columns

### Too Many Indexes

**Symptom:** Slow writes, high storage
**Diagnosis:** Many unused indexes
**Fix:** Drop unused indexes, consolidate with composite indexes

### Large Table Scans

**Symptom:** Query times increase with data size
**Fix:** Add WHERE clause, index, or partition table

## Monitoring and Metrics

### Key Metrics
- **Query time:** p50, p95, p99 latency
- **Index hit ratio:** % queries using indexes
- **Cache hit ratio:** % queries served from cache
- **Slow query log:** Queries exceeding threshold

### Tools
- **EXPLAIN / EXPLAIN ANALYZE:** Query execution plan
- **pg_stat_statements:** PostgreSQL query stats
- **slow_query_log:** MySQL slow queries
- **Query analyzers:** Datadog, New Relic, AWS RDS Performance Insights

## Key Takeaways

- B-tree indexes: O(log n) search, range queries, most common
- Composite indexes: Leftmost prefix rule
- Covering indexes: Best performance (no table lookup)
- Index WHERE, JOIN, ORDER BY columns
- Don't over-index: Storage cost, write penalty
- EXPLAIN is your friend: Always check execution plan
- Avoid SELECT *, functions on indexed columns, COUNT when EXISTS works
- N+1 queries: Use joins, not loops
- For L6/E6: Discuss index types, when not to index, monitoring slow queries
- Balance: Read speed vs write speed vs storage
