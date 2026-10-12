# System Design

Design large-scale, distributed systems. Learn architectural principles, scalability patterns, and trade-offs for building systems that serve millions of users.

---

## Topics

### [Fundamentals](fundamentals.md)
**Scalability - Load Balancing - Caching - Database Sharding - Replication - Consistency**  
System design basics: horizontal vs. vertical scaling, load distribution, data partitioning, consistency models (ACID, BASE, CAP theorem), trade-offs between availability and consistency.

### [Building Blocks](building-blocks.md)
**Load Balancers - Caches - Databases - Message Queues - CDN - Rate Limiting - API Gateways**  
Core components: when to use each, configuration patterns, failure modes, monitoring, cost-benefit analysis, integration strategies.

### [Scalability & Resilience](scalability-resilience.md)
**Auto-scaling - Circuit Breakers - Timeouts - Retries - Bulkhead - Failover - Graceful Degradation**  
Building resilient systems: handling failures, recovery strategies, monitoring metrics (latency, throughput, error rates), alerting, chaos engineering.

### [Worked Examples](worked-examples.md)
**Twitter - YouTube - Uber - Payment System - Notification Service - Case Studies**  
Real-world system design examples: problem breakdown, architecture evolution, component interactions, trade-off decisions, estimated capacity calculations.

---

## Quick Interview Prep

**System design interview flow:**
1. Clarify requirements (functional + non-functional)
2. Estimate scale (users, QPS, storage, bandwidth)
3. High-level architecture (APIs, components, databases)
4. Deep-dive components (caching, queues, replication, partitioning)
5. Identify bottlenecks & scaling strategies
6. Discuss trade-offs (consistency, latency, cost)

**Key metrics to estimate:**
- QPS (queries per second), P99 latency, storage requirements
- Bandwidth, concurrent connections, cache hit ratio
