# Spring Cloud & Microservices

Building distributed systems with Spring. Complements [Microservices Principles](../distributed-cloud/principles.md).

## Service discovery
Services register and look each other up dynamically (no hard-coded hosts).
- **Eureka** (client-side discovery) or Consul; clients resolve instances and load-balance
  (**Spring Cloud LoadBalancer**, formerly Ribbon).

## Centralized config
- **Spring Cloud Config Server** serves config from a git repo; clients refresh via `/actuator/refresh` or
  Spring Cloud Bus. Externalizes config per environment.

## API gateway
- **Spring Cloud Gateway** — routing, filters (auth, rate limiting), circuit breaking at the edge.

## Resilience
- **Resilience4j** (Hystrix successor) — circuit breaker, retry, bulkhead, rate limiter, time limiter as
  annotations/decorators. See [Scalability & Resilience](../system-design/scalability-resilience.md).

## Distributed tracing
- **Micrometer Tracing** (+ OpenTelemetry/Zipkin) propagates trace ids across services → see [Observability](../devops/observability.md).

## Inter-service communication
- Sync: REST (`RestClient`/`WebClient`/OpenFeign) with client-side load balancing.
- Async: **Kafka/RabbitMQ** via Spring Cloud Stream for event-driven decoupling.

## Data patterns
- **Database per service**; keep data private.
- **Saga** for distributed transactions (choreography via events, or orchestration).
- **Outbox pattern** for reliable event publishing.
- **CQRS** to separate read/write models.

## Config vs code
Prefer configuration + conventions; keep services stateless so they scale horizontally.
