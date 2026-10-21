# Java Versions & Features

## Java Migration: 8 → 11 → 17 → 21

**Migration strategy**: Don't jump directly; go incrementally: 8 → 11 (easiest) → 17 (moderate) → 21 (modern).

### Java 8 → Java 11 (Easiest Migration)

**Effort:** Low | **Breaking changes:** Few

```
What breaks:
- Java EE modules removed (CORBA, XML, RMI) → Use separate libraries
- Nashorn engine removed (Java 15) → Use GraalVM if needed
- deprecated APIs → Warnings in logs

What works out of the box:
- Lambdas/Streams: Already in 8, just maintained
- Type inference (`var`): Optional, can gradually adopt
- Thread model: Same OS threads
```

**Checklist:**
```
[ ] Review dependencies (java.xml, jakarta, etc.)
[ ] Test with Java 11 JDK (compile + run)
[ ] Update build tools (Maven 3.6.3+, Gradle 5.6+)
[ ] Replace URLConnection with modern HTTP client
[ ] Check for Nashorn usage (migrate to GraalVM or scripting alternatives)
[ ] Test in staging: Full integration/performance tests
```

**Example fixes:**
```java
// OLD (Java 8)
String response = IOUtils.toString(new URL(url).openStream());

// NEW (Java 11+)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder(URI.create(url)).build();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

### Java 11 → Java 17 (Moderate Migration)

**Effort:** Moderate | **Breaking changes:** Some (careful review needed)

```
What breaks:
- Strong encapsulation: --illegal-access=deny (default Java 17)
  - Can't access sun.misc.Unsafe, internal APIs
- Removed features: Applet API, deprecated RMI APIs
- Type system: Sealed classes & records (optional, but recommended)
- Text blocks: Now standard (no longer preview)

What's new:
- Pattern matching (instanceof): Cleaner type checks
- Records: Immutable data holders (replaces boilerplate)
- Sealed classes: API boundaries, exhaustive pattern matching
```

**Checklist:**
```
[ ] Scan for sun.* API usage (use tools: jdeps, indy)
  Command: jdeps -jdkinternals app.jar
[ ] Migrate boilerplate classes to Records (if applicable)
[ ] Update sealed hierarchies (if designing new types)
[ ] Use pattern matching (instanceof) in place of casts
[ ] Test --illegal-access=deny (simulate Java 17 behavior)
[ ] Verify no reflection on JDK internals
[ ] Update build tools (Maven 3.8.1+, Gradle 7.0+)
```

**Example fixes:**
```java
// OLD (Java 11)
public class User {
    private String name;
    private int age;
    
    public User(String name, int age) { ... }
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { ... }
    @Override public String toString() { ... }
}

// NEW (Java 17 - Records)
public record User(String name, int age) { }
// Auto-generates: constructor, equals, hashCode, toString, accessors

// Usage:
if (obj instanceof User(String name, int age)) {  // Pattern matching
    System.out.println(name);
}
```

### Java 17 → Java 21 (Modern Adoption)

**Effort:** Low-Moderate | **Breaking changes:** None

```
What's new:
- Virtual threads: Millions of lightweight tasks (game-changer for I/O)
- Record patterns: Destructure in pattern matches
- Switch patterns: Exhaustive, type-safe switch
- Sequenced collections: Guaranteed order API

What breaks:
- Nothing! Full backward compatibility
```

**Checklist:**
```
[ ] Evaluate virtual threads for I/O-bound workloads
[ ] Update thread pools to virtual threads where beneficial
[ ] Test with newer GC options (ZGC if latency-critical)
[ ] Adopt pattern matching for switch (optional, but cleaner)
[ ] Benchmark virtual threads vs traditional threads
[ ] Update CI/CD Java version
```

**Example: Virtual threads adoption**
```java
// OLD (Java 17 - OS threads)
ExecutorService executor = Executors.newFixedThreadPool(100);
for (int i = 0; i < 10000; i++) {
    executor.submit(() -> blockingIO());  // 10k tasks queued (bottleneck)
}

