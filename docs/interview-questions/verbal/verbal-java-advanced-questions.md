# Java Advanced Interview Questions

Advanced concepts: patterns, performance, frameworks, concurrency, and enterprise design.

---

## 1. What's the difference between volatile, synchronized, and Atomic classes?

<details>
<summary>View Answer</summary>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Property</th>
<th>volatile</th>
<th>synchronized</th>
<th>Atomic</th>
</tr>
<tr>
<td><strong>Purpose</strong></td>
<td>Visibility guarantee</td>
<td>Mutual exclusion (lock)</td>
<td>Atomic operations (lock-free)</td>
</tr>
<tr>
<td><strong>Lock</strong></td>
<td>None</td>
<td>Monitor lock</td>
<td>Compare-and-swap (CAS)</td>
</tr>
<tr>
<td><strong>Thread-safe</strong></td>
<td>Read/write visibility only</td>
<td>Full exclusion</td>
<td>Atomic operations only</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Fast (no locking)</td>
<td>Slow (locks)</td>
<td>Fast (lock-free, hardware)</td>
</tr>
<tr>
<td><strong>Use case</strong></td>
<td>Flag variables, stop signals</td>
<td>Shared state, compound operations</td>
<td>Counters, simple updates</td>
</tr>
</table>

<h3>volatile: Visibility Only</h3>

```java
volatile boolean shutdown = false;

// Thread 1
while (!shutdown) {  // Sees latest value
    process();
}

// Thread 2
shutdown = true;    // Visible immediately to Thread 1
```

<p>✅ Fast, but only guarantees visibility.</p>

<h3>synchronized: Mutual Exclusion</h3>

```java
int count = 0;

synchronized void increment() {
    count++;  // Atomic: read-modify-write
}
```

<p>✅ Full thread-safety, but slower (lock contention).</p>

<h3>Atomic: Lock-Free</h3>

```java
AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();  // Hardware CAS operation
```

<p>✅ Fast and atomic, best for simple operations.</p>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency</a></p>

</details>

---

## 2. What's the difference between Runnable and Callable?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr>
<th>Feature</th>
<th>Runnable</th>
<th>Callable</th>
</tr>
<tr>
<td><strong>Return value</strong></td>
<td>void (no return)</td>
<td>Returns result (&lt;T&gt;)</td>
</tr>
<tr>
<td><strong>Exception</strong></td>
<td>Cannot throw checked</td>
<td>Can throw checked</td>
</tr>
<tr>
<td><strong>Method</strong></td>
<td><code>run()</code></td>
<td><code>call()</code></td>
</tr>
<tr>
<td><strong>Executor</strong></td>
<td><code>execute()</code></td>
<td><code>submit()</code></td>
</tr>
</table>

<h3>Runnable: No Return</h3>

```java
Runnable task = () -> {
    System.out.println("Task running");
    // No return value
};

executor.execute(task);  // Fire-and-forget
```

<h3>Callable: With Return</h3>

```java
Callable<Integer> task = () -> {
    return 42;  // Returns result
};

Future<Integer> future = executor.submit(task);
Integer result = future.get();  // Block and get result
```

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/threading/">Learn more: Threading</a></p>

</details>

---

## 3. Explain Future vs CompletableFuture

<details>
<summary>View Answer</summary>

<h3>Future: Blocking</h3>

<p><strong>Represents result of async computation, but getting result blocks.</strong></p>

```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Result";
});

String result = future.get();  // ❌ Blocks here until ready
```

<p><strong>Problems:</strong> Blocking, limited composition, no callbacks</p>

<h3>CompletableFuture: Non-Blocking</h3>

<p><strong>Non-blocking chains and callbacks.</strong></p>

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Step 1";
})
.thenApply(s -> s + " → Step 2")      // Chain operations
.thenAccept(s -> System.out.println(s));  // Callback (non-blocking)

