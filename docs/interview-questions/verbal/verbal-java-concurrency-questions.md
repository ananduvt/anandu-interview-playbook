# Java Concurrency Interview Questions

Threads, locks, memory visibility, deadlocks, and asynchronous coordination with Future and CompletableFuture.

---

## 1. Explain deadlock in Java with an example

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A deadlock occurs when two or more threads each hold a lock the other needs and wait forever. None can proceed.</p>

<h3>Why / Context</h3>
<p>All four Coffman conditions must hold:</p>
<ul>
<li><strong>Mutual exclusion:</strong> a resource is held exclusively.</li>
<li><strong>Hold and wait:</strong> a thread holds one lock while requesting another.</li>
<li><strong>No preemption:</strong> locks cannot be taken away.</li>
<li><strong>Circular wait:</strong> a cycle of threads each waiting for the next one's lock.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
Object a = new Object(), b = new Object();

Thread t1 = new Thread(() -> {
    synchronized (a) { sleep(100); synchronized (b) { work(); } }   // a then b
});
Thread t2 = new Thread(() -> {
    synchronized (b) { sleep(100); synchronized (a) { work(); } }   // b then a
});
t1.start(); t2.start();   // each holds one lock and waits for the other
```

<h3>When to Use</h3>
<ul>
<li><strong>Prevention:</strong> acquire locks in a single global order (break circular wait); hold fewer locks and for less time; avoid calling foreign or overridable code while holding a lock.</li>
<li><strong>Timeouts:</strong> <code>ReentrantLock.tryLock(timeout, unit)</code> lets a thread back off and retry (break hold-and-wait).</li>
<li><strong>Detection:</strong> thread dump (<code>jstack &lt;pid&gt;</code> reports "Found one Java-level deadlock"), <code>ThreadMXBean.findDeadlockedThreads()</code>, VisualVM.</li>
<li>Related problems: livelock (threads keep reacting but make no progress) and starvation (a thread never gets the lock).</li>
<li>Prefer higher-level tools (concurrent collections, executors, immutable data) to reduce manual locking.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 2. What is the difference between a class lock and an object lock?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>An object lock is the monitor of a specific instance (used by instance <code>synchronized</code> methods and <code>synchronized(this)</code>). A class lock is the monitor of the <code>Class</code> object (used by <code>static synchronized</code> methods and <code>synchronized(Foo.class)</code>). They are different monitors, so they do not block each other.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Object lock</th><th>Class lock</th></tr>
<tr><td>Monitor on</td><td>The instance</td><td>The <code>Class</code> object (one per class loader)</td></tr>
<tr><td>Syntax</td><td><code>synchronized void m()</code>, <code>synchronized(this)</code></td><td><code>static synchronized void m()</code>, <code>synchronized(Foo.class)</code></td></tr>
<tr><td>Scope</td><td>Threads using the same instance serialize; different instances run in parallel</td><td>All threads across all instances serialize on the static section</td></tr>
<tr><td>Protects</td><td>Instance state</td><td>Static state</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
class Counter {
    private int count;
    private static int total;

    synchronized void inc() { count++; }              // lock: this
    static synchronized void incTotal() { total++; }  // lock: Counter.class

    void both() {
        synchronized (this) { count++; }
        synchronized (Counter.class) { total++; }     // different monitors, can run together
    }
}
```