// NEW (Java 21 - Virtual threads)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1000000; i++) {
        executor.submit(() -> blockingIO());  // 1M tasks, scales effortlessly
    }
}
```

### Migration Path Decision Table

| Starting Point | Target | Effort | Timeline | Risk |
|---|---|---|---|---|
| Java 8 | Java 11 | Low | 2-4 weeks | Low |
| Java 8 | Java 17 | Moderate | 4-8 weeks | Moderate |
| Java 8 | Java 21 | Moderate | 6-10 weeks | Moderate |
| Java 11 | Java 17 | Low-Moderate | 2-4 weeks | Low |
| Java 11 | Java 21 | Low | 1-2 weeks | Low |
| Java 17 | Java 21 | Minimal | < 1 week | Very Low |

### Best Practices for Migration

1. **Use jdeps** (Java Dependency Analyzer)
   ```bash
   jdeps -jdkinternals myapp.jar  # Find internal API usage
   ```

2. **Gradual adoption** (don't do all at once)
   - Compile with new version
   - Run integration tests
   - Verify in staging
   - Gradual prod rollout (10% → 50% → 100%)

3. **Keep fallback** (if critical)
   - Keep Java 8 running for 2-4 weeks
   - Easy rollback if issues arise

4. **Version-specific compiler flags**
   ```xml
   <!-- Maven: Support multiple JDK versions during build -->
   <properties>
       <maven.compiler.release>11</maven.compiler.release>
   </properties>
   ```

5. **Test automation**
   - Run full test suite with new Java version
   - Performance benchmarks (compare latency)
   - Load tests (especially for GC changes)

### Troubleshooting Common Migration Issues

```
Error: "java.lang.NoSuchMethodError: java.lang.String.stripIndent()"
Fix: Code written for Java 15+ text blocks on Java 11 JVM
Solution: Check --source/--target compiler flags, recompile

Error: "WARNING: An illegal reflective access operation has occurred"
Fix: Code accessing sun.misc.Unsafe or internal APIs (Java 9+)
Solution: Use public APIs instead (VarHandle, MethodHandles)

Error: "Class not found: javax.xml.bind.JAXBContext"
Fix: Java EE modules removed in Java 11
Solution: Add dependency: <groupId>javax.xml.bind</groupId><artifactId>jaxb-api</artifactId>

