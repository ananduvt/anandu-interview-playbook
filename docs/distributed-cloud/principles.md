# Microservices Principles

- **Single Responsibility** — each service owns one business capability.
- **API Gateway** — a single entry point that routes, aggregates, and cross-cuts (auth, rate limiting) requests.
- **Loose Coupling & High Cohesion** — services are independent but internally focused.
- **Statelessness** — services don't hold session state; externalize it (e.g. Redis) so any instance can serve any request.
- **Event-Driven Architecture** — asynchronous communication via Kafka / RabbitMQ for decoupling and resilience.
- **Service Discovery** — dynamic registration/lookup (Eureka, Consul) instead of hard-coded endpoints.
- **Database per service** — each service owns its data; avoid the shared-database anti-pattern.

## Trade-offs to mention
- Independent deploy/scale and fault isolation **vs** distributed-system complexity, network failures, and
  eventual consistency.
- Prefer async messaging for decoupling; prefer sync REST for simple request/response where latency coupling is acceptable.

## Related
- Resilience patterns (timeout, retry, circuit breaker, bulkhead) → see [Scalability & Resilience](../system-design/scalability-resilience.md).
- Messaging systems (Kafka vs RabbitMQ) → see [Messaging](messaging.md).