<h3>When to Use</h3>
<ul>
<li>Protect static data with the class lock, instance data with the object lock; mixing them does not give mutual exclusion.</li>
<li>Prefer a private final lock object over <code>this</code> to stop outside code from locking on your instance.</li>
<li>Locks are reentrant: a thread holding one can enter another synchronized block on the same monitor.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 3. What is the difference between a user thread and a daemon thread?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The JVM stays alive while at least one user (non-daemon) thread is running. Daemon threads are background helpers that the JVM abandons abruptly when the last user thread finishes.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>User thread</th><th>Daemon thread</th></tr>
<tr><td>Keeps JVM alive</td><td>Yes</td><td>No</td></tr>
<tr><td>Typical use</td><td>Main work, request processing</td><td>GC, housekeeping, monitoring, timers</td></tr>
<tr><td>On JVM exit</td><td>Runs to completion</td><td>Killed abruptly: <code>finally</code> blocks may not run</td></tr>
<tr><td>Default</td><td>Inherits from the creating thread (main is user)</td><td>Must call <code>setDaemon(true)</code></td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
Thread t = new Thread(() -> { while (true) heartbeat(); });
t.setDaemon(true);   // must be set BEFORE start(), otherwise IllegalThreadStateException
t.start();
// when main and other user threads finish, the JVM exits even though t is still looping
```

<h3>When to Use</h3>
<ul>
<li>Use daemons for non-critical background tasks; never for work that must complete (file writes, transactions).</li>
<li>A thread created by a daemon thread is a daemon by default.</li>
<li>Executor pools create user threads by default; supply a <code>ThreadFactory</code> if you want daemons, or shut the pool down so the JVM can exit.</li>
<li>Virtual threads (Java 21) are always daemon threads.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 4. How do you implement multithreading and concurrency control in Java? Discuss thread safety and pitfalls

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Run tasks on an <code>ExecutorService</code> (not hand-made threads), and make shared state safe by preferring immutability and confinement first, then atomics and concurrent collections, and explicit locks or <code>synchronized</code> only where needed.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Thread safety</strong> needs atomicity (compound actions are indivisible), visibility (writes seen by other threads), and ordering.</li>
<li><strong>Tools, lightest first:</strong> immutable objects, thread confinement, <code>volatile</code> (visibility flag), <code>Atomic*</code>/<code>LongAdder</code> (lock-free counters), <code>ConcurrentHashMap</code> and other concurrent collections, <code>synchronized</code>/<code>ReentrantLock</code>/<code>ReadWriteLock</code>/<code>StampedLock</code>, coordination (<code>CountDownLatch</code>, <code>CyclicBarrier</code>, <code>Semaphore</code>, <code>BlockingQueue</code>).</li>
<li><strong>Pitfalls:</strong> race conditions (check-then-act, <code>i++</code>), deadlock, livelock, starvation, lost wake-ups, unsafe publication, double-checked locking without <code>volatile</code>, <code>SimpleDateFormat</code> shared between threads, <code>ThreadLocal</code> leaks in pools, unbounded pools or queues, swallowed exceptions in tasks.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
ExecutorService pool = Executors.newFixedThreadPool(8);   // bounded, reused threads
AtomicInteger processed = new AtomicInteger();
ConcurrentMap<String, Integer> totals = new ConcurrentHashMap<>();

for (Order o : orders) {
    pool.submit(() -> {
        totals.merge(o.customer(), o.amount(), Integer::sum);   // atomic update
        processed.incrementAndGet();
    });
}
pool.shutdown();
pool.awaitTermination(1, TimeUnit.MINUTES);
```

<h3>When to Use</h3>
<ul>
<li>Keep critical sections small; never call unknown code or do I/O while holding a lock.</li>
<li>Size pools from the workload: about CPU cores for CPU-bound, larger for blocking I/O, or use virtual threads (Java 21+) for large numbers of blocking tasks.</li>
<li>Always shut executors down; handle exceptions inside tasks (they are captured in the <code>Future</code> from <code>submit</code>).</li>
<li>Test with stress tests and tools (jcstress, thread dumps), since races may not appear on a developer machine.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/threading/">Learn more: Threading Deep-Dive</a></p>

</details>

---

## 5. What is the volatile keyword and when should it be used?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>volatile</code> on a field guarantees visibility and ordering: a write by one thread is immediately visible to subsequent reads by other threads, and the JVM cannot reorder around it. It does <strong>not</strong> make compound operations atomic.</p>

<h3>Why / Context</h3>
<ul>
<li>Without it, a thread may cache a value in a register or CPU cache and never see another thread's update (a loop may spin forever).</li>
<li><strong>Happens-before:</strong> a write to a volatile variable happens-before every later read of that variable, and everything the writer did before the write is visible to the reader after the read.</li>
<li><code>count++</code> on a volatile is still a read-modify-write race; use <code>AtomicInteger</code> or a lock.</li>
<li>Writes to <code>long</code> and <code>double</code> are atomic when declared volatile (otherwise tearing is allowed by the spec on some JVMs).</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
class Worker implements Runnable {
    private volatile boolean running = true;     // stop flag

    public void run() { while (running) doWork(); }
    public void stop() { running = false; }      // visible to the worker thread
}