Error: "java.lang.module.FindException: Module not found"
Fix: Module system changes in Java 9+
Solution: Add --add-modules java.base, or use --illegal-access flag temporarily
```

---

# Java Versions & Features

## Long-Term Support (LTS) Overview

**LTS Releases** receive **extended support** for 5+ years (sometimes 10+ years for Java 8 and 11):
- **Java 8** (March 2014): Extended support until December 2030
- **Java 11** (September 2018): Extended support until September 2026
- **Java 17** (September 2021): Extended support until September 2029
- **Java 21** (September 2023): Extended support until September 2031

**Non-LTS Releases** receive ~6 months of support (overlap period only). Since Java 9, a new version ships every 6 months; LTS versions are chosen every ~2–3 years.

---

Since Java 9, releases ship every 6 months; **LTS** releases (long-term support) land every ~2–3 years
(8, 11, 17, 21). Know the LTS versions and the headline features per release.

| Version | Year | Headline features |
|---------|------|-------------------|
| 1.0 | 1996 | Initial release: classes, objects, inheritance, interfaces, JVM |
| 1.1 | 1997 | Inner classes, JDBC, improved AWT/event handling |
| 1.2 (J2SE) | 1998 | **Collections Framework**, Swing, JNDI |
| 1.3 | 2000 | HotSpot JVM, Java Sound API |
| 1.4 | 2002 | `assert`, regular expressions, NIO |
| 5 | 2004 | **Generics**, enums, autoboxing, enhanced for-loop, **annotations** |
| 6 | 2006 | Performance, scripting (Compiler API), JAXB/web services |
| 7 | 2011 | try-with-resources, diamond `<>`, multi-catch |
| 8 | 2014 | **Lambdas, Stream API, `java.time`**, default methods (LTS) |
| 9 | 2017 | Module system (Jigsaw), HTTP/2 client (incubator), JShell |
| 10 | 2018 | `var` (local-variable type inference) |
| 11 | 2018 | Standard **HTTP client**, run single-file source, removed Java EE (LTS) |
| 12–13 | 2019 | switch expressions (preview), text blocks (preview) |
| 14 | 2020 | records (preview), pattern matching for `instanceof` (preview), helpful NPEs |
| 15 | 2020 | text blocks (standard), sealed classes (preview) |
| 16 | 2021 | records (standard), pattern matching for `instanceof` (standard) |
| 17 | 2021 | **sealed classes (standard)** (LTS) |
| 18 | 2022 | UTF-8 by default, `@Deprecated` refinements |
| 19 | 2022 | Record patterns (preview), pattern matching for `switch` (preview) |
| 20 | 2023 | Scoped values (preview), virtual threads (preview), structured concurrency (preview) |
| 21 | 2023 | **Virtual threads (Loom)**, record patterns, pattern matching for `switch`, sequenced collections (LTS) |
| 22 | 2024 | Foreign Function & Memory API (3rd preview), unnamed variables for pattern matching (preview) |
| 23 | 2024 | Flexible constructor bodies (preview), primitive types in patterns (preview) |
| 24 | 2025 | Primitive types in patterns (2nd preview), flexible constructor bodies (2nd preview) |

---

## Quick Version Comparison (LTS Releases)

Quick reference to understand differences between major LTS versions:

| Aspect | Java 8 (2014) | Java 11 (2018) | Java 17 (2021) | Java 21 (2023) |
|--------|--------|---------|---------|---------|
| **Support Until** | Dec 2030 | Sept 2026 | Sept 2029 | Sept 2031 |
| **Key Features** | Lambdas, Streams, functional interfaces, default methods, java.time, Optional, method references | HTTP/2 client, `var` type inference, clean JDK (Java EE removed), ZGC, single-file execution | Sealed classes, Records (standard), pattern matching for instanceof, Foreign Function & Memory API, strong encapsulation | Virtual threads (Loom), record patterns, pattern matching for switch (standard), sequenced collections, structured concurrency |
| **Concurrency Model** | OS threads (heavy) | OS threads | OS threads | Virtual threads (lightweight) |
| **Thread Scalability** | ~1,000 threads | ~1,000 threads | ~1,000 threads | Millions of threads |
| **Type Safety** | Basic generics | Generics + var | Sealed classes, pattern matching | Advanced patterns, exhaustive matching |
| **Functional Support** | Lambdas, streams, Optional | Mature API | Mature + pattern matching | Pattern matching for switch |
| **Collections** | Basic (List, Set, Map) | Stable | Stable | Sequenced Collections (order guarantees) |
| **Build Adoption** | ⭐⭐⭐ (ubiquitous) | ⭐⭐⭐ (very high) | ⭐⭐ (growing) | ⭐ (emerging) |
| **Enterprise Use** | ✅ Standard | ✅ Very common | ✅ Growing adoption | ⭐ Early adoption |
| **Best For** | Legacy systems, broad compatibility | Modern systems, standard choice | New greenfield projects, microservices | High-concurrency I/O-bound apps |
| **When NOT to Use** | Performance-critical systems (no streams) | Very old frameworks | Complex inheritance hierarchies (sealed classes) | CPU-bound workloads (OS threads better) |

### **Quick Decision Guide**

- **Stuck on Java 8?** → Migration path: 8 → 11 (low cost) → 17 (moderate cost)
- **Starting new project?** → Use **Java 21 LTS** (virtual threads + patterns)
- **Legacy enterprise?** → Java 11 LTS (good balance of stability + modern features)
- **High I/O workloads?** → Java 21 (virtual threads = game-changer)
- **Need sealed classes + exhaustive pattern matching?** → Java 17 minimum
- **Not using record patterns or virtual threads?** → Java 17 is sufficient

---

## LTS Version Comparison Table

Feature matrix showing what each LTS version introduced:

| Feature | Java 8 | Java 11 | Java 17 | Java 21 |
|---------|--------|---------|---------|---------|
| **Functional Programming** | ✅ Lambdas, Streams | | | |
| Functional Interfaces | ✅ `@FunctionalInterface` | | | |
| Default/Static Methods in Interfaces | ✅ New | | | |
| `java.time` API | ✅ Immutable date/time | | | |
| Optional<T> | ✅ Introduced | ✅ Mature | ✅ Mature | ✅ Mature |
| Method References | ✅ `ClassName::method` | | | |
| **Type Inference** | | ✅ `var` (from Java 10) | ✅ Enhanced | ✅ Enhanced |
| **HTTP Support** | | ✅ Standard HTTP/2 client | ✅ Stable | ✅ Stable |
| **Single-File Execution** | | ✅ `java HelloWorld.java` | ✅ Supported | ✅ Supported |
| **Records** | | | ✅ Standard | ✅ Mature |
| **Pattern Matching - instanceof** | | | ✅ Standard | ✅ Standard |
| **Sealed Classes** | | | ✅ New | ✅ Mature |
| **Pattern Matching - switch** | | | | ✅ Standard |
| **Record Patterns** | | | | ✅ Standard |
| **Virtual Threads** | | | | ✅ **Game-changer** |
| **Sequenced Collections** | | | | ✅ Order guarantees |
| **Structured Concurrency** | | | | ✅ Preview |
| **Garbage Collection** | OS threads, G1GC | ZGC option | G1GC default, ZGC option | Virtual threads, ZGC |
| **Module System** | | | ✅ Mature | ✅ Mature |
| **Strong Encapsulation** | | | ✅ Enforced (illegal-access denied) | ✅ Enforced |
| **JDK Removed Java EE** | | ✅ (CORBA, XML, RMI) | | |
| **Support Until** | Dec 2030 | Sept 2026 | Sept 2029 | Sept 2031 |

---

### **Feature Categories by LTS**

**Java 8 (2014):** Functional foundation
- Lambdas, Streams, Functional interfaces
- Default methods, java.time, Optional
- Focus: Making Java code more concise and expressive

**Java 11 (2018):** Modern essentials
- HTTP/2 client (modern web support)
- `var` for type inference (cleaner code)
- JDK cleanup (removed Java EE baggage)
- Focus: Production-ready, clean platform

**Java 17 (2021):** Type safety
- Records (immutable data classes)
- Sealed classes (restrict inheritance)
- Pattern matching for `instanceof` (exhaustive checking)
- Focus: Safer, more maintainable code

**Java 21 (2023):** Concurrency revolution
- **Virtual threads** (millions of concurrent tasks)
- Record patterns & switch pattern matching (advanced patterns)
- Sequenced collections (guaranteed order)
- Focus: Scalable, high-concurrency systems

---

## LTS Feature Deep-Dive

### Java 8 (2014) — Functional Paradigm Shift

**Why it matters**: Java 8 fundamentally changed how Java code is written, introducing functional programming as a first-class paradigm.

- **Lambdas** (`(x, y) -> x + y`): Anonymous functions enabling concise, expressive code
- **Streams API**: Functional collections processing; lazy evaluation and composability (`.map()`, `.filter()`, `.reduce()`)
- **Functional Interfaces** (`@FunctionalInterface`): Single abstract method; enables lambda usage
- **Default Methods** in interfaces: Non-breaking API evolution; allow interface implementations without abstract forcing
- **`java.time` API**: Immutable, thread-safe date/time; replaces outdated `Date`/`Calendar`
- **Optional<T>**: Expressive null-handling; chain operations with `.orElse()`, `.map()`, `.flatMap()`
- **Method References** (`ClassName::methodName`): Syntactic sugar for lambdas; improves readability

**Interview angle**: "What problems did Java 8 solve?" → Concise code, functional composition, null safety patterns.

---

### Java 11 (2018) — Modern JDK Essentials

**Why it matters**: Java 11 standardized practical improvements; removed Java EE and introduced HTTP/2 client.

- **HTTP/2 Client** (java.net.http): Modern, non-blocking alternative to URLConnection; reactive streams compatible
- **`var` (from Java 10)**: Local variable type inference; reduces boilerplate while maintaining strong typing
- **Remove Java EE modules**: Deprecated EE (CORBA, XML, RMI) moved to separate libraries; keeps JDK lean
- **Single-file source execution** (`java HelloWorld.java`): REPL-like experimentation without compilation
- **ZGC (Z Garbage Collector)**: Low-latency alternative to G1; sub-millisecond pause times (experimental)
- **Unified Logging**: Centralized JVM logging via `-Xlog` flags; powerful diagnostics
- **Nashorn JavaScript engine deprecation** (removed in 17): Signals migration toward GraalVM for polyglot needs

**Interview angle**: "What's special about Java 11?" → HTTP/2 support, cleaner JDK, var keyword, single-file execution.

---

### Java 17 (2021) — Pattern Matching & Sealed Classes

**Why it matters**: Java 17 brought sealed classes, improving type safety and exhaustiveness checking in pattern matching.

- **Sealed Classes** (`sealed class Shape permits Circle, Square`): Restrict inheritance; enable compiler to verify exhaustive pattern matches
- **Pattern Matching for `instanceof`** (standard):
  ```java
  if (obj instanceof String s) {
      System.out.println(s.toUpperCase());  // s already cast
  }
  ```
- **Records** (standard): Immutable data carriers; auto-generate `equals()`, `hashCode()`, `toString()`, constructor
- **Strong Encapsulation of JDK Internals**: `--illegal-access` default DENY; forces migration from `sun.misc.Unsafe` to official APIs
- **Foreign Function & Memory API** (Incubating): Safely call native code without JNI boilerplate
- **Removal of Applet API**: Signals end of browser-based Java deployment

**Interview angle**: "How do sealed classes improve code safety?" → Compiler-verified exhaustiveness, cleaner pattern matching, better API boundaries.

---

### Java 21 (2023) — Virtual Threads & Pattern Matching Extended

**Why it matters**: Java 21 introduced virtual threads (Project Loom), enabling millions of concurrent tasks with simpler code than OS threads.

- **Virtual Threads** (`Thread.ofVirtual().start(task)`): Lightweight, JVM-managed threads; 1000s–millions feasible on single machine
  ```java
  // Instead of threadPool.execute(task)
  Thread.ofVirtual().name("vt-", 1).start(() -> {
      // Blocks don't exhaust OS resources; scheduled automatically
      blockingIO();  // e.g., HTTP call
  });
  ```
- **Record Patterns**: Destructure records in pattern matches
  ```java
  if (obj instanceof Point(int x, int y)) { /* use x, y directly */ }
  ```
- **Pattern Matching for `switch`** (standard): Type-safe, exhaustive switch without casting
  ```java
  String result = switch (obj) {
      case Integer i when i > 0 -> "positive";
      case Integer i -> "non-positive";
      case String s -> "string: " + s;
      default -> "unknown";
  };
  ```
- **Sequenced Collections** (List, Set, LinkedHashMap order guarantees): Clear API for encounter order
- **Unnamed Variables & Patterns** (`var _`): Suppress unused parameter warnings elegantly
- **Structured Concurrency** (Preview): Simplify task coordination; automatic resource cleanup

**Interview angle**: "How do virtual threads change concurrency?" → Scale to millions; fewer threads per task; simpler async patterns without callbacks/reactive frameworks.

---

## Interview tips
- Know the **LTS** line: **8, 11, 17, 21**.
- Be ready to explain the marquee features: **8** (lambdas/streams), **11** (HTTP client, `var` from 10),
  **17** (sealed classes), **21** (**virtual threads**).
- Since Java 9, Oracle uses a **6-month cadence** with LTS every few years.
- Understand the **progression**: 8→11 (functional; JDK cleanup), 11→17 (pattern matching; sealed classes), 17→21 (virtual threads for scale).

---

## Garbage Collection (GC)

**Garbage Collection** automatically frees memory from objects no longer in use. Different GC algorithms optimize for latency, throughput, or memory.

### Memory Layout

```
JVM Heap:
┌─────────────────┐
│  Young Gen (1)  │ ← New objects created here (fast, frequent GC)
├─────────────────┤
│  Old Gen (2)    │ ← Long-lived objects (slow, infrequent GC)
├─────────────────┤
│  Metaspace (3)  │ ← Class metadata (not heap, off-heap)
└─────────────────┘

