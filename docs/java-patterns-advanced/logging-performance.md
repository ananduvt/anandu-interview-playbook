# Logging

## Landscape
- **SLF4J** — logging facade (API). Code against it, bind an implementation at runtime.
- **Implementations**: **Logback** (default in Spring Boot), **Log4j2**, `java.util.logging`.
- `log4j-over-slf4j` / bridges route legacy logging through SLF4J.

## Levels
`TRACE < DEBUG < INFO < WARN < ERROR`. Set per-package; raise to DEBUG only when diagnosing.

## Best practices
- Use parameterized logging (avoids string building when disabled):
  ```java
  log.debug("user {} did {}", userId, action);
  ```
- **Correlation id** via **MDC** (`MDC.put("traceId", id)`) to trace a request across components/threads.
- **Never log secrets/PII** — mask sensitive fields (card numbers, tokens, emails).
- Structured/JSON logging for aggregation (ELK, Splunk, OpenSearch).
- Async appenders for throughput; don't log in tight loops.

## Spring Boot
- `application.yml` sets levels; Logback config in `logback-spring.xml`.
- Runtime level changes via Actuator `/loggers` endpoint.

## Performance Tuning

### Memory & Heap Management
- **Heap size**: Set `-Xms` (initial) and `-Xmx` (max) equal to avoid resizing overhead
  ```bash
  java -Xms4G -Xmx4G -XX:+UseG1GC MyApp
  ```
- **String interning**: Reduce memory for duplicate strings (small pools help)
- **Object pooling**: Reuse expensive objects (connection pools, thread pools)
- **Garbage collection**: See [Advanced Topics](advanced.md#garbage-collection-gc) for GC tuning

### Application Performance

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| High latency | Blocking I/O, slow DB queries, synchronous calls | Use async/non-blocking (WebFlux, Spring Cloud), optimize queries, cache |
| High CPU | Tight loops, excessive logging, GC thrashing | Profile with JProfiler; reduce log level; increase heap; tune GC |
| High memory | Memory leak, unbounded collections, large payloads | Heap dump analysis; use pagination; compress data |
| Network bottleneck | Chatty interfaces, large payloads, uncompressed responses | Batch requests, compress JSON, use binary protocols (gRPC) |

### Database Performance
- **N+1 problem**: Query in loop → many DB calls. Fix: Join fetch or batch load
  ```java
  // ❌ Bad: 1 query for user + N queries for orders
  List<User> users = userRepo.findAll();
  users.forEach(u -> u.getOrders());  // N+1 queries!
  
  // ✅ Good: Single query with join
  List<User> users = userRepo.findAllWithOrders();  // JPQL: LEFT JOIN FETCH
  ```
- **Query optimization**: Add indexes on frequently filtered columns
  ```sql
  CREATE INDEX idx_user_email ON users(email);
  ```
- **Connection pooling**: Limit DB connections to prevent exhaustion
  ```yaml
  spring:
    datasource:
      hikari:
        maximum-pool-size: 10
        minimum-idle: 5
  ```

### Caching Strategies
- **Cache-aside (Lazy Loading)**: App checks cache, misses → query DB, store in cache
- **Write-through**: Write to cache + DB synchronously (consistent, slower)
- **Write-behind**: Write to cache, async write to DB (fast, eventual consistency risk)
- Tools: Redis, Memcached, Caffeine (in-process)

```java
@Cacheable(value = "users", key = "#id")  // Cache result by id
public User getUser(Long id) {
    return userRepo.findById(id).orElse(null);
}

@CachePut(value = "users", key = "#result.id")  // Update cache
public User updateUser(User user) {
    return userRepo.save(user);
}

@CacheEvict(value = "users", key = "#id")  // Invalidate cache
public void deleteUser(Long id) {
    userRepo.deleteById(id);
}
```

### Thread Pool Tuning
- **Core threads**: Always running; `spring.task.execution.pool.core-size`
- **Max threads**: Upper limit; `spring.task.execution.pool.max-size`
- **Queue depth**: Pending tasks; `spring.task.execution.pool.queue-capacity`
- Too few threads: Bottleneck; too many: Context switch overhead
  ```yaml
  spring:
    task:
      execution:
        pool:
          core-size: 5
          max-size: 20
          queue-capacity: 100
      scheduling:
        pool:
          size: 5
  ```

### Profiling Tools
- **JProfiler**: CPU sampling, memory snapshots, hotspot identification
- **JFR (Java Flight Recorder)**: Built-in, low-overhead recording
  ```bash
  java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder MyApp
  jcmd <pid> JFR.start duration=60s filename=profile.jfr
  ```
- **jstat**: Real-time GC, class loading, compilation stats
  ```bash
  jstat -gc -h10 <pid> 1s  # GC every 1 second
  ```