// Main thread continues, doesn't wait
```

<h3>Comparison Table</h3>

<table>
<tr>
<th>Feature</th>
<th>Future</th>
<th>CompletableFuture</th>
</tr>
<tr>
<td><strong>Blocking</strong></td>
<td>Yes (.get())</td>
<td>Optional</td>
</tr>
<tr>
<td><strong>Chaining</strong></td>
<td>Limited</td>
<td>Rich (.thenApply, .thenCombine)</td>
</tr>
<tr>
<td><strong>Callbacks</strong></td>
<td>No</td>
<td>Yes (.thenAccept, .whenComplete)</td>
</tr>
<tr>
<td><strong>Error handling</strong></td>
<td>Exception on .get()</td>
<td>.exceptionally, .handle</td>
</tr>
<tr>
<td><strong>Combining</strong></td>
<td>Manual</td>
<td>Built-in (.thenCombine, .anyOf, .allOf)</td>
</tr>
</table>

<h3>CompletableFuture Methods</h3>

<ul>
<li><strong>.thenApply():</strong> Transform result</li>
<li><strong>.thenAccept():</strong> Consume result (no return)</li>
<li><strong>.thenCombine():</strong> Combine two futures</li>
<li><strong>.exceptionally():</strong> Handle errors</li>
<li><strong>.allOf():</strong> Wait for all futures</li>
<li><strong>.anyOf():</strong> Wait for first future</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency/">Learn more: Concurrency & Futures</a></p>

</details>

---

## 4. What are design patterns and when to use them?

<details>
<summary>View Answer</summary>

<h3>Categories of Design Patterns</h3>

<h4>Creational: Object Creation</h4>

<ul>
<li><strong>Singleton:</strong> One instance per application</li>
<li><strong>Factory:</strong> Create objects without specifying exact classes</li>
<li><strong>Builder:</strong> Construct complex objects step-by-step</li>
</ul>

<h4>Structural: Object Composition</h4>

<ul>
<li><strong>Decorator:</strong> Add behavior to objects dynamically</li>
<li><strong>Adapter:</strong> Make incompatible interfaces compatible</li>
<li><strong>Proxy:</strong> Provide placeholder for another object</li>
</ul>

<h4>Behavioral: Object Collaboration</h4>

<ul>
<li><strong>Observer:</strong> Notify multiple objects of state changes</li>
<li><strong>Strategy:</strong> Encapsulate interchangeable algorithms</li>
<li><strong>Template Method:</strong> Define algorithm skeleton, subclasses fill details</li>
</ul>

<h3>When to Use</h3>

<table>
<tr>
<th>Problem</th>
<th>Pattern</th>
<th>Example</th>
</tr>
<tr>
<td>Need single instance</td>
<td>Singleton</td>
<td>Logger, Database connection</td>
</tr>
<tr>
<td>Create families of objects</td>
<td>Factory</td>
<td>Database drivers (MySQL, PostgreSQL)</td>
</tr>
<tr>
<td>Complex object construction</td>
<td>Builder</td>
<td>HTTP request with many options</td>
</tr>
<tr>
<td>Add behavior at runtime</td>
<td>Decorator</td>
<td>Compression, encryption wrappers</td>
</tr>
<tr>
<td>Make incompatible things work</td>
<td>Adapter</td>
<td>Legacy API integration</td>
</tr>
<tr>
<td>Notify multiple observers</td>
<td>Observer</td>
<td>Event handling, MVC</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 5. How does garbage collection work?

<details>
<summary>View Answer</summary>

<h3>GC Process</h3>

<ol>
<li><strong>Mark:</strong> Identify reachable objects from GC roots</li>
<li><strong>Sweep:</strong> Free unreachable objects</li>
<li><strong>Compact:</strong> Defragment heap (some algorithms)</li>
</ol>

<h3>GC Algorithms</h3>

<table>
<tr>
<th>Algorithm</th>
<th>Latency</th>
<th>Throughput</th>
<th>Best For</th>
</tr>
<tr>
<td><strong>Serial GC</strong></td>
<td>High pauses</td>
<td>Good</td>
<td>Single-core, small heaps</td>
</tr>
<tr>
<td><strong>Parallel GC</strong></td>
<td>Medium pauses</td>
<td>Best</td>
<td>Batch processing</td>
</tr>
<tr>
<td><strong>G1GC</strong></td>
<td>Low pauses</td>
<td>Good</td>
<td>Large heaps, low latency</td>
</tr>
<tr>
<td><strong>ZGC</strong></td>
<td>Very low (&lt;10ms)</td>
<td>Good</td>
<td>Ultra-low latency apps</td>
</tr>
</table>

<h3>Generational Hypothesis</h3>

<p><strong>Most objects die young; old objects rarely become garbage.</strong></p>

```
Young Generation (Frequent GC)
├─ Eden
├─ Survivor 1
└─ Survivor 2