(1) Young: Minor GC frequent, fast
(2) Old: Major/Full GC rare, slow
(3) Metaspace: Class definitions, method data
```

### Generational Hypothesis

Most objects die young; few survive to old age. GC focuses on Young Generation.

```
Object lifecycle:
New Object → [Minor GC] → Old Gen (if survives) → [Full GC if needed] → Freed
```

### GC Algorithms Comparison

| GC | Latency | Throughput | Memory | Best For | Config |
|----|---------|-----------|---------|----|---------|
| **Serial** | High ❌ | Low ❌ | Low ✅ | Single-threaded, embedded | `-XX:+UseSerialGC` |
| **Parallel** | High ❌ | High ✅ | Low ✅ | Batch processing, high throughput | `-XX:+UseParallelGC` (default Java 8) |
| **G1GC** | Low ✅ | Medium ✅ | Medium | Heap > 4GB, consistent latency (default Java 9+) | `-XX:+UseG1GC` |
| **ZGC** | Very Low ✅✅ | Medium ✅ | High | Sub-ms pause times, latency-critical | `-XX:+UseZGC` (Java 11+) |
| **Epsilon** | Varies | N/A ⚠️ | Grows unbounded | Testing, benchmarking (no GC) | `-XX:+UseEpsilonGC` (Java 11+) |
| **Shenandoah** | Very Low ✅✅ | Medium | High | Consistent latency (RedHat) | `-XX:+UseShenandoahGC` |

### Serial GC (Legacy)

**Single-threaded; pauses entire application.**

```
GC Pause Timeline:
[Running] ███ [PAUSE] ░░░ [Running] ██ [PAUSE] ░░ [Running]
          Stop    GC      Resume   Stop   GC   Resume
