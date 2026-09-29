# Caching

## Why cache
Reduce latency and load on databases/downstreams by keeping frequently-read data close.

## Patterns
- **Cache-aside (lazy)** — app checks cache; on miss, loads from DB and populates. Most common.
- **Read-through** — cache library loads from DB on miss.
- **Write-through** — write to cache + DB synchronously (consistent, slower writes).
- **Write-behind** — write to cache, async flush to DB (fast, risk of loss).

## Eviction policies
**LRU** (least recently used), LFU, FIFO, **TTL** (time-based). Bound size to avoid memory pressure.

## Local vs distributed
| | Local (in-JVM: EhCache, Caffeine) | Distributed (Redis, Memcached) |
|--|-----------------------------------|-------------------------------|
| Speed | fastest (no network) | network hop |
| Scope | per instance (coherence issue) | shared across instances |
| Capacity | heap-bound | large, independent |
| Survives restart | no | yes |

## Invalidation ("one of the two hard things")
TTL expiry, explicit eviction on write, or an explicit refresh endpoint. Beware **stale data** and
**cache stampede / thundering herd** on expiry → mitigate with TTL jitter, locks, or request coalescing.

## Redis
In-memory data store; strings/hashes/lists/sets/sorted-sets; TTL; pub/sub; used as distributed cache,
session store, rate limiter, leaderboard.

## Spring
`spring-boot-starter-cache` + `@Cacheable/@CachePut/@CacheEvict` over a provider (Caffeine, EhCache, Redis).