// Safe lazy singleton: volatile prevents seeing a half-constructed instance
class Config {
    private static volatile Config instance;
    static Config get() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) instance = new Config();
            }
        }
        return instance;
    }
}
```

<h3>When to Use</h3>
<ul>
<li>Use for: status flags, safely publishing an immutable object, double-checked locking, one writer with many readers.</li>
<li>Do not use for: counters, check-then-act logic, or invariants spanning several variables (use <code>synchronized</code> or atomics).</li>
<li>Versus <code>AtomicInteger</code>: volatile gives visibility only; atomics add atomic read-modify-write via CAS.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/threading/">Learn more: Threading Deep-Dive</a></p>

</details>

---

## 6. How do you make sure a thread completes and collect its result?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use <code>Thread.join()</code> to wait for a thread, or submit a <code>Callable</code> to an <code>ExecutorService</code> and call <code>Future.get()</code> to wait and receive the value. <code>CompletableFuture</code> adds non-blocking composition.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Mechanism</th><th>Returns value</th><th>Notes</th></tr>
<tr><td><code>Thread.join()</code></td><td>No (use shared state)</td><td>Blocks until done; <code>join(ms)</code> for timeout</td></tr>
<tr><td><code>Future.get()</code></td><td>Yes</td><td>Blocks; throws <code>ExecutionException</code> wrapping task failure; <code>get(timeout)</code></td></tr>
<tr><td><code>invokeAll</code></td><td>List of Futures</td><td>Waits for all tasks</td></tr>
<tr><td><code>CountDownLatch</code></td><td>No</td><td>Wait for N events</td></tr>
<tr><td><code>CompletableFuture</code></td><td>Yes</td><td>Chain callbacks, combine, handle errors</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
ExecutorService pool = Executors.newFixedThreadPool(3);
Future<Integer> f = pool.submit(() -> compute());      // Callable<Integer>
try {
    int result = f.get(5, TimeUnit.SECONDS);            // wait with timeout
} catch (TimeoutException e) {
    f.cancel(true);
} catch (ExecutionException e) {
    handle(e.getCause());                               // the task's real exception
} finally {
    pool.shutdown();
}

Thread t = new Thread(task);
t.start();
t.join();                                               // happens-before: t's writes are now visible
```

<h3>When to Use</h3>
<ul>
<li>Plain <code>join()</code> for a handful of threads where results are shared via fields.</li>
<li><code>Future</code> for simple request/response tasks; always use a timeout in production.</li>
<li><code>CompletableFuture</code> to chain or combine tasks without blocking a thread per call.</li>
<li>Handle <code>InterruptedException</code> by restoring the flag: <code>Thread.currentThread().interrupt()</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 7. What is the difference between synchronized and ReentrantLock? What are the trade-offs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>synchronized</code> is a built-in keyword with automatic lock release; <code>ReentrantLock</code> is an explicit lock object with extra capabilities (timeouts, interruptibility, fairness, multiple conditions) that you must unlock yourself in <code>finally</code>.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Feature</th><th>synchronized</th><th>ReentrantLock</th></tr>
<tr><td>Release</td><td>Automatic on block exit or exception</td><td>Manual: <code>unlock()</code> in finally</td></tr>
<tr><td>Try without blocking</td><td>No</td><td><code>tryLock()</code>, <code>tryLock(timeout)</code></td></tr>
<tr><td>Interruptible wait</td><td>No</td><td><code>lockInterruptibly()</code></td></tr>
<tr><td>Fairness</td><td>No (unfair)</td><td>Optional: <code>new ReentrantLock(true)</code></td></tr>
<tr><td>Conditions</td><td>One wait set (<code>wait/notify</code>)</td><td>Many <code>Condition</code> objects</td></tr>
<tr><td>Scope</td><td>Block or method</td><td>Any code, across methods</td></tr>
<tr><td>Both reentrant</td><td colspan="2">Yes, same thread can re-acquire</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
private final ReentrantLock lock = new ReentrantLock();