Old Generation (Infrequent GC)
```

<p>✅ Optimize for young generation garbage collection.</p>

<p>📚 <a href="/anandu-interview-playbook/java-core/jvm-internals/">Learn more: JVM & Garbage Collection</a></p>

</details>

---

## 6. What's the difference between type erasure and generics?

<details>
<summary>View Answer</summary>

<h3>Generics: Compile-Time Type Safety</h3>

```java
List<String> list = new ArrayList<>();
list.add("Hello");
String s = list.get(0);  // No cast needed, type-safe
```

<h3>Type Erasure: Runtime Behavior</h3>

<p><strong>Generics info ERASED at runtime.</strong></p>

```java
// At compile-time (generic)
List<String> stringList = new ArrayList<>();

// At runtime (erased)
List stringList = new ArrayList();  // Type info gone!
```

<h3>Implications</h3>

```java
// ❌ INVALID: Runtime doesn't know generic type
List<String> list = new ArrayList<>();
if (list instanceof List<String>) { }  // Compile error

// ✅ Valid: Can check raw type
if (list instanceof List) { }
```

<h3>Why Type Erasure?</h3>

<ul>
<li>Backward compatibility with pre-Java 5 code</li>
<li>Reduce runtime memory (no type info stored)</li>
</ul>

<h3>PECS Principle: Producer Extends, Consumer Super</h3>

```java
// List<String> producer
List<? extends String> producer = list;

// List accepts String consumer
List<? super String> consumer = list;
```

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Generics</a></p>

</details>

---

## 7. How do you identify and fix performance bottlenecks?

<details>
<summary>View Answer</summary>

<h3>Identification Steps</h3>

<ol>
<li><strong>Profile:</strong> Measure CPU, memory, I/O usage</li>
<li><strong>Find hotspot:</strong> Identify slowest method/component</li>
<li><strong>Measure baseline:</strong> Before/after comparison</li>
<li><strong>Iterate:</strong> Fix one issue at a time</li>
</ol>

<h3>Profiling Tools</h3>

<ul>
<li><strong>JProfiler:</strong> CPU/memory profiling</li>
<li><strong>YourKit:</strong> Advanced performance analysis</li>
<li><strong>Async Profiler:</strong> Low-overhead sampling</li>
<li><strong>JMH:</strong> Microbenchmarking</li>
</ul>

<h3>Common Bottlenecks & Fixes</h3>

<table>
<tr>
<th>Issue</th>
<th>Symptoms</th>
<th>Fix</th>
</tr>
<tr>
<td><strong>GC pauses</strong></td>
<td>High latency spikes</td>
<td>Tune heap, use low-latency GC (ZGC)</td>
</tr>
<tr>
<td><strong>Lock contention</strong></td>
<td>High thread wait time</td>
<td>Use ConcurrentHashMap, lock-free structures</td>
</tr>
<tr>
<td><strong>Database queries</strong></td>
<td>Slow response time</td>
<td>Add indexes, optimize SQL, cache</td>
</tr>
<tr>
<td><strong>N+1 problem</strong></td>
<td>Many queries, slow</td>
<td>Use JOIN FETCH, entity graphs</td>
</tr>
<tr>
<td><strong>Memory leaks</strong></td>
<td>Growing heap, OOM</td>
<td>Profile, fix reference leaks</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Performance Optimization</a></p>

</details>

---

## 8. Explain the decorator pattern with examples

<details>
<summary>View Answer</summary>

<h3>Decorator Pattern: Add Behavior Dynamically</h3>

<p><strong>Wrap objects to add behavior without modifying original class.</strong></p>

<h3>Example: Coffee Shop</h3>

```java
// Base interface
interface Coffee {
    double cost();
    String description();
}

