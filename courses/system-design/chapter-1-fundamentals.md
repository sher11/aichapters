---
layout: chapter
title: System Design Fundamentals
course_id: system-design
chapter_number: 1
---

**One-liner:** Core concepts and trade-offs that form the foundation of every distributed system design.

## CAP Theorem

**Consistency, Availability, Partition Tolerance - pick 2**

- **Consistency (C):** All nodes see same data at same time
- **Availability (A):** Every request gets response (success/failure)
- **Partition Tolerance (P):** System works despite network failures

**Reality:** Network partitions happen, so choose CP or AP
- **CP:** Wait for consistency, sacrifice availability (banking, inventory)
- **AP:** Always available, eventual consistency (social media, DNS)

## Scalability Approaches

### Vertical Scaling (Scale Up)
- Add more resources to single machine (CPU, RAM, disk)
- **Pros:** Simple, no code changes, ACID guarantees
- **Cons:** Hardware limits, single point of failure, expensive
- **Max:** ~96 cores, ~3TB RAM in practice

### Horizontal Scaling (Scale Out)
- Add more machines to distribute load
- **Pros:** No ceiling, fault tolerance, cost-effective
- **Cons:** Complex (sharding, consistency, transactions)
- **Required for:** Billions of users, petabytes of data

## Consistency Models

### Strong Consistency
- Read always returns most recent write
- **How:** Synchronous replication, quorum writes
- **Cost:** Higher latency, lower availability
- **Use:** Banking, inventory

### Eventual Consistency
- Reads may return stale data briefly
- **How:** Async replication, conflict resolution
- **Benefit:** Low latency, high availability
- **Use:** Social media, DNS, caching

### Read-Your-Writes Consistency
- User sees their own writes immediately
- Others may see stale data
- **Use:** User profiles, comments

## Latency Numbers Every Engineer Should Know

| Operation | Latency | Comparison |
|-----------|---------|------------|
| L1 cache | 0.5 ns | - |
| L2 cache | 7 ns | 14x L1 |
| RAM | 100 ns | 200x L1 |
| SSD | 150 μs | 1,500x RAM |
| HDD | 10 ms | 100,000x RAM |
| Network (same datacenter) | 500 μs | 5x SSD |
| Network (cross-continent) | 150 ms | 300x datacenter |

**Key Insight:** Memory is 1000x faster than SSD, network is 100x slower than SSD

## Load Balancing

### Algorithms
- **Round Robin:** Rotate through servers (simple, assumes equal capacity)
- **Least Connections:** Route to server with fewest active connections
- **Least Response Time:** Route to fastest responding server
- **IP Hash:** Same client → same server (session affinity)
- **Weighted:** Different capacities for different servers

### Layers
- **Layer 4 (Transport):** Based on IP/port, fast but no content awareness
- **Layer 7 (Application):** Based on HTTP headers/URL, flexible but slower

## Sharding (Data Partitioning)

### Strategies

**1. Range-Based**
- Users A-M → Shard 1, N-Z → Shard 2
- **Pro:** Range queries easy
- **Con:** Hotspots (uneven distribution)

**2. Hash-Based**
- Hash(user_id) mod N → Shard
- **Pro:** Uniform distribution
- **Con:** Range queries require all shards

**3. Geographic**
- US users → US shard, EU → EU shard
- **Pro:** Low latency, data residency
- **Con:** Cross-region queries expensive

### Challenges
- Cross-shard joins (avoid or denormalize)
- Resharding when adding nodes (use consistent hashing)
- Hotspots (celebrity problem - use cache)

## Caching Strategies

### Cache-Aside (Lazy Loading)
```
data = cache.get(key)
if not data:
    data = db.read(key)
    cache.set(key, data)
```
**Pro:** Only cache what's used | **Con:** Cache miss penalty

### Write-Through
```
db.write(key, data)
cache.set(key, data)
```
**Pro:** Cache always fresh | **Con:** Write latency

### Write-Behind (Write-Back)
```
cache.set(key, data)
async db.write(key, data)
```
**Pro:** Fast writes | **Con:** Data loss risk

### Refresh-Ahead
- Proactively refresh before expiration
- **Use:** Predictable access patterns

## Common Bottlenecks

1. **Database:** Add read replicas, cache, indexing
2. **CPU:** Horizontal scaling, async processing
3. **Network:** CDN, compression, fewer round trips
4. **Disk I/O:** SSDs, caching, in-memory databases

## Back-of-Envelope Estimation

**Example:** Design Twitter (300M DAU, 50M tweets/day)

**Reads:**
- 300M users × 100 timeline views/day = 30B reads/day
- 30B / 86,400s ≈ 350K reads/sec
- Peak (3x average) = 1M reads/sec

**Writes:**
- 50M tweets/day = 580 writes/sec
- Peak (5x) = 3K writes/sec

**Storage:**
- 50M tweets/day × 280 chars × 2 bytes = 28GB/day
- 28GB × 365 × 5 years ≈ 50TB

**Bandwidth:**
- 1M reads × 5KB response = 5GB/sec = 40Gbps

## Trade-offs to Discuss

### SQL vs NoSQL
- **SQL:** ACID, complex queries, structured data
- **NoSQL:** Scalability, flexible schema, high throughput
- **Reality:** Use both (polyglot persistence)

### Sync vs Async
- **Sync:** Consistency, simplicity, higher latency
- **Async:** Performance, complexity, eventual consistency

### Normalization vs Denormalization
- **Normalized:** No redundancy, complex joins
- **Denormalized:** Fast reads, storage cost, update complexity

## Key Takeaways

- CAP theorem: Network partitions force CP or AP choice
- Vertical scaling is simple but limited; horizontal scales infinitely but adds complexity
- Memory is 1000x faster than disk - cache aggressively
- Sharding distributes data but complicates queries and transactions
- Always discuss trade-offs: consistency vs availability, latency vs throughput
- For L6/E6: Quantify with numbers (QPS, storage, bandwidth)
- Start simple, identify bottlenecks, scale what's needed
