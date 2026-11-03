# Java Versions & Features

Java release cadence, LTS versions, headline features per version, comparison tables, and migration guidance. For the JVM, memory model and garbage collection, see [JVM Internals](jvm-internals.md).

---

## Long-Term Support (LTS) Overview

**LTS Releases** receive **extended support** for 5+ years (sometimes 10+ years for Java 8 and 11):
- **Java 8** (March 2014): Extended support until December 2030
- **Java 11** (September 2018): Extended support until January 2032
- **Java 17** (September 2021): Extended support until September 2029
- **Java 21** (September 2023): Extended support until September 2031
- **Java 25** (September 2025): Extended support until September 2033 (Premier Support until September 2030)

> Oracle's no-fee terms for JDK 21 updates run for one year after JDK 25's release (until September 2026); after that, Oracle's roadmap points users to JDK 25 for no-fee updates. This applies to Oracle's JDK builds; check your vendor's terms (OpenJDK distributions such as Temurin have their own support policies).

**Non-LTS Releases** receive ~6 months of support (overlap period only). Since Java 9, a new version ships every 6 months; LTS versions are chosen every ~2–3 years (8, 11, 17, 21, 25; the next planned LTS is Java 29 in September 2027).

---

## Version Timeline

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
| 22 | 2024 | **Foreign Function & Memory API (standard)**, unnamed variables & patterns (standard), launch multi-file source programs, statements before `super()` (preview) |
| 23 | 2024 | Markdown doc comments, **generational ZGC by default**, primitive types in patterns (preview), module import declarations (preview), flexible constructor bodies (2nd preview) |
| 24 | 2025 | **Stream Gatherers (standard)**, Class-File API (standard), virtual threads without `synchronized` pinning, Security Manager permanently disabled, flexible constructor bodies (3rd preview), primitive types in patterns (2nd preview) |
| 25 | 2025 | **Scoped values**, flexible constructor bodies, compact source files and instance `main`, module import declarations, compact object headers, AOT method profiling (LTS) |

---

## Quick Version Comparison (LTS Releases)

Quick reference to understand differences between major LTS versions:

| Aspect | Java 8 (2014) | Java 11 (2018) | Java 17 (2021) | Java 21 (2023) | Java 25 (2025) |
|--------|--------|---------|---------|---------|---------|
| **Support Until** | Dec 2030 | Jan 2032 | Sept 2029 | Sept 2031 | Sept 2033 |
| **Key Features** | Lambdas, Streams, functional interfaces, default methods, java.time, Optional, method references | HTTP/2 client, `var` type inference, clean JDK (Java EE removed), ZGC, single-file execution | Sealed classes, Records (standard), pattern matching for instanceof, Foreign Function & Memory API, strong encapsulation | Virtual threads (Loom), record patterns, pattern matching for switch (standard), sequenced collections, structured concurrency | Scoped values, flexible constructor bodies, compact source files and instance `main`, module import declarations, compact object headers, AOT method profiling (plus 22–24: FFM API, unnamed variables, stream gatherers, Class-File API) |
| **Concurrency Model** | OS threads (heavy) | OS threads | OS threads | Virtual threads (lightweight) | Virtual threads (no pinning on `synchronized` since 24) |
| **Thread Scalability** | ~1,000 threads | ~1,000 threads | ~1,000 threads | Millions of threads | Millions of threads |
| **Type Safety** | Basic generics | Generics + var | Sealed classes, pattern matching | Advanced patterns, exhaustive matching | Same as 21; primitive patterns still preview |
| **Functional Support** | Lambdas, streams, Optional | Mature API | Mature + pattern matching | Pattern matching for switch | Pattern matching for switch + Stream Gatherers (final since 24) |
| **Collections** | Basic (List, Set, Map) | Stable | Stable | Sequenced Collections (order guarantees) | Sequenced Collections |
| **Build Adoption** | ⭐⭐⭐ (ubiquitous) | ⭐⭐⭐ (very high) | ⭐⭐ (growing) | ⭐ (emerging) | ⭐ (emerging) |
| **Enterprise Use** | ✅ Standard | ✅ Very common | ✅ Growing adoption | ⭐ Early adoption | ⭐ Early adoption |
| **Best For** | Legacy systems, broad compatibility | Modern systems, standard choice | New greenfield projects, microservices | High-concurrency I/O-bound apps | New projects that want the newest LTS and the longest support runway |
| **When NOT to Use** | Performance-critical systems (no streams) | Very old frameworks | Complex inheritance hierarchies (sealed classes) | CPU-bound workloads (OS threads better) | Dependencies or tooling not yet certified for Java 25 |

### **Quick Decision Guide**

