# 12 — System Design Basics (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🟡 Important  
> Covers: HLD, LLD, Scalability, Caching, Load Balancing, Databases, Common System Designs

---

## 📌 Table of Contents

1. [System Design Approach](#1-system-design-approach)
2. [Key Concepts](#2-key-concepts)
3. [Scalability](#3-scalability)
4. [Load Balancing](#4-load-balancing)
5. [Caching](#5-caching)
6. [Database Design](#6-database-design)
7. [Message Queues](#7-message-queues)
8. [CAP Theorem](#8-cap-theorem)
9. [Common System Designs](#9-common-system-designs)
10. [Interview Questions & Answers](#10-interview-questions--answers)

---

## 1. System Design Approach (Framework)

> Use this 4-step framework in every system design interview:

### Step 1: Requirements Clarification (2-3 min)
```
Functional Requirements:
- What features? (e.g., users can post, follow, like)
- Who are the users?
- What are the core use cases?

Non-Functional Requirements:
- Scale? (how many users? requests/sec?)
- Availability? (99.9%?)
- Latency? (< 200ms?)
- Consistency vs Availability tradeoff?
```

### Step 2: Back-of-Envelope Estimation (2-3 min)
```
Users: 10M daily active
Posts: 500K/day
Reads: 100:1 read/write ratio → 50M reads/day
QPS: 50M / 86400 ≈ 580 reads/sec
Storage: 500K × 1KB per post = 500MB/day → ~180GB/year
```

### Step 3: High-Level Design (10-15 min)
```
Draw components: Client → LB → API Servers → Cache → DB
Identify services, data flow, storage
```

### Step 4: Deep Dive (10-15 min)
```
Pick 2-3 interesting components and go deep:
- Database schema
- API design
- Caching strategy
- Scaling bottlenecks
```

---

## 2. Key Concepts

### Latency Numbers Every Programmer Should Know
| Operation | Time |
|-----------|------|
| L1 cache reference | 0.5 ns |
| L2 cache reference | 7 ns |
| RAM reference | 100 ns |
| SSD random read | 150 μs |
| HDD seek | 10 ms |
| Network round trip (same DC) | 0.5 ms |
| Network round trip (cross-continent) | 150 ms |

### Throughput Estimates
```
Web server: ~1,000-10,000 QPS (depending on logic)
Database: ~1,000-5,000 QPS (PostgreSQL)
Cache (Redis): ~100,000+ QPS
Message Queue (Kafka): ~100,000+ messages/sec per partition
```

---

## 3. Scalability

### Vertical Scaling (Scale Up)
```
Add more CPU, RAM, Disk to a single machine
✅ Simple, no code change
❌ Has a limit, single point of failure
```

### Horizontal Scaling (Scale Out)
```
Add more machines
✅ Theoretically unlimited, fault tolerant
❌ Complex (distributed systems, data consistency)
```

### Stateless Services
```
Stateless → can scale horizontally easily
Sessions stored in Redis/DB, not in-memory
Any server can handle any request
```

---

## 4. Load Balancing

```
Client → Load Balancer → Server 1
                       → Server 2
                       → Server 3

Algorithms:
- Round Robin: 1 → 2 → 3 → 1 → ...
- Weighted Round Robin: based on server capacity
- Least Connections: route to server with fewest active connections
- IP Hash: same client always goes to same server
- Consistent Hashing: for distributed caching

Tools: Nginx, HAProxy, AWS ALB/NLB, Spring Cloud Gateway
```

### L4 vs L7 Load Balancer
| Feature | L4 (Transport) | L7 (Application) |
|---------|----------------|-------------------|
| Layer | TCP/UDP | HTTP/HTTPS |
| Decisions | IP, Port | URL, Headers, Cookies |
| Speed | Faster | Slower (inspects content) |
| Example | AWS NLB | AWS ALB, Nginx |

---

## 5. Caching

### Cache Strategies

```
1. CACHE-ASIDE (Lazy Loading) — most common
   Read: Check cache → miss → read DB → populate cache
   Write: Write DB → invalidate cache

2. WRITE-THROUGH
   Write: Write cache → cache writes to DB
   ✅ Cache always consistent
   ❌ Higher write latency

3. WRITE-BEHIND (Write-Back)
   Write: Write cache → async write to DB
   ✅ Fast writes
   ❌ Risk of data loss

4. READ-THROUGH
   Read: Cache manages DB reads automatically
```

### Cache Eviction Policies
| Policy | Description |
|--------|-------------|
| LRU | Remove least recently used |
| LFU | Remove least frequently used |
| FIFO | Remove oldest |
| TTL | Remove after time expires |

### Where to Cache
```
Browser Cache → CDN → API Gateway Cache → Application Cache (Redis) → DB Query Cache
```

### Redis — Most Common Cache
```
Key-Value store, in-memory, supports:
- Strings, Lists, Sets, Sorted Sets, Hashes
- TTL (time-to-live)
- Pub/Sub
- Distributed locks
- ~100K+ ops/sec
```

---

## 6. Database Design

### SQL vs NoSQL

| Feature | SQL (PostgreSQL) | NoSQL (MongoDB) |
|---------|-----------------|-----------------|
| Schema | Fixed (structured) | Flexible (schema-less) |
| Scaling | Vertical (mostly) | Horizontal (sharding) |
| Transactions | ACID | Eventually consistent |
| Joins | ✅ Efficient | ❌ Limited |
| Best for | Complex queries, relationships | High volume, flexible data |

### Database Scaling
```
1. Read Replicas: Master (writes) → Replica1, Replica2 (reads)
2. Sharding: Split data across DBs by key (user_id % 4)
3. Partitioning: Split large table by range (by date, region)
4. Connection Pooling: HikariCP (reuse DB connections)
```

### Consistent Hashing — for distributed databases/caches
```
Nodes placed on a virtual ring
Key hashed → placed on ring → find next clockwise node
Adding/removing a node only affects neighbors
Used by: Cassandra, DynamoDB, Redis Cluster
```

---

## 7. Message Queues

### Why Use Message Queues?
```
1. Decoupling — services don't depend on each other
2. Async processing — don't block user for slow operations
3. Load leveling — handle traffic spikes
4. Reliability — messages persist until processed
```

### Kafka vs RabbitMQ
| Feature | Kafka | RabbitMQ |
|---------|-------|----------|
| Model | Log-based (pull) | Queue (push) |
| Ordering | Per partition | Per queue |
| Throughput | Very high | Medium |
| Replay | ✅ Yes (offset) | ❌ No (consumed = gone) |
| Best for | Event streaming, analytics | Task queues, RPC |

---

## 8. CAP Theorem

> **In a distributed system, you can only guarantee 2 out of 3:**

| Property | Meaning |
|----------|---------|
| **C**onsistency | Every read returns latest write |
| **A**vailability | Every request gets a response |
| **P**artition Tolerance | System works despite network splits |

```
P is mandatory (networks will fail), so real choice is:
- CP: Consistent + Partition-tolerant (HBase, MongoDB)
  → May reject requests during partition
- AP: Available + Partition-tolerant (Cassandra, DynamoDB)
  → May return stale data during partition
```

---

## 9. Common System Designs

### URL Shortener (TinyURL)
```
Key decisions:
- Base62 encoding of auto-increment ID
- Hash table: shortCode → longURL
- Read-heavy → cache heavily (Redis)
- 301 (permanent) vs 302 (temporary) redirect

Components: LB → API Server → Redis Cache → Database
```

### Rate Limiter
```
Algorithms:
1. Token Bucket — refill tokens at fixed rate
2. Sliding Window — count requests in rolling time window
3. Fixed Window — count per time interval

Storage: Redis (INCR + EXPIRE for sliding window)
Response: 429 Too Many Requests + Retry-After header
```

### Notification System
```
User → API → Notification Service → Queue (Kafka)
                                         ↓
                              ┌──────────┼──────────┐
                              ↓          ↓          ↓
                          Email       Push        SMS
                          Service    Service    Service
```

### Chat Application
```
- WebSocket for real-time communication
- Message queue for async delivery
- Store messages in Cassandra (high write throughput)
- Presence service (online/offline) → Redis
```

---

## 10. Interview Questions & Answers

### Q1: How would you design a system that handles 1 million requests per second?
**A:** Horizontal scaling with stateless services behind load balancers, CDN for static content, Redis caching to reduce DB load, database read replicas, asynchronous processing via message queues (Kafka), database sharding if needed.

### Q2: How does consistent hashing work?
**A:** Nodes and keys are mapped to a hash ring. Each key is assigned to the first node found clockwise. When adding/removing a node, only keys between it and the previous node need to be redistributed — minimizing data movement.

### Q3: When would you choose SQL over NoSQL?
**A:** SQL for complex queries with joins, strong consistency needs (banking), well-defined schema, transactions. NoSQL for flexible/evolving schema, horizontal scaling, high write throughput, hierarchical/document data.

### Q4: How do you ensure data consistency across microservices?
**A:** Saga pattern (choreography or orchestration), eventual consistency with events, idempotent operations, outbox pattern (write events to DB table, async publish), compensating transactions for rollback.

---

> **Next:** [13_Docker_Kubernetes.md](13_Docker_Kubernetes.md) →
