# Backlog — Topics to Add

Pending topics to write up, grouped by new structure. Move each into its proper section once fleshed out.

## Java Core
- Basic CLI: `java`, `javac`, folder layout (`bin`, `build`, `META-INF`, `resources`)
- `var` vs explicit type; pass-by-value deep-dive; bitwise operations
- Conversions: list ↔ array ↔ stream, int ↔ String
- Records, marker annotations
- `yield` / switch expressions ([baeldung](https://www.baeldung.com/java-yield-switch))

## Java Collections & Concurrency
- `hashCode` collisions; internal working of `Set`
- Executor / asynchronous calls; thread pool; connection pool
- `synchronized` vs `ConcurrentHashMap`
- `AtomicReference`; handling transactions without `@Transactional`

## Java Patterns & Advanced
- Singleton & Builder patterns; thread-safe singleton (double-checked locking)
- Lombok annotations
- ✅ **DONE**: Garbage Collection (G1, tuning, diagnostics)
- ✅ **DONE**: Performance tuning (memory, caching, DB optimization, thread pools, profiling)
- Reflection basics; annotation processing

## Spring & Web
- ✅ **DONE**: Core annotations (`@SpringBootApplication`, `@Configuration`, stereotypes, injection)
- ✅ **DONE**: Common annotations (40+ with examples: request handling, config, transactions, async)
- ✅ **DONE**: Exception handling with `@ControllerAdvice`
- ✅ **DONE**: Transaction management and profiles
- Spring Security config for role-based login; securing an API
- API versioning; setting HTTP `202 Accepted`
- HTTP methods deep-dive (GET/PUT/POST/DELETE); `PATCH` vs `PUT`
- Thymeleaf; Java FTL (FreeMarker) processing
- Spring WebFlux & reactive programming
- Spring Boot Actuator & health checks

## Databases
- Joins & `UNION`; `HAVING` vs `WHERE`
- Indexing; query performance; `UPSERT`; sharding
- Redis cache basics ([heap DS ref](https://www.geeksforgeeks.org/heap-data-structure/))

## Microservices & Distributed Systems
- ✅ **DONE**: Resilience patterns (Circuit Breaker, Retry, Timeout, Bulkhead)
- ✅ **DONE**: Communication patterns (Saga choreography, API Gateway, Service Discovery, Load Balancer)
- ✅ **DONE**: Data patterns (Database-Per-Service, Event Sourcing, CQRS)
- ✅ **DONE**: Deployment patterns (Blue-Green, Canary, Rolling, Shadow)
- gRPC / RPC calls
- Distributed consensus & eventual consistency
- Sidecar pattern

## Cloud Services
- Infrastructure as Code (Terraform, CloudFormation)
- Multi-cloud & hybrid cloud patterns
- Cost optimization
- Disaster recovery & backup strategies

## DevOps
- Git: merge vs rebase vs stash; amend author (`git commit --amend --author="Name <email>"`)
- Common environments: dev / UAT / prod
- SonarQube (without IDE integration)

## Coding Exercises
- Print 1–100 without loops
- Sort students by marks and assign rank
- Java Dataset operations; regex practice

## Interview Question Sets
- IBM questions: Java collections, threads, synchronization, Spring Boot, microservices, Kafka
