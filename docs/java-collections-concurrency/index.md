# Java Collections & Concurrency

Master Java's collection framework, functional programming paradigm, concurrent utilities, and stream processing for efficient, thread-safe applications.

---

## Topics

### [Collections](collections.md)
**List - Set - Map - ArrayList - HashMap - HashSet - LinkedHashMap - TreeMap - ConcurrentHashMap**  
Collection framework: interface hierarchy, implementations, time complexity (access/insert/delete), when to use each, thread-safety, null handling, iteration patterns.

### [Functional Programming](functional-programming.md)
**Lambdas - Functional Interfaces - Method References - Function Composition - Higher-Order Functions**  
Functional paradigm in Java: lambda syntax, @FunctionalInterface, method references, function composition, common functional interfaces (Function, Predicate, Consumer, Supplier).

### [Streams](streams.md)
**Stream API - Lazy Evaluation - Intermediate Operations - Terminal Operations - Collectors - Reduction**  
Stream processing: creating streams, filter/map/flatMap/reduce, collect operations, parallel streams, performance considerations, common patterns.

### [Concurrency & Threads](concurrency.md)
**Thread Creation - Synchronization - Locks - Atomics - Volatile - Executors - Thread Pools - Future**  
Concurrent programming: thread lifecycle, race conditions, synchronization mechanisms, ReentrantLock, ConcurrentHashMap, thread pools, async patterns with Future/CompletableFuture.

### [Threading Deep-Dive & Virtual Threads](threading.md)
**Thread Synchronization - Atomic Operations - Thread Pools - Virtual Threads (Java 21) - Structured Concurrency - Concurrency Patterns**  
Advanced threading: synchronized vs. locks, atomics without locking, ExecutorService, virtual threads for millions of concurrent tasks, comparison with reactive frameworks, producer-consumer and barrier patterns.

### [Testing](testing.md)
**Unit Testing - Mocking - Test Collections - Concurrency Testing - Thread Safety Verification**  
Testing Java applications: JUnit, Mockito, testing collections, concurrent code testing, race condition detection, integration testing.

---

## Quick Interview Prep

**Collections mastery:**
- **ArrayList** O(1) access, O(n) insert/delete; **LinkedList** O(n) access, O(1) insert/delete
- **HashMap** O(1) average, O(n) worst; **TreeMap** O(log n) all operations, sorted order
- **Thread-safe**: ConcurrentHashMap, Collections.synchronizedList(), CopyOnWriteArrayList

**Concurrency essentials:**
- **synchronized** keyword for methods/blocks; **ReentrantLock** for fine-grained control
- **volatile** for visibility; **AtomicInteger**, **AtomicReference** for atomic operations
- **Thread pools**: Executors.newFixedThreadPool(), newCachedThreadPool()
- **CompletableFuture** for async chaining without callbacks

**Streams patterns:**
- filter-map-collect for transformations
- reduce() for aggregations
- parallel() for multi-threaded processing (careful with side effects!)
