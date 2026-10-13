# Java Advanced Interview Questions

Advanced concepts: patterns, performance, frameworks, concurrency, and enterprise design.

---

## Question Categories

### Design Patterns
- Creational (Singleton, Factory, Builder)
- Structural (Decorator, Adapter, Proxy)
- Behavioral (Observer, Strategy, Template Method)
- When and why to use each pattern

### Concurrency & Threading
- Thread pool, ExecutorService, Future
- Producer-consumer, barrier patterns
- Deadlock prevention, race conditions
- Virtual threads (Java 21+)

### Performance & Optimization
- JVM tuning, garbage collection algorithms
- Profiling, identifying bottlenecks
- Caching strategies

### Functional Programming
- Streams API, lambda expressions
- Functional interfaces
- Monads, functional composition

### Advanced Collections
- ConcurrentHashMap, CopyOnWriteArrayList
- BlockingQueue, PriorityQueue
- Custom collection implementations

### Generics & Type System
- Type erasure, bounded wildcards
- Variance (covariance, contravariance)
- PECS principle

### Reflection & Bytecode
- Reflection APIs
- Annotations and annotation processing
- Bytecode manipulation

---

## Common Questions & Answers

### Volatile vs Synchronized vs Atomic
**Q: What's the difference between volatile, synchronized, and Atomic classes?**

| Feature | volatile | synchronized | Atomic |
|---------|----------|--------------|--------|
| **Purpose** | Visibility | Mutual exclusion | Atomicity (CAS) |
| **Lock** | None | Monitor lock | Compare-and-swap (hardware) |
| **Thread-safe** | Read/write visibility only | Full exclusion | Atomic operations only |
| **Performance** | Fast | Slow (locks) | Fast (lock-free) |
| **Use case** | Flag variables | Shared state, compound operations | Counters, simple updates |

```java
volatile boolean flag;       // Visible to all threads
AtomicInteger counter;       // Atomic increment
synchronized void method() {} // Mutual exclusion
```

---

### Runnable vs Callable
**Q: What's the difference between Runnable and Callable?**

| Feature | Runnable | Callable |
|---------|----------|----------|
| **Return value** | void | Returns result (generic type) |
| **Exception** | Cannot throw checked exceptions | Can throw checked exceptions |
| **Usage** | `Thread`, `Executor` | `ExecutorService` |
| **Interface method** | `run()` | `call()` |

```java
// Runnable (no return)
Runnable task = () -> System.out.println("No return");

// Callable (returns result)
Callable<Integer> task = () -> 42;  // Must call via executor/submit
```

---

### Future vs CompletableFuture
**Q: What's the difference between Future and CompletableFuture?**

- **Future**: Represents async computation result; blocking (`.get()` waits)
  - Created by `ExecutorService.submit()`
  - Limited chaining/composition
- **CompletableFuture**: More powerful; non-blocking chains and callbacks
  - Create with `CompletableFuture.supplyAsync()`
  - Chain with `.thenApply()`, `.thenAccept()`, `.thenCombine()`

```java
// Future (blocking)
Future<Integer> future = executor.submit(() -> 42);
Integer result = future.get();  // Blocks

// CompletableFuture (non-blocking)
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> 42);
cf.thenApply(x -> x * 2).thenAccept(System.out::println);
```

---

### Thread-Safe Singleton
**Q: How do you create a thread-safe singleton?**

**Approach 1: Enum (best)**
```java
enum Singleton {
    INSTANCE;
    // methods
}
Singleton.INSTANCE.method();
```

