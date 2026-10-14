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

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency.md">Learn more: Concurrency</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/threading.md">Learn more: Threading</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/concurrency.md">Learn more: Concurrency & Futures</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns.md">Learn more: Design Patterns</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/versions-features.md">Learn more: JVM & Garbage Collection</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/basics.md">Learn more: Java Generics</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance.md">Learn more: Performance Optimization</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns.md">Learn more: Design Patterns</a></p>

</details>

---

**See also:** [Design Patterns](../../java-patterns-advanced/design-patterns.md), [Concurrency](../../java-collections-concurrency/concurrency.md), [Interview Questions Overview](../index.md)