// Concrete implementation
class SimpleCoffee implements Coffee {
    public double cost() { return 2.0; }
    public String description() { return "Simple Coffee"; }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    public CoffeeDecorator(Coffee coffee) { this.coffee = coffee; }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) { super(coffee); }
    public double cost() { return coffee.cost() + 0.5; }
    public String description() { return coffee.description() + ", Milk"; }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) { super(coffee); }
    public double cost() { return coffee.cost() + 0.2; }
    public String description() { return coffee.description() + ", Sugar"; }
}
```

<h3>Usage</h3>

```java
Coffee coffee = new SimpleCoffee();  // $2.0
coffee = new MilkDecorator(coffee);   // $2.5
coffee = new SugarDecorator(coffee);  // $2.7

System.out.println(coffee.description());  // Simple Coffee, Milk, Sugar
System.out.println(coffee.cost());         // 2.7
```

<h3>Real-World Examples</h3>

<ul>
<li><strong>Java I/O:</strong> InputStream → BufferedInputStream → DataInputStream</li>
<li><strong>Spring:</strong> @Transactional decorator on methods</li>
<li><strong>Compression:</strong> FileOutputStream → GZIPOutputStream</li>
</ul>

<h3>Decorator vs Inheritance</h3>

<p><strong>Decorator:</strong> Combine at runtime, flexible, multiple decorators</p>
<p><strong>Inheritance:</strong> Fixed at compile-time, rigid class hierarchy</p>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 9. How do you diagnose and solve an OutOfMemoryError?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>First read the message after <code>OutOfMemoryError</code> to learn which area is exhausted, capture a heap dump, find what is retaining memory, and fix the root cause (leak or sizing). Raising <code>-Xmx</code> is a stopgap, not a fix, unless the workload truly needs more.</p>

<h3>Why / Context</h3>
<table>
<tr><th>Message</th><th>Meaning</th><th>Typical fix</th></tr>
<tr><td>Java heap space</td><td>Heap full of live objects</td><td>Fix leak, reduce data held, raise <code>-Xmx</code></td></tr>
<tr><td>GC overhead limit exceeded</td><td>GC spending nearly all time reclaiming almost nothing</td><td>Same as heap space</td></tr>
<tr><td>Metaspace</td><td>Too many loaded classes (often classloader leaks, redeploys)</td><td>Fix classloader leak; <code>-XX:MaxMetaspaceSize</code></td></tr>
<tr><td>unable to create native thread</td><td>OS thread/native memory limit</td><td>Smaller thread pools, smaller <code>-Xss</code>, virtual threads</td></tr>
<tr><td>Direct buffer memory</td><td>Off-heap NIO buffers</td><td>Release buffers, <code>-XX:MaxDirectMemorySize</code></td></tr>
</table>

<h3>Example / Real-World Use</h3>

```bash
# Dump automatically when the error occurs
java -Xms512m -Xmx2g -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/dumps/ -jar app.jar

