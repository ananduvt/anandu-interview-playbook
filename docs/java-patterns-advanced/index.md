# Java Patterns & Advanced

Deep-dive into design patterns, performance tuning, garbage collection, logging, and advanced JVM concepts for production systems.

---

## Topics

### [Design Patterns](design-patterns.md)
**Creational - Structural - Behavioral - Factory - Singleton - Observer - Strategy - Adapter - Decorator**  
Gang of Four patterns: when to use each, trade-offs, real-world examples in Java frameworks (Spring, Hibernate), anti-patterns to avoid, implementation best practices.

### [Logging & Performance](logging-performance.md)
**SLF4J - Logback - Log4j2 - Structured Logging - Performance Tuning - Caching - Thread Pools - Profiling**  
Production concerns: logging best practices, avoiding performance pitfalls, database query optimization (N+1 problem), connection pooling, caching strategies, thread pool configuration.

### [Advanced Topics](advanced.md)
**Garbage Collection - GC Tuning - JVM Diagnostics - Memory Management - Profiling Tools - G1GC - ZGC**  
JVM internals: garbage collection algorithms (Serial, Parallel, CMS, G1, ZGC, Shenandoah), pause time tuning, heap analysis, memory leaks, JProfiler/JFR tools.

---

## Quick Interview Prep

**Design Patterns to know:**
- **Creational**: Singleton (thread-safe), Factory (object creation), Builder (fluent API)
- **Structural**: Adapter (interface conversion), Decorator (runtime behavior), Proxy (controlled access)
- **Behavioral**: Observer (event-driven), Strategy (algorithm selection), Command (action encapsulation)

**Performance tuning essentials:**
- **GC tuning**: -Xms/-Xmx equal, -XX:+UseG1GC, -XX:MaxGCPauseMillis, -XX:InitiatingHeapOccupancyPercent
- **Caching**: Cache-aside, write-through, write-behind strategies; @Cacheable, @CachePut, @CacheEvict
- **Database**: N+1 query detection, JOIN FETCH, indexing on filtered columns, connection pool limits
- **Logging**: Parameterized logging (avoids string building), async appenders, structured JSON logging

**Profiling & diagnostics:**
- **jstat**: Real-time GC stats (`jstat -gc -h10 <pid> 1s`)
- **JFR**: Java Flight Recorder for low-overhead profiling
- **Heap dumps**: jmap analysis to detect memory leaks
