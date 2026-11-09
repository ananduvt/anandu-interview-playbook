# Threading Deep-Dive & Virtual Threads

Advanced threading concepts: thread synchronization, thread pools, and Java 21's virtual threads that revolutionize concurrent programming.

**See also:** [Concurrency & Threads](concurrency.md) for thread basics, lifecycle, `Callable`/`Future`, `CompletableFuture` and IPC, and [Spring Threading & Async](../spring-web/spring-threading.md).


---

## Thread vs Runnable

### Thread Class Approach

```java
// Extend Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in thread: " + Thread.currentThread().getName());
    }
}

// Create and start
MyThread thread = new MyThread();
thread.start();  // Starts new thread, calls run() asynchronously
```

**Pros:**
- Direct access to Thread methods (setName, setPriority, setDaemon)
- Simple for single purpose classes

**Cons:**
- ❌ **Can only extend one class** (Java doesn't support multiple inheritance)
- Locks you into Thread class; can't extend another class
- Less flexible

### Runnable Interface Approach

```java
// Implement Runnable interface
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Running in thread: " + Thread.currentThread().getName());
    }
}

// Create thread with runnable
Thread thread = new Thread(new MyRunnable());
thread.start();  // Starts new thread, calls run() asynchronously

// Lambda syntax (modern)
new Thread(() -> {
    System.out.println("Running in thread");
}).start();

// Using ExecutorService (recommended)
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(new MyRunnable());  // Runnable submitted to pool
```

**Pros:**
- ✅ **Implement alongside other interfaces** (can extend another class)
- Better for object composition
- Reusable runnable object
- Works with thread pools/executors naturally

**Cons:**
- Need to wrap in Thread object manually (without executor)

### Comparison Table

| Aspect | Thread | Runnable |
|--------|--------|----------|
| **Type** | Class | Interface |
| **Inheritance** | Extends Thread; only one per class | Implements; can implement others too |
| **Flexibility** | Low; tied to Thread | High; composable |
| **Thread Management** | Manual `new Thread(...)` | Use ExecutorService (cleaner) |
| **Reusability** | Single-use object | Reusable across threads |
| **Object creation** | New object per thread | One Runnable can be shared across threads |
| **Best For** | Simple one-off threads | Thread pools, enterprise apps |

### Best Practice: Runnable + ExecutorService

```java
@Service
public class TaskProcessor {
    private ExecutorService executor = Executors.newFixedThreadPool(10);
    
    // Define reusable task
    private class DataProcessor implements Runnable {
        private final Long userId;
        
        public DataProcessor(Long userId) {
            this.userId = userId;
        }
        
        @Override
        public void run() {
            User user = userService.findById(userId);
            processUser(user);
        }
    }
    
    // Submit many runnables to executor
    public void processAllUsers(List<Long> userIds) {
        for (Long userId : userIds) {
            executor.submit(new DataProcessor(userId));
        }
    }
}
```

---

## Runnable vs Callable

The comparison table and `Callable` / `Future` examples are in [Concurrency & Threads: Runnable vs Callable](concurrency.md#runnable-vs-callable). In short: `Runnable.run()` returns nothing and can't throw checked exceptions; `Callable.call()` returns a value and can throw. Submit either to an `ExecutorService` to get a `Future`.

---

## Thread Synchronization

### 1. **Synchronized Keyword**

Ensures only one thread can access a block/method at a time.

```java
// Synchronized method
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // Only one thread can execute at a time
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Synchronized block (more fine-grained)
public class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized (lock) {
            count++;  // Only lock this critical section
        }
        // Do other stuff without holding the lock
    }
}
```

**Pros:** Simple, built-in, intuitive  
**Cons:** Coarse-grained, can deadlock, reentrant issues

---

### Class Lock vs Object Lock

Every object has one intrinsic monitor, and every `Class` object (`Counter.class`) has its own. `synchronized` decides which one is locked.

| Form | Lock acquired | Effect |
| :-- | :-- | :-- |
| `synchronized void m()` | `this` (object lock) | One thread per **instance** |
| `synchronized (this) { }` | `this` | Same as above |
| `static synchronized void m()` | `Counter.class` (class lock) | One thread per **class** (per classloader), across all instances |
| `synchronized (Counter.class) { }` | Class object | Same as above |
| `synchronized (lock) { }` | a private `lock` object | Fine-grained, not exposed to callers |

```java
public class Counter {
    private static int total;
    private int count;

    public synchronized void inc()             { count++; }  // locks this
    public static synchronized void incTotal() { total++; }  // locks Counter.class

    public void both() {
        synchronized (this)          { count++; }
        synchronized (Counter.class) { total++; }
    }
}
```

**Key behaviours**

- Threads on **different instances** calling instance-synchronized methods do not block each other.
- Instance-synchronized and static-synchronized methods use **different locks**, so they run concurrently. `inc()` does not protect `total`; a shared static field needs the class lock (or an `AtomicInteger`).
- Locks are reentrant: a thread holding a lock can enter other code guarded by the same lock.
- Non-synchronized methods are never blocked by either lock.
- Prefer a **private final lock object**: outside code can lock `this` or `Counter.class` too, causing contention or deadlock.
- Virtual threads: blocking inside `synchronized` pinned the carrier thread in Java 21; Java 24+ (so the Java 25 LTS) removed most pinning ([JEP 491](https://openjdk.org/jeps/491)).

**Common interview questions**

- *Can a static synchronized and an instance synchronized method run at the same time?* Yes, different monitors.
- *What does `wait()` release?* Only the monitor of the object it is called on; other held locks stay held.
- *Class lock with several classloaders?* The lock is per `Class` object, so the same class loaded by two loaders has two locks.

### 2. **Volatile Keyword**

Ensures visibility of changes across threads, but NOT atomicity.

```java
public class Flag {
    private volatile boolean running = true;  // Always read latest value
    
    public void stop() {
        running = false;  // All threads see this immediately
    }
    
    public void work() {
        while (running) {
            // Do work
        }
    }
}
```

**Use case:** Simple flag changes, not counters  
**Difference from synchronized:** Visibility only; no mutual exclusion

**What it guarantees (Java Memory Model):** a write to a `volatile` variable happens-before every later read of it, so the reader sees that write and everything the writer did before it. It also forbids reordering around the access (why double-checked locking needs `volatile`). It does not lock, and `count++` (read, add, write) is still a race.

| | `volatile` | Atomic classes (`AtomicInteger`...) |
| :-- | :-- | :-- |
| Visibility | Yes | Yes (internally volatile) |
| Atomic read-modify-write | No | Yes, via CAS (`incrementAndGet`, `compareAndSet`) |
| Use for | Single-writer flags, safe publication | Shared counters (`LongAdder` under high contention) |

---

### 3. **Atomic Variables**

Thread-safe operations without locking (lock-free via CAS - Compare-And-Swap).

```java
import java.util.concurrent.atomic.*;

public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic, no synchronization needed
    }
    
    public int getCount() {
        return count.get();
    }
    
    // Compare-and-swap
    public boolean compareAndSet(int expect, int update) {
        return count.compareAndSet(expect, update);
    }
}
```

**Common types:** `AtomicInteger`, `AtomicLong`, `AtomicReference`, `AtomicBoolean`  
**Pros:** Lock-free, high performance  
**Use:** Counters, flags, simple values

---

### AtomicReference Example and the ABA Problem

`AtomicReference<V>` gives lock-free compare-and-set on an **object reference** (compared by identity `==`, not `equals`). Typical use: publish an immutable snapshot and swap it atomically.

```java
import java.util.concurrent.atomic.AtomicReference;

public class Config {
    record Settings(int timeoutMs, String region) {}

    private final AtomicReference<Settings> ref =
        new AtomicReference<>(new Settings(1000, "eu"));

    public Settings current() { return ref.get(); }

    // Retry loop: read, build new immutable value, CAS; retry if another thread won
    public void raiseTimeout(int delta) {
        Settings prev, next;
        do {
            prev = ref.get();
            next = new Settings(prev.timeoutMs() + delta, prev.region());
        } while (!ref.compareAndSet(prev, next));
    }

    // Equivalent shorthand (Java 8+)
    public void raiseTimeout2(int delta) {
        ref.updateAndGet(s -> new Settings(s.timeoutMs() + delta, s.region()));
    }
}
```

The function given to `updateAndGet`/`accumulateAndGet` may run several times under contention, so it must be **side-effect free**.

**ABA problem:** a thread reads `A`; another thread changes `A -> B -> A`; the first thread's CAS succeeds because the value looks unchanged although state changed in between. It bites lock-free stacks and queues that reuse nodes. Fix by attaching a version stamp.

```java
import java.util.concurrent.atomic.AtomicStampedReference;

AtomicStampedReference<String> asr = new AtomicStampedReference<>("A", 0);

int[] holder = new int[1];
String val = asr.get(holder);   // value "A", stamp 0
int stamp = holder[0];

// Meanwhile another thread does A -> B -> A, bumping the stamp to 2

boolean ok = asr.compareAndSet(val, "C", stamp, stamp + 1);
System.out.println(ok);         // false: stamp changed, ABA detected
```

| Class | Use |
| :-- | :-- |
| `AtomicReference` | Swap immutable snapshots; ABA harmless if only the latest value matters |
| `AtomicStampedReference` | Reference plus `int` version; detects ABA |
| `AtomicMarkableReference` | Reference plus `boolean` mark (for example a logically-deleted flag) |

**Interview tips:** garbage collection prevents the C/C++ memory-reuse form of ABA (an object is not recycled while referenced), but logical ABA remains. Keep values in an `AtomicReference` **immutable**; mutating the object inside defeats the CAS.

### 4. **Locks (java.util.concurrent.locks)**

Fine-grained control over locking behavior.

```java
import java.util.concurrent.locks.*;

public class LockCounter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();  // Always unlock in finally
        }
    }
    
    // Try-lock with timeout
    public boolean tryIncrement(long timeout, TimeUnit unit) {
        if (lock.tryLock(timeout, unit)) {
            try {
                count++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;  // Couldn't acquire lock in time
    }
}

// ReadWriteLock: multiple readers, single writer
public class Cache<K, V> {
    private final Map<K, V> cache = new HashMap<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    public V get(K key) {
        lock.readLock().lock();
        try {
            return cache.get(key);  // Multiple threads can read
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void put(K key, V value) {
        lock.writeLock().lock();
        try {
            cache.put(key, value);  // Exclusive write access
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

**Pros:** Flexible, timeout support, fair/unfair options  
**Use:** When synchronized is too coarse, or need timeouts/fairness

---

## Thread Pools & ExecutorService

Instead of creating threads manually, use thread pools for efficiency.

```java
import java.util.concurrent.*;

// Fixed thread pool
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submit tasks
for (int i = 0; i < 100; i++) {
    final int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " on " + Thread.currentThread().getName());
    });
}

executor.shutdown();  // No new tasks accepted
executor.awaitTermination(1, TimeUnit.MINUTES);  // Wait for completion

// Better: try-with-resources (Java 19+, ExecutorService is AutoCloseable)
try (ExecutorService executor = Executors.newFixedThreadPool(10)) {
    IntStream.range(0, 100).forEach(i ->
        executor.submit(() -> doWork(i))
    );
}  // Automatically shutdown
```

**Common Pool Types:**
- `newFixedThreadPool(n)`: Fixed number of threads; queues excess tasks
- `newCachedThreadPool()`: Creates threads as needed; reuses idle threads
- `newSingleThreadExecutor()`: One thread; tasks run sequentially
- `newScheduledThreadPool(n)`: For delayed/periodic tasks

**Executor hierarchy:** `Executor` (only `execute(Runnable)`) -> `ExecutorService` (lifecycle plus `submit`, `invokeAll`, `invokeAny`) -> `ScheduledExecutorService` (delayed and periodic tasks; a more robust replacement for `java.util.Timer`).

| Method | Behaviour |
|--------|-----------|
| `execute(Runnable)` | Fire and forget; no result; an uncaught exception goes to the thread's handler |
| `submit(Callable<T>)` / `submit(Runnable)` | Returns a `Future`; exceptions are captured inside it (see [Future exceptions](concurrency.md#future-exception-handling)) |
| `invokeAll(tasks)` | Runs a batch, blocks until all finish, returns `List<Future<T>>` |
| `invokeAny(tasks)` | Returns the result of the first task to succeed and cancels the rest |
| `shutdown()` | Stops accepting tasks; queued tasks still run |
| `shutdownNow()` | Interrupts running tasks and returns the never-started ones |
| `awaitTermination(t, unit)` | Blocks until the pool terminates or the timeout expires |

Pools never close on their own; non-daemon worker threads keep the JVM alive. Standard shutdown pattern:

```java
executor.shutdown();
try {
    if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

!!! warning "Executors factory caveats"
    `newFixedThreadPool` and `newSingleThreadExecutor` use an unbounded queue (possible `OutOfMemoryError` under load); `newCachedThreadPool` has no thread limit. For production, build a `ThreadPoolExecutor` with a bounded queue and a rejection policy. Cached idle threads are removed after 60 seconds.

**Risks of thread pools:**
- **Pool deadlock:** every pool thread waits for a result from a task still queued because no thread is free.
- **Thread leakage:** a thread leaves the pool to run a task and isn't returned (for example an uncaught exception kills it), so the pool slowly shrinks.
- **Resource thrashing:** too many threads waste time on context switching and can starve each other. Size pools to the workload (CPU-bound: about the core count; I/O-bound: larger).

---

## Virtual Threads (Java 21) — Game Changer

Virtual threads are lightweight, JVM-managed threads that enable **millions of concurrent tasks** without OS thread overhead.

### Problem: OS Thread Limits

```java
// ❌ Traditional OS threads (heavy)
ExecutorService executor = Executors.newFixedThreadPool(1000);

for (int i = 0; i < 10000; i++) {
    executor.submit(() -> {
        // Make HTTP request (blocks)
        response = httpClient.get("https://api.example.com/data");
        // Process response
    });
}
// Problem: 1000 threads × 1-2MB = 1-2GB memory, context switching overhead
```

### Solution: Virtual Threads

```java
// ✅ Virtual threads (lightweight, millions feasible)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100000; i++) {
        executor.submit(() -> {
            // Make HTTP request (blocks, but cheap suspend/resume)
            response = httpClient.get("https://api.example.com/data");
            // Process response
        });
    }
}  // Automatic shutdown and wait for all to complete
```

**Key differences:**
- **OS Thread:** 1-2MB memory, expensive context switch
- **Virtual Thread:** Few KB, lightweight suspend/resume (no context switch)
- **Scalability:** ~1,000 OS threads vs. millions of virtual threads

### Virtual Thread Examples

**Simple creation:**
```java
Thread vt = Thread.ofVirtual()
    .name("vt-", 1)  // Name prefix, starting number
    .start(() -> {
        System.out.println("Virtual thread running");
    });

vt.join();  // Wait for completion
```

**High-concurrency scenario:**
```java
// Before (OS threads): bottleneck at ~1,000 concurrent requests
ExecutorService executor = Executors.newFixedThreadPool(1000);

// After (virtual threads): handle 100,000+ concurrent requests
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100000; i++) {
        executor.submit(() -> {
            // Blocking I/O is now cheap
            String result = blockingIoCall();
            processResult(result);
        });
    }
}
```

**Spring Boot integration (Java 21+):**
```java
@Configuration
public class ThreadingConfig {
    
    @Bean
    public TaskExecutor taskExecutor() {
        // Use virtual threads for async tasks
        return new SimpleAsyncTaskExecutor("vt-");  // Spring auto-detects virtual thread env
    }
}

@Service
public class PaymentService {
    
    @Async  // Runs on virtual thread
    public CompletableFuture<PaymentResponse> processPaymentAsync(Order order) {
        // Blocking I/O is now scalable
        PaymentResponse response = paymentGateway.charge(order);
        return CompletableFuture.completedFuture(response);
    }
}
```

---

## Virtual Threads vs. Reactive Frameworks

| Aspect | Virtual Threads | Reactive (WebFlux) |
|--------|-----------------|-------------------|
| **Code Style** | Synchronous, imperative | Async, callback-heavy |
| **Learning Curve** | Easy (looks like normal code) | Steep (chains, subscribers, backpressure) |
| **Debugging** | Stack traces are clear | Stack traces fragmented |
| **Thread Pool** | Millions (virtual) | Bounded (usually ~N threads) |
| **Blocking I/O** | ✅ OK to block (cheap suspend) | ❌ Blocks thread pool |
| **Error Handling** | Normal try-catch | Operators (`.onErrorResume()`, etc.) |
| **When to Use** | I/O-bound, greenfield Java 21+ | Existing reactive systems, strict resource limits |

**Virtual threads eliminate most reasons to use reactive frameworks!**

---

## Common Concurrency Patterns

### 1. **Producer-Consumer**
```java
BlockingQueue<Message> queue = new LinkedBlockingQueue<>();

// Producer thread
executor.submit(() -> {
    for (int i = 0; i < 100; i++) {
        queue.put(new Message(i));  // Blocks if queue full
    }
});

// Consumer threads
for (int i = 0; i < 5; i++) {
    executor.submit(() -> {
        while (true) {
            Message msg = queue.take();  // Blocks if queue empty
            processMessage(msg);
        }
    });
}
```

### 2. **Countdown Latch** (Wait for N threads to finish)
```java
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    executor.submit(() -> {
        doWork();
        latch.countDown();  // Signal completion
    });
}

latch.await();  // Wait until all count down
System.out.println("All tasks done!");
```

### 3. **Cyclic Barrier** (Wait for N threads at a point)
```java
CyclicBarrier barrier = new CyclicBarrier(3);

for (int i = 0; i < 3; i++) {
    executor.submit(() -> {
        System.out.println(Thread.currentThread().getName() + " is ready");
        barrier.await();  // Wait for all to reach this point
        System.out.println(Thread.currentThread().getName() + " starting!");
    });
}
```

### 4. **Semaphore** (Control access to N resources)
```java
Semaphore semaphore = new Semaphore(3);  // Max 3 concurrent accesses

executor.submit(() -> {
    semaphore.acquire();  // Wait if 3 already using
    try {
        accessLimitedResource();
    } finally {
        semaphore.release();
    }
});
```

---

## Interview Talking Points

- **Synchronized vs. locks?** Synchronized is simpler, locks are flexible (timeout, fairness, try-lock)
- **Volatile vs. synchronized?** Volatile = visibility only; synchronized = mutual exclusion + visibility
- **Virtual threads impact?** Eliminates thread pool as bottleneck; I/O scales to millions; code stays imperative
- **Virtual thread limitations?** Pinning to OS thread if using synchronized (use locks instead); JVM-managed
- **Thread pool sizing?** Compute-bound = CPU count; I/O-bound = much larger or use virtual threads
- **Structured concurrency (Java 21)?** Automatic resource cleanup for thread groups (preview feature)
- **When to use AtomicInteger vs. synchronized?** Atomic: simple counters/flags; synchronized: complex shared state

---

**See also:** [Concurrency & Threads (Basics)](concurrency.md), [Java Collections & Concurrency Index](index.md)

**Practice questions:** [Java Concurrency](../interview-questions/verbal/verbal-java-concurrency-questions.md), [Java Advanced](../interview-questions/verbal/verbal-java-advanced-questions.md)
