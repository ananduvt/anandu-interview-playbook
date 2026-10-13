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
  String result = obj switch {
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