- **Stuck on Java 8?** → Migration path: 8 → 11 (low cost) → 17 (moderate cost)
- **Starting new project?** → Use **Java 25 LTS** (longest support runway), or Java 21 LTS (virtual threads + patterns, widely supported)
- **Legacy enterprise?** → Java 11 LTS (good balance of stability + modern features)
- **High I/O workloads?** → Java 21 (virtual threads = game-changer)
- **Need sealed classes + exhaustive pattern matching?** → Java 17 minimum
- **Not using record patterns or virtual threads?** → Java 17 is sufficient

---

## LTS Version Comparison Table

Feature matrix showing what each LTS version introduced:

| Feature | Java 8 | Java 11 | Java 17 | Java 21 | Java 25 |
|---------|--------|---------|---------|---------|--------|
| **Functional Programming** | ✅ Lambdas, Streams | | | |  |
| Functional Interfaces | ✅ `@FunctionalInterface` | | | |  |
| Default/Static Methods in Interfaces | ✅ New | | | |  |
| `java.time` API | ✅ Immutable date/time | | | |  |
| Optional<T> | ✅ Introduced | ✅ Mature | ✅ Mature | ✅ Mature | ✅ Mature |
| Method References | ✅ `ClassName::method` | | | |  |
| **Type Inference** | | ✅ `var` (from Java 10) | ✅ Enhanced | ✅ Enhanced | ✅ Enhanced |
| **HTTP Support** | | ✅ Standard HTTP/2 client | ✅ Stable | ✅ Stable | ✅ Stable |
| **Single-File Execution** | | ✅ `java HelloWorld.java` | ✅ Supported | ✅ Supported | ✅ Supported |
| **Records** | | | ✅ Standard | ✅ Mature | ✅ Mature |
| **Pattern Matching - instanceof** | | | ✅ Standard | ✅ Standard | ✅ Standard |
| **Sealed Classes** | | | ✅ New | ✅ Mature | ✅ Mature |
| **Pattern Matching - switch** | | | | ✅ Standard | ✅ Standard |
| **Record Patterns** | | | | ✅ Standard | ✅ Standard |
| **Virtual Threads** | | | | ✅ **Game-changer** | ✅ Mature (no `synchronized` pinning since 24) |
| **Sequenced Collections** | | | | ✅ Order guarantees | ✅ Order guarantees |
| **Structured Concurrency** | | | | ✅ Preview | ✅ Preview (5th) |
| **Scoped Values** | | | | ✅ Preview | ✅ Standard |
| **Flexible Constructor Bodies** (code before `super()`) | | | | | ✅ Standard |
| **Compact Source Files / Instance `main`** | | | | ✅ Preview | ✅ Standard |
| **Module Import Declarations** | | | | | ✅ Standard |
| **Compact Object Headers** | | | | | ✅ Opt-in product option |
| **Primitive Types in Patterns** | | | | | ✅ Preview (3rd) |
| **Stream Gatherers** | | | | | ✅ Standard (final in 24) |
| **Default GC** | Parallel GC (G1 optional) | G1GC (ZGC experimental) | G1GC (ZGC production) | G1GC (ZGC production; generational ZGC) | G1GC (ZGC generational only; generational Shenandoah) |
| **Module System** | | | ✅ Mature | ✅ Mature | ✅ Mature |
| **Strong Encapsulation** | | | ✅ Enforced (illegal-access denied) | ✅ Enforced | ✅ Enforced |
| **JDK Removed Java EE** | | ✅ (CORBA, XML, RMI) | | |  |
| **Support Until** | Dec 2030 | Jan 2032 | Sept 2029 | Sept 2031 | Sept 2033 |

---

## LTS Feature Deep-Dive

### Java 8 (2014) — Functional Paradigm Shift

**Why it matters**: Java 8 fundamentally changed how Java code is written, introducing functional programming as a first-class paradigm.

**Theme:** Concise, expressive code

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

**Theme:** Production-ready, clean platform

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

**Theme:** Safer, more maintainable code (type safety)

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

**Theme:** Scalable, high-concurrency systems

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
- **Unnamed Variables & Patterns** (`_`, preview in 21, standard in 22): Mark unused variables and pattern components explicitly
- **Structured Concurrency** (Preview): Simplify task coordination; automatic resource cleanup

**Interview angle**: "How do virtual threads change concurrency?" → Scale to millions; fewer threads per task; simpler async patterns without callbacks/reactive frameworks.

---

### Java 25 (2025) — Cleaner Code & Leaner Runtime

**Why it matters**: Java 25 is the LTS that finalizes the language and runtime work from Java 22–25 (scoped values, flexible constructor bodies, compact source files), so most teams jump here from 21.

**Theme:** Cleaner everyday code and a leaner runtime

