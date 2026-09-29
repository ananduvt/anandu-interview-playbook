# Scalability & Resilience

## Resilience patterns
- **Timeout** — never wait unbounded on a downstream.
- **Retry** — only for idempotent + transient failures; add exponential backoff + jitter; cap attempts.
- **Circuit breaker** — stop calling a failing service; fail fast; half-open probe to recover.
- **Bulkhead** — isolate resources (thread pools) so one slow dependency can't starve others.
- **Fallback / graceful degradation** — return cached/partial/empty instead of failing the whole request.
- **Rate limiting / throttling** — protect services from overload.

## Failover mechanisms
- **Active-passive** — standby takes over on primary failure (failover time).
- **Active-active** — all nodes serve traffic; load-balanced; better utilization + instant failover.
- **Health checks** — LB/orchestrator removes unhealthy instances.
- **Replication** — data copies across nodes/regions for durability + read scaling.
- **Leader election** — (ZooKeeper/Raft) pick a coordinator; re-elect on failure.

## High availability
- Eliminate single points of failure (redundancy at every tier).
- Multi-AZ / multi-region deployment; automated failover; backups + tested restores.
- **SLA / SLO / SLI**; measure availability in "nines" (99.9% ≈ 8.7h/yr downtime).

## Java / Spring
- **Resilience4j** (successor to Hystrix) — circuit breaker, retry, rate limiter, bulkhead, time limiter as
  composable decorators/annotations.
- Idempotent handlers + safe retries; unwrap wrapped async exceptions.
