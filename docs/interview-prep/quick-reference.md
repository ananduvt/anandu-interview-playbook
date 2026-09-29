# Quick Reference

Last-minute rapid recall.

## Design checklist (any system-design prompt)
Clarify → estimate scale → API → components → data/storage → scale (LB, cache, async) → resilience
(timeout/retry/circuit-breaker/bulkhead) → trade-offs.

## Resilience
Timeout · Retry (idempotent + backoff+jitter) · Circuit breaker · Bulkhead · Fallback/graceful degradation.

## Caching
cache-aside/read-through · TTL + LRU · local (per-instance) vs distributed (Redis) · invalidation + stampede.

## REST
Verbs & idempotency (GET/PUT/DELETE idempotent, POST not) · status 400/401/403/404/409/429/5xx · URI versioning ·
pagination (offset vs cursor) · statelessness · mask sensitive data.

## Java quick hits
== vs equals · equals+hashCode · HashMap (buckets, 0.75, treeify@8) · ConcurrentHashMap vs synchronizedMap ·
streams lazy+single-use · Optional (no `.get`) · records = immutable DTOs · constructor injection ·
singleton beans stateless · @Transactional rolls back on unchecked.

## Concurrency quick hits
volatile=visibility · synchronized/Lock=mutual exclusion · Atomic=CAS · deadlock 4 conditions · ThreadLocal
confinement · immutability = free thread safety · virtual threads (Java 21).

## Theory
ACID vs BASE · CAP (C vs A under partition) · SOLID/DRY/KISS/YAGNI · idempotency · statelessness ·
at-least-once vs exactly-once.

## Coding-round process
Clarify → example → brute force + Big-O → optimize → clean code → test edges → state complexity.
Patterns: hashmap · two-pointer · sliding window · binary search · BFS/DFS · heap/top-k · DP basics · LRU cache.

## In the room
Think out loud · state trade-offs · lead with impact then how · quantify · "I don't know, here's how I'd find out".