# Dump a running process
jmap -dump:live,format=b,file=heap.hprof <pid>
```

<ul>
<li>Open the dump in Eclipse MAT or VisualVM; check the dominator tree and "path to GC roots" for the biggest retainers.</li>
<li>Typical culprits: unbounded caches or static collections, unclosed resources, large result sets loaded into memory, ThreadLocals in pools, huge request bodies.</li>
<li>In containers set <code>-XX:MaxRAMPercentage</code> so the heap respects the memory limit; otherwise the OS kills the process (OOMKilled), which is not a Java OOME.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Resize the heap only after confirming memory is genuinely needed; set <code>-Xms</code> equal to <code>-Xmx</code> on servers to avoid resizing pauses.</li>
<li>Prefer streaming/pagination, bounded caches (size or TTL), and fewer retained objects.</li>
<li>Use GC logs (<code>-Xlog:gc*</code>) to see whether memory grows steadily (leak) or spikes (burst load).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/jvm-internals/">Learn more: JVM Internals</a></p>

</details>

---

## 10. Explain Java's memory management model: heap, stack, garbage collection, and optimization

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Each thread has its own stack (frames with locals and references); all objects live on the shared heap, which the garbage collector reclaims automatically. Class metadata lives in Metaspace (native memory).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Area</th><th>Holds</th><th>Scope</th><th>Failure</th></tr>
<tr><td>Stack</td><td>Frames, primitives, references</td><td>Per thread, freed on method return</td><td>StackOverflowError</td></tr>
<tr><td>Heap</td><td>Objects, arrays, String pool</td><td>Shared; young and old generations</td><td>OutOfMemoryError: heap</td></tr>
<tr><td>Metaspace</td><td>Class metadata (replaced PermGen in Java 8)</td><td>Shared, native memory</td><td>OutOfMemoryError: Metaspace</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li><strong>Generational GC:</strong> most objects die young, so new objects go to Eden, survivors move between survivor spaces, and long-lived ones are promoted to the old generation. Minor GCs are cheap; full GCs are expensive.</li>
<li><strong>Collectors:</strong> G1 (default since Java 9), ZGC and Shenandoah (low pause), Parallel (throughput), Serial (small heaps).</li>
<li>An object is eligible for GC when it is unreachable from GC roots (thread stacks, statics, JNI refs); reference counting is not used.</li>
</ul>

<h3>Example / Real-World Use</h3>

```bash
java -Xms4g -Xmx4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xlog:gc*:file=gc.log -jar app.jar
```

<ul>
<li>Optimization: reduce allocation rate (avoid needless boxing and temporary objects), reuse expensive objects (pools for connections and threads, not for cheap objects since modern allocation is fast), size generations only after measuring.</li>
<li>Use weak or soft references for caches; always bound caches.</li>
<li>Escape analysis may allocate short-lived objects on the stack or eliminate them entirely.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Latency-sensitive services: G1 or ZGC with a pause target; batch jobs: Parallel GC.</li>
<li>Tune from GC logs and profiling, not guesses. Default settings are good for most applications.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/jvm-internals/">Learn more: JVM Internals</a></p>

</details>

---

## 11. What causes a memory leak in Java?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Despite garbage collection, a leak occurs when objects are no longer needed but remain reachable from a GC root, so they can never be collected. Heap usage keeps growing until <code>OutOfMemoryError</code>.</p>

<h3>Why / Context</h3>
<table>
<tr><th>Cause</th><th>Why it leaks</th><th>Fix</th></tr>
<tr><td>Static or long-lived collections</td><td>Entries added, never removed (caches, registries)</td><td>Bounded cache with eviction (size/TTL), remove on lifecycle end</td></tr>
<tr><td>Unclosed resources</td><td>Streams, connections, and statements hold native and heap memory</td><td>try-with-resources</td></tr>
<tr><td>Listeners and callbacks</td><td>Registered but never deregistered</td><td>Unregister, use weak references</td></tr>
<tr><td>ThreadLocal in thread pools</td><td>Pool threads live forever, so values never go away</td><td><code>remove()</code> in a finally block</td></tr>
<tr><td>Bad equals/hashCode or mutated keys</td><td>Entries in HashMap/HashSet cannot be found or removed</td><td>Immutable keys, correct contract</td></tr>
<tr><td>Inner classes and lambdas</td><td>Hold an implicit reference to the outer instance</td><td>Use static nested classes where possible</td></tr>
<tr><td>Classloader leaks</td><td>Redeployed apps keep old classes (Metaspace growth)</td><td>Clean up threads, drivers, ThreadLocals on undeploy</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
class Registry {
    private static final Map<String, Session> SESSIONS = new HashMap<>(); // grows forever

    static void open(String id, Session s) { SESSIONS.put(id, s); }
    // missing: remove(id) on logout/expiry
}
```

<ul>
<li>Detect: heap usage after full GC trends upward (GC logs, JMX), then compare two heap dumps in MAT or use JFR's old-object sample.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Design caches with a maximum size and expiry (for example Caffeine) instead of raw maps.</li>
<li>Prefer <code>WeakHashMap</code> only when entries should vanish once keys are unreferenced elsewhere.</li>
<li>Do not blame GC first: most "leaks" are application-held references.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/jvm-internals/">Learn more: JVM Internals</a></p>

</details>

---

**See also:** [Design Patterns](../../java-patterns-advanced/design-patterns.md), [Concurrency](../../java-collections-concurrency/concurrency.md), [Interview Preparation](../index.md)
