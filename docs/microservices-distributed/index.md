# Microservices & Distributed Systems

Architect scalable, resilient microservices systems: design patterns, service communication, data consistency, and deployment strategies.

---

## Topics

### [Principles & Patterns](principles.md)
**Microservices - Service Boundaries - Resilience Patterns - Communication Patterns - Data Patterns - Deployment Patterns**  
Core patterns: Circuit Breaker, Retry, Timeout, Bulkhead for resilience; Saga choreography for distributed transactions; API Gateway for routing; Service Discovery; Database-Per-Service; Event Sourcing; CQRS; Blue-Green/Canary/Rolling deployments.

### [Spring Cloud Implementation](spring-cloud.md)
**Spring Cloud - Eureka - Config Server - Ribbon - Hystrix - Zuul - Spring Cloud Gateway**  
Spring Cloud ecosystem: service discovery with Eureka, centralized configuration, client-side load balancing, circuit breakers, API gateway routing, distributed tracing.

### [Service Discovery](service-discovery.md)
**Service Registration - Dynamic Discovery - Eureka - Consul - Health Checks - Load Balancing**  
Dynamic service location: client-side vs. server-side discovery, Eureka registration/discovery, health checks and heartbeats, combining with circuit breakers, failure handling.

### [Messaging (Kafka)](messaging.md)
**Event Streaming - Kafka - Topics - Partitions - Consumers - Producers - Event-Driven Architecture**  
Message-driven systems: Kafka architecture (broker, topic, partition, consumer group), event sourcing, asynchronous communication, failure handling, exactly-once semantics.

---

## Quick Interview Prep

**Microservices design principles:**
- **Bounded Contexts**: Each service owns its data; no direct database access across services
- **Resilience Patterns**: Circuit Breaker (fail fast), Retry (exponential backoff), Timeout (fail safe), Bulkhead (resource isolation)
- **Event-Driven**: Services communicate via events (Kafka topics); eventual consistency model
- **Service Discovery**: Dynamic registration (Eureka, Consul); health checks; load balancing

**Distributed transaction handling:**
- **Saga Choreography**: Services publish events; other services react (decoupled, complex to debug)
- **Saga Orchestration**: Central orchestrator coordinates steps (simpler flow, central coordination)
- **Compensation**: Rollback mechanisms for failed transactions

**Deployment strategies:**
- **Blue-Green**: Two identical environments; instant cutover; zero downtime; 2x cost
- **Canary**: Gradual traffic shift (5% → 50% → 100%); monitor metrics before full shift
- **Rolling**: Incremental replacement; slow rollout; gradual resource increase; standard in Kubernetes
- **Shadow**: Mirror traffic to new version; validate before routing real traffic

**Kafka essentials:**
- **Topics**: Logical feed of records; immutable event log
- **Partitions**: Parallelism; each partition ordered; consumer group reads from multiple partitions
- **Exactly-once**: Idempotent producers + transactional consumers