- **Scoped Values** (standard): Immutable, bounded-lifetime data shared with child threads; a safer, cheaper alternative to `ThreadLocal`, designed for virtual threads
- **Flexible Constructor Bodies** (standard): Run validation or compute arguments *before* calling `super(...)` or `this(...)`
  ```java
  class Square extends Shape {
      Square(double side) {
          if (side <= 0) throw new IllegalArgumentException("side must be positive");  // before super()
          super(side, side);
      }
  }
  ```
- **Compact Source Files & Instance `main`** (standard): Beginner-friendly programs without the `public static void main(String[] args)` boilerplate
  ```java
  void main() {
      System.out.println("Hello");
  }
  ```
- **Module Import Declarations** (standard): `import module java.base;` imports every package a module exports
- **Compact Object Headers** (product option, opt-in via `-XX:+UseCompactObjectHeaders`): Smaller object headers reduce heap usage
- **Ahead-of-Time cache improvements**: Command-line ergonomics and method profiling for faster startup and warm-up (builds on AOT class loading & linking from 24)
- **Generational Shenandoah**: Low-pause collector with a generational mode
- **Key Derivation Function API**, plus JFR improvements (cooperative sampling, method timing & tracing)
- **Still preview or incubating in 25**: Structured Concurrency (5th preview), Primitive Types in Patterns (3rd preview), Stable Values, PEM encodings, Vector API (10th incubator)

**Also arrived since Java 21** (Java 22–24): Foreign Function & Memory API and Unnamed Variables & Patterns (standard in 22), generational ZGC by default (23), Stream Gatherers and the Class-File API (standard in 24), virtual threads no longer pin on `synchronized` (24), Security Manager permanently disabled (24).

**Interview angle**: "What changed between Java 21 and 25?" → Scoped values and flexible constructor bodies are final, compact source files simplify entry points, virtual threads no longer pin on `synchronized`, and the runtime gets compact object headers and AOT improvements. Structured concurrency and primitive patterns are still preview.

---

## Java Migration: 8 → 11 → 17 → 21 → 25

**Migration strategy**: Don't jump directly; go incrementally: 8 → 11 (easiest) → 17 (moderate) → 21 (modern) → 25 (current LTS).

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

### Java 21 → Java 25 (Current LTS)

**Effort:** Low | **Breaking changes:** Few (removals and warnings introduced in 22–24)

```
What's new since 21:
- Language: scoped values, flexible constructor bodies, compact source files, module import declarations (all standard)
- Runtime: compact object headers (opt-in), generational ZGC by default (non-generational mode removed), AOT class loading and method profiling
- Libraries: Foreign Function & Memory API, Stream Gatherers, Class-File API, Key Derivation Function API

What can break or warn:
- Security Manager permanently disabled (System.setSecurityManager throws)
- Warnings for sun.misc.Unsafe memory-access methods and for JNI use (preparing to restrict native access)
- Non-generational ZGC removed (old ZGC flags are obsolete)
- 32-bit x86 ports removed (Windows in 24, Linux in 25)
- Bytecode tools and agents (ASM, ByteBuddy, etc.) must support class-file version 69
```

**Checklist:**
```
[ ] Upgrade build plugins, frameworks, and bytecode libraries to versions that support Java 25
[ ] Remove Security Manager usage
[ ] Fix Unsafe / JNI warnings (use FFM API or VarHandle where possible; add --enable-native-access if needed)
[ ] Re-test GC flags (drop obsolete ZGC options)
[ ] Benchmark compact object headers and AOT cache (both optional)
[ ] Update CI/CD and container base images to Java 25
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
| Java 8 | Java 25 | Moderate-High | 8-12 weeks | Moderate |
| Java 11 | Java 25 | Moderate | 3-6 weeks | Moderate |
| Java 17 | Java 25 | Low-Moderate | 2-4 weeks | Low |
| Java 21 | Java 25 | Low | 1-2 weeks | Low |

*Timelines are rough estimates; they depend on test coverage and dependency support.*

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

## Interview tips
- Know the **LTS** line: **8, 11, 17, 21, 25**.
- Be ready to explain the marquee features: **8** (lambdas/streams), **11** (HTTP client, `var` from 10),
  **17** (sealed classes), **21** (**virtual threads**), **25** (scoped values, flexible constructor bodies, compact source files).
- Since Java 9, Oracle uses a **6-month cadence** with LTS every few years.
- Understand the **progression**: 8→11 (functional; JDK cleanup), 11→17 (pattern matching; sealed classes), 17→21 (virtual threads for scale), 21→25 (finalized scoped values and constructor flexibility; leaner runtime).

**Practice questions:** [Java Modern Features and Generics](../interview-questions/verbal/verbal-java-modern-questions.md)
