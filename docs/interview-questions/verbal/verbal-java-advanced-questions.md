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
