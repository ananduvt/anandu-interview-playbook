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
- JVM args and their uses; profiling & performance tuning
- Reflection basics; annotation processing

## Spring & Web
- `@SpringBootApplication`, `@Configuration`, dependency injection deep-dive
- `@ControllerAdvice` exception handling
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
- API gateway, load balancer, sidecar pattern
- Service discovery (Eureka/Consul)
- gRPC / RPC calls
- Distributed consensus & eventual consistency
- Saga pattern for distributed transactions
- Circuit breaker & resilience patterns

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