```

- All threads stop; one thread does GC
- **Pause time:** 100ms–1000ms (long!)
- **Use:** Single-core systems, embedded, small heaps only
- **Config:** `-XX:+UseSerialGC`

### Parallel GC (Throughput-Focused)

**Multiple threads doing GC in parallel; optimized for batch workloads.**

```
GC Pause Timeline:
[Running] ███ [PAUSE - Parallel GC] ░░░░ [Running]
          Multiple threads collect garbage simultaneously
```

- All threads stop; multiple threads do GC (parallel)
- **Pause time:** 50ms–500ms
- **Throughput:** High (more time running vs paused)
- **Best for:** Batch processing, data pipelines, high throughput
- **Config:** `-XX:+UseParallelGC` (default Java 8)
- **Example:** `java -XX:+UseParallelGC -Xmx4G app.jar`

### G1GC (Generational + Regions)

**Divides heap into regions; garbage-first collection; balanced latency/throughput.**

```
Heap divided into ~2048 regions:
┌──┬──┬──┬──┬──┐
│E │S │O │E │F│  E=Eden (Young), O=Old, S=Survivor, F=Free
├──┼──┼──┼──┼──┤
│E │O │E │S │O│  Non-contiguous; flexible
└──┴──┴──┴──┴──┘
```

- Collects highest-garbage regions first (Garbage-First)
- Mixed collections: Young + concurrent with Old
- **Pause time:** 10ms–200ms (predictable)
- **Default:** Java 9+ (`-XX:+UseG1GC`)
- **Best for:** Large heaps (> 4GB), consistent latency
- **Stages:**
  1. Young collection (concurrent marking happens in background)
  2. Concurrent marking (marks live objects without stopping app)
  3. Mixed collection (Young + Old)
  4. Full GC (if mixed falls behind)

**Example config:**
```bash
java -XX:+UseG1GC -Xms4G -Xmx4G -XX:MaxGCPauseMillis=200 app.jar
# Max pause target: 200ms
```

**Tuning:**
```java
-XX:+UseG1GC                    # Enable G1GC
-XX:MaxGCPauseMillis=200        # Target max pause (ms)
-XX:G1HeapRegionSize=8          # Region size (1-32MB)
-XX:G1ReservePercent=10         # Reserve for humongous objects
```

### ZGC (Z Garbage Collector)

**Sub-millisecond pause times; concurrent collection; Java 11+.**

```
ZGC Timeline (Concurrent):
[Running Ops] [GC Mark] [Running Ops] [GC Relocate] [Running Ops]
                ↓                            ↓
              Concurrent                 Concurrent
            (Pauses < 1ms)             (Pauses < 1ms)
