# System Design Fundamentals

## A framework for any design question
1. **Clarify requirements** — functional + non-functional (scale, latency, consistency, availability).
2. **Estimate scale** — QPS, data size, read/write ratio.
3. **API design** — endpoints, request/response, pagination.
4. **High-level components** — client → gateway/LB → services → data stores / caches.
5. **Data model & storage** — SQL vs NoSQL, indexing, partitioning.
6. **Scale it** — horizontal scaling, load balancing, caching, async/queues, CDN.
7. **Resilience** — timeouts, retries, circuit breakers, graceful degradation → [Scalability & Resilience](scalability-resilience.md).
8. **Trade-offs** — state them explicitly.

## Building blocks
- **Load balancer** — L4 (transport) vs L7 (application); round-robin, least-connections.
- **Caching** — cut latency + load → [Caching](../distributed-cloud/caching.md).
- **Database** — replication (read scaling), sharding/partitioning (write scaling), indexing.
- **Message queue** — decouple + smooth spikes → [Messaging](../distributed-cloud/messaging.md).
- **CDN** — static assets near users.
- **API gateway** — routing, auth, rate limiting, aggregation.

## Key theory
- **CAP** — during a partition, choose Consistency or Availability → see [DB Principles](../databases/index.md).
- **Consistency models** — strong vs eventual.
- **Statelessness** — externalize session → any instance serves any request → horizontal scale.
- **Idempotency** — safe retries (idempotency keys for writes).
- **Pagination** — offset vs cursor/keyset (cursor scales for deep pages).

## Scaling axes
Vertical (bigger box) vs **horizontal** (more boxes). Prefer horizontal + stateless behind a load balancer.
