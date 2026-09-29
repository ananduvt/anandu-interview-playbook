# Backlog — Topics to Add

Pending topics to write up, grouped by area. Move each into its proper section once fleshed out.

## Java — language & core
- Basic CLI: `java`, `javac`, folder layout (`bin`, `build`, `META-INF`, `resources`)
- `var` vs explicit type; pass-by-value vs "reference"; bitwise operations
- Conversions: list ↔ array ↔ stream, int ↔ String
- `hashCode` collisions; internal working of `Set`
- Singleton & Builder patterns; thread-safe singleton (double-checked locking + breaking it)
- Records, marker annotations, Lombok annotations
- `AtomicReference`; JVM args and their uses
- `yield` / switch expressions ([baeldung](https://www.baeldung.com/java-yield-switch))

## Concurrency
- Executor / asynchronous calls; thread pool; connection pool
- `synchronized` vs `ConcurrentHashMap`
- Handling transactions without `@Transactional`

## Spring / Web
- `@SpringBootApplication`, `@Configuration`, dependency injection
- `@ControllerAdvice` exception handling
- Spring Security config for role-based login; securing an API
- API versioning; setting HTTP `202 Accepted`
- HTTP methods (GET/PUT/POST/DELETE); `PATCH` vs `PUT`
- Thymeleaf; Java FTL (FreeMarker) processing

## SQL / Databases
- Joins & `UNION`; `HAVING` vs `WHERE`
- Indexing; query performance; `UPSERT`; sharding
- Redis cache ([heap DS ref](https://www.geeksforgeeks.org/heap-data-structure/))

## Distributed / Microservices / DevOps
- API gateway, load balancer, sidecar pattern
- Service discovery (Eureka/Consul)
- Kubernetes basics; SonarQube without the IDE
- gRPC / RPC calls
- Git: merge vs rebase vs stash; amend author (`git commit --amend --author="Name <email>"`)
- Common environments: dev / UAT / prod

## Coding exercises to add
- Print 1–100 without loops
- Sort students by marks and assign rank
- Java `Dataset`; regex practice

## Interview question sets to expand
- IBM questions: Java collections, threads, synchronization, Spring Boot, microservices, Kafka
