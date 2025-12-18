---
layout: chapter
title: NoSQL Databases
course_id: databases
chapter_number: 3
---

**One-liner:** Non-relational databases optimized for scale, flexibility, and specific access patterns.

## NoSQL Types

### 1. Key-Value Stores

**Examples:** Redis, Memcached, DynamoDB

**Model:** Simple key -> value mapping
```python
# Redis example
SET user:1000 "{'name': 'Alice', 'email': 'alice@ex.com'}"
GET user:1000
INCR page_views:homepage
EXPIRE session:abc123 3600  # TTL in seconds
```

**Use Cases:**
- Session storage
- Caching
- Real-time analytics
- Shopping carts

**Pros:** Fastest (O(1)), simple, scalable
**Cons:** No complex queries, value is opaque blob

### 2. Document Stores

**Examples:** MongoDB, Couchbase, Firestore

**Model:** JSON-like documents
```javascript
// MongoDB example
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "Alice",
  "email": "alice@example.com",
  "addresses": [
    {"type": "home", "city": "NYC"},
    {"type": "work", "city": "SF"}
  ],
  "orders": [101, 102, 103]
}

// Query
db.users.find({ "addresses.city": "NYC" })
db.users.updateOne(
  { "_id": ObjectId("...") },
  { $push: { "orders": 104 } }
)
```

**Use Cases:**
- Content management
- User profiles
- Product catalogs
- Mobile/web apps

**Pros:** Flexible schema, nested data, easy horizontal scaling
**Cons:** No joins (denormalize or multi-query), eventual consistency

### 3. Column-Family Stores

**Examples:** Cassandra, HBase, ScyllaDB

**Model:** Rows with dynamic columns, grouped into column families
```
Row Key: user123
  profile:name -> "Alice"
  profile:email -> "alice@ex.com"
  metrics:page_views -> 1523
  metrics:last_login -> 2024-01-15
```

**Use Cases:**
- Time-series data
- IoT sensors
- Event logging
- Large-scale analytics

**Pros:** Write-optimized, petabyte scale, time-series queries
**Cons:** Complex data modeling, limited query flexibility

### 4. Graph Databases

**Examples:** Neo4j, Amazon Neptune, JanusGraph

**Model:** Nodes (entities) and edges (relationships)
```cypher
// Neo4j Cypher query
CREATE (alice:Person {name: 'Alice', age: 30})
CREATE (bob:Person {name: 'Bob', age: 28})
CREATE (alice)-[:FRIENDS_WITH {since: 2020}]->(bob)

// Find friends of friends
MATCH (me:Person {name: 'Alice'})-[:FRIENDS_WITH*2..3]->(friend)
RETURN friend.name
```

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

**Pros:** Relationship queries (O(1) traversal), intuitive modeling
**Cons:** Limited scale vs other NoSQL, specialized use case

## SQL vs NoSQL Comparison

| Factor | SQL | NoSQL |
|--------|-----|-------|
| Schema | Fixed, structured | Flexible, schema-less |
| Scaling | Vertical (primarily) | Horizontal (native) |
| ACID | Full support | Varies (BASE model) |
| Queries | Complex joins, SQL | Simple lookups, API calls |
| Consistency | Strong (default) | Eventual (tunable) |
| Use Case | Transactions, analytics | Scale, speed, flexibility |

## CAP Theorem and NoSQL

**CAP:** Consistency, Availability, Partition tolerance (pick 2)

**NoSQL Trade-offs:**
- **MongoDB:** CP (can choose consistency level)
- **Cassandra:** AP (tunable consistency)
- **Redis:** CP (with clustering) or single-node
- **DynamoDB:** AP (eventually consistent by default)

## Data Modeling Patterns

### Embedding vs Referencing

**Embed (Denormalize):**
```javascript
// User with embedded orders
{
  "user_id": 123,
  "name": "Alice",
  "orders": [
    {"order_id": 1, "total": 50},
    {"order_id": 2, "total": 75}
  ]
}
```
**Pros:** Single query, atomic updates
**Cons:** Data duplication, document size limits

**Reference (Normalize):**
```javascript
// User
{"user_id": 123, "name": "Alice"}

// Orders (separate collection)
{"order_id": 1, "user_id": 123, "total": 50}
{"order_id": 2, "user_id": 123, "total": 75}
```
**Pros:** No duplication, smaller docs
**Cons:** Multiple queries (N+1 problem), no joins

**Rule of Thumb:** Embed if always accessed together, reference if accessed independently.

## Sharding and Partitioning

### Shard Key Selection

**Good Shard Key:**
- High cardinality (many unique values)
- Even distribution
- Supports common queries

**Examples:**
- **User ID:** Good (random distribution)
- **Timestamp:** Bad (hotspots on latest data)
- **Country:** Bad (uneven distribution)
- **User ID + Timestamp:** Good (composite)

### Replication

**Primary-Secondary:**
- Write to primary, read from secondaries
- Eventual consistency (replication lag)
- High read throughput

**Multi-Master:**
- Write to any node
- Conflict resolution needed
- High write availability

## Consistency Levels

### Cassandra Example
- **ONE:** Read/write from 1 replica (fastest, least consistent)
- **QUORUM:** Majority of replicas (balanced)
- **ALL:** All replicas (slowest, most consistent)

**Formula:** R + W > N (R=read, W=write, N=replicas) → strong consistency

## Real-World Trade-offs (L6/E6)

### MongoDB vs PostgreSQL
- **Mongo:** Flexible schema, horizontal scale, eventual consistency
- **Postgres:** ACID, complex queries, vertical scale (with read replicas)
- **Reality:** Use both (polyglot persistence)

### Cassandra vs HBase
- **Cassandra:** AP, easier ops, better write throughput
- **HBase:** CP, tighter consistency, better for scans

### Redis vs Memcached
- **Redis:** Richer data structures (lists, sets, sorted sets), persistence
- **Memcached:** Simpler, slightly faster for pure caching

## High-Frequency Interview Topics

1. **When to use NoSQL over SQL** - Scale, schema flexibility, specific patterns
2. **CAP theorem** - Explain with examples (Mongo CP, Cassandra AP)
3. **Eventual consistency** - How it works, acceptable use cases
4. **Sharding strategies** - Hash, range, geo
5. **Embedding vs referencing** - Trade-offs in document stores
6. **Hot partitions** - Celebrity problem, solutions (caching, fanout)

## Common Gotchas

- **No joins:** Must denormalize or make multiple queries
- **Eventual consistency:** Reads may return stale data
- **Shard key immutable:** Can't change after creation
- **Document size limits:** MongoDB 16MB, plan accordingly
- **Hot shards:** Poor shard key causes bottlenecks
- **Secondary indexes:** Can be expensive in distributed systems

## Key Takeaways

- Four types: Key-value, Document, Column-family, Graph
- NoSQL trades ACID for scale and flexibility
- CAP theorem: Pick CP or AP (partition tolerance is mandatory)
- Embedding (fast reads) vs referencing (no duplication)
- Shard key selection is critical - high cardinality, even distribution
- Eventual consistency acceptable for many use cases (social feeds, caching)
- For L6/E6: Discuss when NOT to use NoSQL, consistency tuning, hot partitions
- Real-world: Polyglot persistence (SQL + NoSQL together)
