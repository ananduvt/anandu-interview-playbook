# System Design Interview Questions

Designing scalable, reliable systems: architecture decisions, trade-offs, real-world scenarios.

---

## Question Categories

### Fundamentals & Building Blocks
- Scalability, availability, consistency
- Load balancing, caching, databases
- CDN, message queues, microservices

### Design Patterns & Strategies
- Sharding, replication, partitioning
- Master-slave, leader-election
- CQRS, event sourcing
- Saga pattern, compensation

### Real-World System Design
- URL shortener (bit.ly)
- Social media feed (Twitter, Facebook)
- Search engine (Google)
- E-commerce platform (Amazon)
- Video streaming (Netflix, YouTube)
- Real-time chat (Slack, Discord)
- Ride-sharing (Uber, Lyft)
- Payment systems
- Distributed cache (Redis, Memcached)

### Trade-offs & Analysis
- CAP theorem implications
- Consistency levels: strong, eventual
- Availability vs consistency
- Latency vs throughput
- Cost vs performance

### Capacity Planning
- QPS (queries per second) estimation
- Storage requirements
- Bandwidth calculations
- Resource provisioning

### Failure Scenarios & Resilience
- Handling server failures
- Network partitions, Byzantine failures
- Data loss prevention, backups
- Graceful degradation

---

## Common Questions & Answers

### CAP Theorem & Trade-offs
**Q: Explain CAP theorem and how to choose between consistency and availability.**

In distributed systems, guarantee only **2 of 3**:
- **Consistency**: All replicas have same data
- **Availability**: System responds to all requests
- **Partition Tolerance**: System works despite network failures

**Practical implications:**
- **CP Systems** (HBase, Zookeeper): Block writes during network split → consistent but unavailable
- **AP Systems** (Cassandra, DynamoDB): Accept writes → available but eventually consistent

**Most systems are AP** because network partitions are inevitable.

**Example: 1M-user backend design**
- Prioritize availability (accept eventual consistency)
- Use master-slave replication with read replicas
- Cache heavily to reduce DB load
- Implement circuit breakers to degrade gracefully

---

### Caching Strategy: Redis vs Memcached
**Q: How would you implement a distributed cache for a system with 1M users?**

**Architecture:**
```
Client → App Servers → Cache (Redis/Memcached) → Database
                 ↓
          (Cache miss → fetch DB)
```

**Caching Pattern (Cache-Aside):**
```
1. Request hits app server
2. Check cache for key
3. If HIT: return cached value
4. If MISS: fetch from DB, cache it, return to client
5. Set TTL to invalidate stale data
```

**Redis vs Memcached:**
| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Strings, hashes, lists, sets, sorted sets | Strings only |
| Persistence | Yes (AOF, RDB) | None (in-memory) |
| Replication | Yes (master-slave) | No |
| Performance | Very fast | Slightly faster for simple ops |
| Use case | Complex data, Sessions, Pub/Sub | Simple cache, Leaderboards |

**For 1M users:** Redis cluster with sharding + master-slave replication.

---

### Circuit Breaker Pattern
**Q: Explain the circuit breaker pattern and Hystrix/Resilience4j.**

**Problem:** Cascading failures when downstream service is down

**Solution: Circuit Breaker** (3 states)
```
CLOSED → (requests flow) → OPEN (failure threshold reached)
                           ↓
                      Half-Open (after timeout, test request)
                           ↓
                      CLOSED (if ok) or OPEN (if fail)
```

**Resilience4j Configuration:**
```java
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public User getUser(Long id) {
    return userClient.getUser(id);  // May fail
}

public User fallback(Long id, Exception e) {
    return new User("Unknown");  // Fallback response
}
```

**Benefits:**
- Prevent cascading failures
- Allow time for recovery
- Graceful degradation

---

### Failover & High Availability
**Q: How would you design high availability with automatic failover?**

**Architecture:**
```
Load Balancer → [Primary Server]
             ↘ [Replica Server] (standby)
             
Health checks every N seconds
If Primary fails → DNS/LB redirects to Replica
```

**Approaches:**
1. **Active-Passive**: Replica idle, takes over if primary fails
2. **Active-Active**: Both serve traffic, if one fails other handles load
3. **Master-Slave Replication**: Automatic replication to slave, manual failover to slave

**Tools:** HAProxy, Nginx, Cloud LB (AWS ELB, GCP LB)

---

### Handling Data Consistency in Distributed Systems
**Q: How do you handle data consistency in distributed systems?**

**Consistency Levels:**
1. **Strong Consistency**: All nodes see same data immediately
   - High latency, complex coordination
   - Used for: Financial systems
2. **Eventual Consistency**: All nodes eventually see same data
   - Low latency, simpler architecture
   - Used for: Social media, caches
3. **Causal Consistency**: Related data consistent, others may lag
   - Intermediate approach

**Implementation strategies:**
- **Synchronous replication**: Wait for all replicas → slow, consistent
- **Asynchronous replication**: Fire-and-forget → fast, eventually consistent
- **Read-your-writes consistency**: Client always sees their writes (session affinity)
- **Two-phase commit (2PC)**: For critical transactions (distributed across DBs)

---

### Idempotency in System Design
**Q: Why is idempotency important in distributed systems?**

**Problem:** Network failures cause retries; how to ensure operation runs once?

**Example:**
```
Client → Server: "Transfer $100 from A→B"
Server processes, but response lost
Client retries → Server transfers $100 AGAIN! (Double charge)
```

**Solution: Idempotent Keys**
```
Client Header: Idempotent-Key: <unique-uuid>
Server checks: If seen before → return cached response
              If new → process + cache result
```

**Server-side implementation:**
```java
@PostMapping("/transfers")
public Response transfer(
    @Header("Idempotent-Key") String key,
    @RequestBody TransferRequest req) {
    
    // Check if we've seen this key before
    Response cached = cache.get(key);
    if (cached != null) return cached;
    
    // Process transfer
    Response result = processTransfer(req);
    cache.put(key, result);  // Cache for retries
    return result;
}
```

---

## Sample Topics (To expand)

- Design a URL shortener (bit.ly)
- Design a social media feed (Twitter)
- Design a real-time chat system (Slack)
- Design a video streaming platform (Netflix)
- Design an e-commerce system (Amazon)
- Design a search engine
- How would you scale a system to 1 million concurrent users?
- How do you choose between SQL and NoSQL?
- Explain sharding strategies and trade-offs
- Design a distributed cache
- How would you implement leader election?
- Explain event-driven architecture
- Design a payment processing system
- How would you monitor and debug a distributed system?

---

[More content to be added progressively]

**See also:** [System Design Fundamentals](../../system-design/fundamentals.md), [Scalability & Resilience](../../system-design/scalability-resilience.md), [Worked Examples](../../system-design/worked-examples.md), [Interview Questions Overview](../index.md)