```

- **Pauses:** < 1ms consistently (even with TB heaps!)
- **Concurrent:** Marking & relocation happen while app runs
- **Trade-off:** Higher memory overhead, higher CPU (for concurrency)
- **Best for:** Latency-critical (trading floors, gaming, real-time)
- **Config:** `-XX:+UseZGC -Xmx<size>` (Java 11+)

**Example:**
```bash
java -XX:+UseZGC -Xms10G -Xmx10G app.jar
# Even with 10GB heap, GC pauses < 1ms
```

**When ZGC shines:**
```
Latency-critical app (e.g., stock trading):
- G1GC: 50ms pause → 50ms latency spike (bad!)
- ZGC: < 1ms pause → imperceptible to users ✅
```

**Limitations:**
- Requires >= 2GB heap
- Higher memory footprint
- Not default (opt-in)

### Epsilon GC (No-Op)

**For testing/benchmarking; does NO garbage collection (memory fills up).**

```
Timeline (Epsilon):
[Running] ███████████ [No GC!]
Heap fills until OutOfMemoryError
```

- Useful for: Benchmarking (isolate GC impact), latency measurement
- **Do NOT use in production**
- **Config:** `-XX:+UseEpsilonGC -Xmx10G` (Java 11+)

**Use case:**
```bash
# Measure how much GC impacts performance
java -XX:+UseEpsilonGC -Xmx50G benchmark.jar  # No GC baseline
java -XX:+UseG1GC -Xmx50G benchmark.jar       # With G1GC
# Compare latencies
```

### GC Metrics to Monitor

```java
// Enable GC logging
-Xlog:gc*:file=gc.log:time,level,tags:filecount=5,filesize=100M

