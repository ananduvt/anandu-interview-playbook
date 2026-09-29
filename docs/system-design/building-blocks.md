# Building Blocks

Reusable components you assemble in most system designs.

## Load balancer
Distributes traffic across instances. **L4** (TCP, fast) vs **L7** (HTTP-aware routing).
Algorithms: round-robin, least-connections, IP-hash. Health checks remove bad instances.

## Caching
Cut latency + DB load. Cache-aside, TTL + LRU, local vs distributed (Redis). See [Caching](../distributed-cloud/caching.md).

## Database scaling
- **Replication** — read scaling + HA (leader/follower).
- **Sharding/partitioning** — write scaling by shard key. See [DB Scaling](../databases/scaling.md).

## Message queue / stream
Decouple producers/consumers, absorb spikes, enable async & event-driven. Kafka (stream), RabbitMQ/SQS (queue).
See [Messaging](../distributed-cloud/messaging.md).

## API gateway
Single entry point: routing, auth, rate limiting, aggregation, TLS termination.

## CDN
Cache static assets at edge locations near users; reduces origin load and latency.

## Rate limiting
Protect services from overload/abuse. Algorithms: **token bucket**, leaky bucket, fixed/sliding window.
Often centralized in the gateway or Redis.

## Consistent hashing
Map keys → nodes on a ring so adding/removing a node reshuffles only ~1/N keys. Used by distributed caches/DBs.

## Consensus / coordination
Leader election, distributed locks, config — via **ZooKeeper / etcd / Raft**.

## ID generation
Unique IDs at scale: UUID, **Snowflake** (timestamp + machine + sequence), DB sequences.

## Idempotency & retries
Idempotency keys make retries safe; combine with timeouts, backoff, and circuit breakers
(see [Scalability & Resilience](scalability-resilience.md)).