boolean transfer() throws InterruptedException {
    if (!lock.tryLock(200, TimeUnit.MILLISECONDS)) return false;  // avoid deadlock
    try {
        move();
        return true;
    } finally {
        lock.unlock();                                            // never forget
    }
}
```

<h3>When to Use</h3>
<ul>
<li>Default to <code>synchronized</code>: simpler, no leaked locks, and heavily optimized by the JVM.</li>
<li>Use <code>ReentrantLock</code> for timeouts, interruptible acquisition, fairness, or several conditions; <code>ReadWriteLock</code>/<code>StampedLock</code> for read-heavy data.</li>
<li>Virtual threads (Java 21 to 23) could pin their carrier thread inside <code>synchronized</code> blocks while blocking; JDK 24 removed most of that pinning, and on earlier versions <code>ReentrantLock</code> avoids it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/threading/">Learn more: Threading Deep-Dive</a></p>

</details>

---

## 8. Explain the key CompletableFuture methods

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>CompletableFuture</code> (Java 8) is a future you can complete manually and compose with callbacks, so async pipelines do not block threads waiting on <code>get()</code>.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Purpose</th><th>Methods</th><th>Notes</th></tr>
<tr><td>Start</td><td><code>supplyAsync(supplier)</code>, <code>runAsync(runnable)</code>, <code>completedFuture(v)</code></td><td>Pass your own executor; default is the common pool</td></tr>
<tr><td>Transform</td><td><code>thenApply</code>, <code>thenAccept</code>, <code>thenRun</code></td><td>Map the result, consume it, or just run</td></tr>
<tr><td>Chain async step</td><td><code>thenCompose</code></td><td>Like flatMap: function returns another CompletableFuture</td></tr>
<tr><td>Combine</td><td><code>thenCombine</code>, <code>allOf</code>, <code>anyOf</code></td><td><code>allOf</code> returns <code>CompletableFuture&lt;Void&gt;</code></td></tr>
<tr><td>Errors</td><td><code>exceptionally</code>, <code>handle</code>, <code>whenComplete</code></td><td><code>handle</code> sees result or error and can map it</td></tr>
<tr><td>Timeouts (Java 9)</td><td><code>orTimeout</code>, <code>completeOnTimeout</code></td><td>Fail or default after a delay</td></tr>
<tr><td>Get result</td><td><code>get()</code> (checked), <code>join()</code> (unchecked)</td><td>Blocking</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
CompletableFuture<String> result =
    CompletableFuture.supplyAsync(() -> fetchUserId(), ioPool)
        .thenCompose(id -> CompletableFuture.supplyAsync(() -> fetchProfile(id), ioPool))
        .thenApply(Profile::displayName)
        .orTimeout(2, TimeUnit.SECONDS)
        .exceptionally(ex -> "guest");
```

<h3>When to Use</h3>
<ul>
<li>Gotcha: <code>thenApply</code> vs <code>thenApplyAsync</code>: the non-async version runs on the completing thread (or the caller if already complete); the Async version submits to a pool.</li>
<li>Use a dedicated executor for blocking I/O so you do not starve the common pool.</li>
<li>Errors surface as <code>CompletionException</code> wrapping the cause when using <code>join()</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 9. How do you call two services in parallel and wait for both to complete?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Start both calls asynchronously, then combine: <code>CompletableFuture.supplyAsync</code> for each plus <code>thenCombine</code> (two results) or <code>allOf</code> (many results). Total latency is roughly the slower call rather than the sum.</p>

<h3>Why / Context</h3>
<ul>
<li>Use a dedicated executor for blocking calls rather than the common pool.</li>
<li>Decide the failure policy: fail fast, fall back to a default, or partial results.</li>
<li>Always set timeouts so one slow service cannot hang the request.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
ExecutorService io = Executors.newFixedThreadPool(10);

CompletableFuture<User> userF =
    CompletableFuture.supplyAsync(() -> userClient.fetch(id), io).orTimeout(2, TimeUnit.SECONDS);
CompletableFuture<List<Order>> ordersF =
    CompletableFuture.supplyAsync(() -> orderClient.fetch(id), io).orTimeout(2, TimeUnit.SECONDS);

// Two results
UserSummary summary = userF.thenCombine(ordersF, UserSummary::new).join();

// Many futures, same wait
CompletableFuture.allOf(userF, ordersF).join();
User u = userF.join();                       // already complete: no extra wait
```

<ul>
<li><strong>Executor alternative:</strong> <code>invokeAll(List.of(callable1, callable2))</code> waits for all.</li>
<li><strong>Java 21+ virtual threads:</strong> <code>try (var ex = Executors.newVirtualThreadPerTaskExecutor())</code> with plain blocking calls and two <code>submit</code> calls; closing the executor waits for tasks. Structured concurrency (<code>StructuredTaskScope</code>) is a preview API.</li>
<li>Reactive stacks: <code>Mono.zip(a, b)</code>.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>When calls are independent; if the second call needs the first's result, use <code>thenCompose</code> instead.</li>
<li>Add a fallback with <code>exceptionally</code> or <code>handle</code> for optional data.</li>
<li>Cap concurrency so fan-out does not overload downstream services.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 10. How do you implement a thread-safe singleton, and why does double-checked locking need volatile?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The simplest safe options are an <code>enum</code> singleton or a static holder class (class initialization is thread-safe). If you use lazy double-checked locking (DCL), the field must be <code>volatile</code>, otherwise another thread can see a non-null reference to a partially constructed object.</p>

<h3>Why volatile is required</h3>
<p><code>instance = new Foo()</code> is roughly three steps: allocate memory, run the constructor, assign the reference. Without a happens-before edge the compiler or CPU may reorder it to allocate, assign, then construct. Thread B's unsynchronized first check sees a non-null <code>instance</code> and uses an object whose fields are not yet initialized. <code>volatile</code> forbids that reordering and guarantees visibility (reliable since the Java 5 memory model).</p>

<h3>Example / Real-World Use</h3>

```java
// 1. Enum: thread-safe, serialization-safe, reflection-safe
public enum Registry { INSTANCE; public void register(String s) { /* ... */ } }