// Parse logs
Tools: GCViewer, Universal GC Log Analyzer, JProfiler

Key Metrics:
- Young GC frequency (per minute)
- Young GC pause time (avg, max, p99)
- Full GC frequency (should be rare!)
- Heap usage over time
- GC throughput (% time paused)
```

### Choosing a GC

```
1. Is latency critical? (e.g., trading, gaming, real-time)
   → YES: Use ZGC (-XX:+UseZGC)
   → NO: Continue

2. Is throughput the priority? (e.g., batch processing)
   → YES: Use Parallel GC (-XX:+UseParallelGC)
   → NO: Continue

3. Is heap size > 4GB?
   → YES: Use G1GC (-XX:+UseG1GC)
   → NO: Use Parallel or Serial (smaller heap = simpler)

4. For production systems:
   Default to G1GC (Java 9+): balanced, predictable
```

### Java 8 vs Java 11+ GC Defaults

| Version | Default GC | Comment |
|---------|-----------|---------|
| Java 8  | Parallel GC | High throughput, higher pause |
| Java 11 | G1GC | Balanced latency/throughput |
| Java 17 | G1GC | Same |
| Java 21 | G1GC | Same (with virtual threads, different scaling model) |

### Real-World Scenario

```bash
# Microservice: REST API, latency-sensitive
java -XX:+UseG1GC \
     -Xms2G -Xmx2G \
     -XX:MaxGCPauseMillis=100 \
     api-server.jar

# Batch job: High throughput, can tolerate pauses
java -XX:+UseParallelGC \
     -Xms4G -Xmx4G \
     batch-processor.jar

# Trading system: Sub-ms latencies required
java -XX:+UseZGC \
     -Xms10G -Xmx10G \
     trading-engine.jar
```