**Approach 2: Double-Checked Locking (must use volatile)**
```java
class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {  // 1st check (avoid lock if exists)
            synchronized(Singleton.class) {
                if (instance == null) {  // 2nd check (ensure once)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Why `volatile`?** Without it, partially-constructed object can be published.

---

### Double-Checked Locking Anti-Pattern
**Q: Why can double-checked locking break without volatile?**

Without `volatile`, the constructor writes (field initialization) can be reordered:
1. Other thread reads `instance != null` (but constructor not finished)
2. Returns partially-constructed object
3. NPE or other errors

**Must use `volatile` on the instance field** to prevent reordering and ensure visibility of all constructor writes.

---

## Interview Questions (With Answers)

<details>
<summary>What are the key features of Java 21?</summary>

**Answer:** Java 21 introduces virtual threads (lightweight threads with minimal overhead), sequence collections for ordered data, and records for immutable data carriers. Pattern matching is enhanced with record patterns. Project Loom improves concurrency; Project Panama simplifies native code interoperability.

📚 [Learn more: Java Versions & Features](../../java-core/versions-features.md)
</details>

<details>
<summary>What are the key differences between Java 17 and Java 8?</summary>

**Answer:** Java 17 added records (immutable data classes), sealed classes (restrict inheritance), pattern matching (simplify conditionals), and text blocks (multi-line strings). Java 8 introduced lambdas and streams. Java 17 also has better null safety and performance improvements via HotSpot enhancements.

📚 [Learn more: Java Versions & Features](../../java-core/versions-features.md)
</details>

<details>
<summary>What are the core multithreading concepts?</summary>

**Answer:** Multithreading involves creating multiple threads within a process. Key concepts: Thread lifecycle (new→runnable→running→blocked→terminated), synchronization (mutual exclusion via locks), thread safety (volatile, synchronized, AtomicXX), and communication (wait/notify). Threads share heap memory but have separate stacks.

📚 [Learn more: Threading & Virtual Threads](../../java-collections-concurrency/threading.md)
</details>

<details>
<summary>How does ExecutorService work in Spring?</summary>

**Answer:** ExecutorService manages a thread pool for async task execution. Spring's `@Async` annotation uses a TaskExecutor backed by ExecutorService. Submit tasks via `submit()` or `execute()`; get results with `Future.get()`. Provides better resource management than creating raw threads.

📚 [Learn more: Spring Threading & Async](../../spring-web/spring-threading.md)
</details>

<details>
<summary>What is an asynchronous transaction?</summary>

**Answer:** An asynchronous transaction decouples task submission from completion. The caller doesn't wait for the transaction to finish; instead, it receives a callback or Future. Useful for long-running operations (file uploads, API calls) to prevent blocking. Requires careful error handling and state management.

📚 [Learn more: Spring Boot Advanced](../../spring-web/spring-boot-advanced.md)
</details>

<details>
<summary>What are transactions and their propagation levels in Spring?</summary>

**Answer:** Transactions ensure ACID properties. Spring's @Transactional supports propagation levels: REQUIRED (join existing or create), REQUIRES_NEW (always create), NESTED (savepoint), NOT_SUPPORTED (run without), and others. Propagation determines how nested method calls share or isolate transactions.

📚 [Learn more: Spring Boot Questions](../verbal-spring-boot-questions.md)
</details>

<details>
<summary>Explain a design pattern with an example.</summary>

**Answer:** Example - Singleton Pattern ensures only one instance of a class. Used for: thread-safe caches, database connections, logging. Best implemented via enum or double-checked locking with volatile. Drawbacks: testing difficulty, hidden dependencies.

```java
public enum Singleton { INSTANCE; }
Singleton.INSTANCE.doSomething();
```

📚 [Learn more: Design Patterns](../../java-patterns-advanced/design-patterns.md)
</details>

---

## Sample Topics (To expand)

- Explain common design patterns and when to use them
- What's the difference between Singleton and static class?
- How do you handle concurrency without synchronization?
- What's a ThreadLocal and when should you use it?
- Explain virtual threads and how they differ from OS threads
- How does the Stream API work under the hood?
- What's the difference between CopyOnWriteArrayList and ArrayList?
- Explain generics and type erasure
- What's PECS (Producer-Extends, Consumer-Super)?
- How would you implement a thread-safe cache?
- What are the GC algorithms and when to use each?
- How do you profile and optimize Java applications?

---

[More content to be added progressively]

**See also:** [Design Patterns](../../java-patterns-advanced/design-patterns.md), [Threading](../../java-collections-concurrency/threading.md), [Interview Questions Overview](../index.md)
