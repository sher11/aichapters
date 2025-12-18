---
layout: chapter
title: Databases & Caching
course_id: system-design
chapter_number: 3
---

**One-liner:** Database architecture and caching strategies that power every large-scale system.

## SQL vs NoSQL

| Factor | SQL (Relational) | NoSQL |
|--------|------------------|-------|
| Schema | Fixed, structured | Flexible, schema-less |
| Scaling | Vertical (mainly) | Horizontal (native) |
| ACID | Full support | Varies (eventual consistency) |
| Queries | Complex joins, aggregations | Simple lookups, no joins |
| Use Cases | Financial, transactions | Social media, logs, time-series |

### When to Use SQL
- Structured data with relationships
- ACID transactions required (banking, orders)
- Complex queries and analytics
- Data integrity critical
- **Examples:** PostgreSQL, MySQL

### When to Use NoSQL
- Massive scale (billions of records)
- Flexible schema, rapid iteration
- High write throughput
- Geographic distribution
- **Types:** Key-value (Redis), Document (MongoDB), Column (Cassandra), Graph (Neo4j)

## Database Indexing

### B-Tree Index (Default)
- Sorted tree structure for fast lookups
- **Time:** O(log n) for search/insert/delete
- **Use:** Primary keys, frequent WHERE clauses
- **Cost:** Slower writes, storage overhead

### Hash Index
- Hash table for exact-match lookups
- **Time:** O(1) average
- **Limitation:** No range queries
- **Use:** Unique keys, equality checks only

### Composite Index
- Index on multiple columns: (user_id, created_at)
- **Order matters:** Can use for user_id alone, not created_at alone
- **Gotcha:** Leftmost prefix rule

## Read Scaling

### Read Replicas
- Master handles writes, replicas handle reads
- **Replication:** Async (eventual consistency) or sync (strong)
- **Ratio:** 1 master : 10+ replicas common
- **Challenge:** Replication lag (seconds to minutes)

### Database Sharding
- Partition data across multiple databases
- **Benefit:** Horizontal scaling for both reads and writes
- **Challenge:** Cross-shard queries, joins, transactions

## Caching Layers

### Application Cache (Redis/Memcached)
- In-memory key-value store
- **Speed:** Sub-millisecond latency
- **Use:** Session data, computed results, hot data
- **Eviction:** LRU, LFU, TTL

### CDN (Content Delivery Network)
- Distributed cache for static assets (images, CSS, JS)
- **Benefit:** Geographic proximity, reduce origin load
- **Invalidation:** TTL or explicit purge

### Database Query Cache
- Cache query results at database layer
- **Invalidation:** Write to any table in query invalidates cache
- **Problem:** Low hit rate for writes

## Cache Eviction Policies

| Policy | When Used | Pros | Cons |
|--------|-----------|------|------|
| LRU (Least Recently Used) | General purpose | Good hit rate | Doesn't consider frequency |
| LFU (Least Frequently Used) | Predictable patterns | Accounts for popularity | Slow to adapt |
| TTL (Time To Live) | Time-sensitive data | Automatic expiration | May serve stale |
| Random | Simple systems | Easy to implement | Suboptimal hit rate |

## Cache Patterns

### Cache-Aside
```
data = cache.get(key)
if not data:
    data = db.query(key)
    cache.set(key, data)
return data
```
**Pro:** Simple, only cache needed data
**Con:** Miss penalty, cache-DB inconsistency window

### Write-Through
```
cache.set(key, data)
db.write(key, data)
return data
```
**Pro:** Cache always consistent
**Con:** Write latency (2 operations)

### Write-Behind
```
cache.set(key, data)
queue.add(db_write, key, data)  # Async
```
**Pro:** Ultra-fast writes
**Con:** Data loss if cache fails

## Consistency Challenges

### Cache Invalidation
**Two hard things in CS:** Naming and cache invalidation

**Strategies:**
1. **TTL:** Expire after N seconds (simple, stale data)
2. **Invalidate on write:** Delete cache entry on DB update
3. **Refresh-ahead:** Proactively refresh before expiry

### Cache Stampede
**Problem:** Cache expires, 1000 requests hit DB simultaneously
**Solutions:**
- Lock while refreshing (only 1 query to DB)
- Probabilistic early expiration
- Always serve stale while refreshing

## Database Types Deep Dive

### Key-Value (Redis, Memcached)
- **Speed:** Fastest (in-memory)
- **Use:** Caching, sessions, real-time
- **Limitation:** Simple queries only

### Document (MongoDB, Couchbase)
- **Model:** JSON-like documents
- **Use:** Content management, catalogs
- **Pro:** Flexible schema
- **Con:** No joins (denormalize or multi-query)

### Column-Family (Cassandra, HBase)
- **Model:** Wide columns, optimized for writes
- **Use:** Time-series, logs, analytics
- **Scale:** Petabytes, millions of writes/sec

### Graph (Neo4j, Amazon Neptune)
- **Model:** Nodes and edges
- **Use:** Social networks, recommendations, fraud detection
- **Query:** Cypher, Gremlin (traverse relationships)

## Real-World Trade-offs (L6/E6)

### PostgreSQL vs MongoDB
- **Postgres:** Complex queries, ACID, less scale
- **Mongo:** Flexible schema, horizontal scale, eventual consistency

### Redis vs Memcached
- **Redis:** Richer data structures (lists, sets), persistence
- **Memcached:** Simpler, slightly faster, no persistence

### Sharding Strategies
- **Range:** Easy range queries, hotspots
- **Hash:** Uniform distribution, no range queries
- **Geo:** Low latency, data residency laws

## Key Takeaways

- SQL for structured data + ACID; NoSQL for scale + flexibility
- Index on query patterns, not all columns (write cost)
- Read replicas scale reads; sharding scales writes
- Cache layers: CDN → App cache → DB cache
- Cache invalidation is hard - TTL vs explicit invalidation
- LRU eviction for general use; LFU for stable patterns
- For L6/E6: Discuss replication lag, cache stampede, shard key selection
- Real-world: Polyglot persistence (SQL + NoSQL together)
