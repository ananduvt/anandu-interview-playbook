# Scaling & NoSQL

## SQL vs NoSQL
| | SQL (RDBMS) | NoSQL |
|--|-------------|-------|
| Schema | fixed, structured | flexible / schemaless |
| Scaling | vertical (mostly) | horizontal (built-in) |
| Consistency | strong (ACID) | tunable / eventual (BASE) |
| Joins | yes | limited/none |
| Best for | transactions, relations, integrity | huge scale, flexible data, high write throughput |

## NoSQL types
- **Key-value** — Redis, DynamoDB (caching, sessions).
- **Document** — MongoDB, Couchbase (JSON documents).
- **Column-family** — Cassandra, HBase (wide, write-heavy, time-series).
- **Graph** — Neo4j (relationships, recommendations).

## Replication
Copies of data across nodes for **read scaling + high availability**.
- **Leader-follower (primary-replica)** — writes to leader, reads from replicas; async (lag) vs sync (slower).
- **Multi-leader / leaderless** (Dynamo-style) — higher availability, conflict resolution needed.
- Failover: promote a replica when the leader dies.

## Partitioning / Sharding
Split data across nodes for **write scaling** + larger datasets.
- **Horizontal (sharding)** — rows split by a **shard key** (range, hash, or directory-based).
- **Vertical** — split columns/tables by access pattern.
- **Consistent hashing** — minimizes reshuffling when nodes are added/removed.
- Challenges: cross-shard joins/transactions, hotspots (bad shard key), rebalancing.

## Consistency (CAP applied)
- Under a partition, choose **CP** (reject to stay consistent) or **AP** (serve, reconcile later).
- **Quorum** reads/writes (`R + W > N`) tune the consistency/availability balance.

## Caching in front of the DB
Cache hot reads (Redis) to offload the database — see [Caching](../distributed-cloud/caching.md).