// 2. Holder idiom: lazy, no explicit locking
public final class Config {
    private Config() {}
    private static class Holder { static final Config INSTANCE = new Config(); }
    public static Config get() { return Holder.INSTANCE; }
}

// 3. Double-checked locking
public final class Cache {
    private static volatile Cache instance;      // volatile is essential
    private Cache() {}
    public static Cache get() {
        Cache local = instance;                  // one volatile read on the fast path
        if (local == null) {
            synchronized (Cache.class) {
                local = instance;
                if (local == null) instance = local = new Cache();
            }
        }
        return local;
    }
}
```

<h3>Comparison Table</h3>
<table>
<tr><th>Approach</th><th>Lazy</th><th>Notes</th></tr>
<tr><td>Enum</td><td>On first use of the enum class</td><td>Recommended by Effective Java; cannot extend a class</td></tr>
<tr><td>Holder class</td><td>Yes</td><td>Simple, no volatile needed</td></tr>
<tr><td>DCL + volatile</td><td>Yes</td><td>Error-prone; mainly for heavy initialization</td></tr>
<tr><td><code>synchronized</code> getter</td><td>Yes</td><td>Correct but locks on every call</td></tr>
</table>

<h3>When to Use</h3>
<ul>
<li>In Spring apps prefer container-managed singleton beans over hand-written singletons (easier to test).</li>
<li>Reflection and serialization can break a classic private-constructor singleton; enum avoids both.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>

---

## 11. Concurrency recall: volatile, synchronized, Atomic, deadlock, ThreadLocal, immutability, virtual threads

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Thread safety needs both visibility and atomicity. Each tool gives a different guarantee, and the cheapest correct one wins.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Tool</th><th>Guarantees</th><th>Does not give you</th></tr>
<tr><td><code>volatile</code></td><td>Visibility and ordering of one variable</td><td>Atomic compound actions (<code>count++</code>)</td></tr>
<tr><td><code>synchronized</code> / <code>Lock</code></td><td>Mutual exclusion plus visibility</td><td>Freedom from deadlock; adds contention</td></tr>
<tr><td><code>Atomic*</code> / <code>LongAdder</code></td><td>Lock-free atomic updates via CAS</td><td>Atomicity across multiple variables</td></tr>
<tr><td><code>ThreadLocal</code></td><td>Confinement: each thread has its own copy</td><td>Sharing; leaks in thread pools unless <code>remove()</code> is called</td></tr>
<tr><td>Immutability</td><td>Safe sharing with no locks (final fields, records)</td><td>Safety if it holds references to mutable objects</td></tr>
</table>

<h3>Deadlock</h3>
<p>Four conditions must all hold (Coffman): mutual exclusion, hold and wait, no preemption, circular wait. Break any one: acquire locks in a global order, use <code>tryLock</code> with a timeout, or reduce lock scope. Diagnose with a thread dump (<code>jcmd PID Thread.print</code> reports detected deadlocks).</p>

<h3>Example / Real-World Use</h3>

```java
AtomicLong hits = new AtomicLong();
hits.incrementAndGet();                       // CAS loop, no lock

try (var ex = Executors.newVirtualThreadPerTaskExecutor()) {   // Java 21
    ex.submit(() -> callBlockingService());   // cheap threads for blocking I/O
}
```

<h3>Gotchas</h3>
<ul>
<li>Virtual threads (final in Java 21) suit blocking I/O, not CPU-bound work; do not pool them. Pinning inside <code>synchronized</code> was removed in Java 24.</li>
<li><code>ThreadLocal</code> in pools: always clear it, otherwise values leak across requests.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Flag or safe publication: <code>volatile</code>. Counters: Atomic or LongAdder. Invariants across several fields: a lock.</li>
<li>Default to immutable objects and confinement before reaching for locks.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Threads</a></p>

</details>
