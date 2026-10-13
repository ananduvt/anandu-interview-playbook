# Threading Deep-Dive & Virtual Threads

Advanced threading concepts: thread synchronization, thread pools, and Java 21's virtual threads that revolutionize concurrent programming.

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

// Better: try-with-resources (Java 7+)
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
