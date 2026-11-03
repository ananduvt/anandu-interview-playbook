# Java Collections and Streams Interview Questions

HashMap internals, choosing between collection types, and Stream API behavior.

---

## 1. How does HashMap work, and what happens when two keys have the same hash code?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>HashMap stores entries in an array of buckets. The key's <code>hashCode()</code> is spread (<code>h ^ (h &gt;&gt;&gt; 16)</code>) and masked to a bucket index. Keys that land in the same bucket (collision) are chained; lookups walk the chain comparing with <code>equals()</code>.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>put:</strong> compute index; empty bucket gets a node; otherwise compare hash then <code>equals</code> to replace an existing key or append a new node.</li>
<li><strong>Collisions:</strong> bucket is a linked list; since Java 8, when a bucket reaches 8 nodes and the table has at least 64 buckets, it becomes a red-black tree (worst case O(log n) instead of O(n)); it reverts to a list at 6 or fewer.</li>
<li><strong>Resize:</strong> default capacity 16, load factor 0.75; at capacity x 0.75 entries the table doubles and entries are redistributed.</li>
<li>Same hash code does not mean equal keys; the contract is: equal objects must have equal hash codes, not the reverse.</li>
<li>One <code>null</code> key allowed; not thread-safe; iteration order is unspecified.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
Map<String, Integer> m = new HashMap<>(64);   // presize to avoid resizes
m.put("Aa", 1);
m.put("BB", 2);                                // "Aa" and "BB" share hashCode 2112
System.out.println(m.get("BB"));               // 2: equals() distinguishes them
```

<h3>When to Use</h3>
<ul>
<li>Average O(1) get/put; treat O(n)/O(log n) as the worst case under heavy collisions.</li>
<li>Always override <code>equals</code> and <code>hashCode</code> together for custom keys; tree bins work best if keys are also <code>Comparable</code>.</li>
<li>Pre-size using expected size / 0.75 + 1.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 2. Can HashMap keys be mutable? Why or why not?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Technically yes, but it is dangerous. If a key's fields used in <code>hashCode()</code>/<code>equals()</code> change after insertion, the entry sits in the wrong bucket and can no longer be found or removed, which also leaks memory.</p>

<h3>Why / Context</h3>
<ul>
<li>The map computes the bucket once at insertion and never rehashes existing keys (except when resizing, which reuses the stored hash).</li>
<li>After mutation <code>get(key)</code> looks in the new bucket and misses; <code>containsKey</code> returns false while the entry still occupies memory and shows up in iteration.</li>
<li>This is why <code>String</code>, <code>Integer</code>, enums, and records of immutable components make ideal keys.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
class Point { int x; Point(int x) { this.x = x; }
    @Override public boolean equals(Object o) { return o instanceof Point p && p.x == x; }
    @Override public int hashCode() { return Integer.hashCode(x); } }

Map<Point, String> map = new HashMap<>();
Point k = new Point(1);
map.put(k, "a");
k.x = 2;                       // mutate the key
map.get(k);                    // null: looks in bucket for hash 2
map.get(new Point(1));         // null: bucket matches but equals() now fails
map.size();                    // 1: entry is still there, unreachable
```

<h3>When to Use</h3>
<ul>
<li>Make keys immutable (final fields, records).</li>
<li>If a mutable key is unavoidable, do not change fields that take part in <code>equals</code>/<code>hashCode</code> while it is in the map; or remove, mutate, then re-insert.</li>
<li>Same rule applies to <code>HashSet</code> elements.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 3. Compare List, Set, and Map implementations including time and space complexity

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Choose by access pattern: <code>ArrayList</code> for indexed reads, <code>LinkedList</code>/<code>ArrayDeque</code> for queues, <code>HashSet</code>/<code>HashMap</code> for fast lookup, <code>Tree*</code> for sorted order, <code>Linked*</code> for insertion order.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Structure</th><th>get / contains</th><th>add / put</th><th>remove</th><th>Ordering</th><th>Notes</th></tr>
<tr><td>ArrayList</td><td>O(1) by index; O(n) contains</td><td>O(1) amortized at end; O(n) middle</td><td>O(n)</td><td>Insertion</td><td>Compact array, cache friendly</td></tr>
<tr><td>LinkedList</td><td>O(n)</td><td>O(1) at ends or at an iterator</td><td>O(1) at known node</td><td>Insertion</td><td>Extra node object per element</td></tr>
<tr><td>HashSet / HashMap</td><td>O(1) average</td><td>O(1) average</td><td>O(1) average</td><td>None</td><td>Needs good hashCode</td></tr>
<tr><td>LinkedHashSet / LinkedHashMap</td><td>O(1)</td><td>O(1)</td><td>O(1)</td><td>Insertion (or access order)</td><td>Good for LRU caches</td></tr>
<tr><td>TreeSet / TreeMap</td><td>O(log n)</td><td>O(log n)</td><td>O(log n)</td><td>Sorted</td><td>Range queries, floor/ceiling</td></tr>
<tr><td>PriorityQueue</td><td>peek O(1)</td><td>O(log n)</td><td>poll O(log n)</td><td>Heap order</td><td>Not sorted iteration</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li>Space is O(n) for all; hash structures keep spare buckets (load factor), linked structures pay per-node overhead.</li>
<li>Sets are backed by maps (<code>HashSet</code> uses a <code>HashMap</code>), so they share the same costs.</li>
<li>Set and Map keys depend on <code>equals</code>/<code>hashCode</code> (hash) or <code>compareTo</code> (tree).</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Default to <code>ArrayList</code> and <code>HashMap</code>; switch only for a measured need (ordering, sorted views, stack/queue use).</li>
<li>Use <code>EnumMap</code>/<code>EnumSet</code> for enum keys; <code>ArrayDeque</code> for stacks and queues.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 4. What is the difference between ArrayList and LinkedList, and when would you choose LinkedList?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>ArrayList</code> is a resizable array: fast random access, slow middle insert/delete. <code>LinkedList</code> is a doubly linked list: O(1) insert/delete at a known node or at the ends, but O(n) access by index. In practice <code>ArrayList</code> wins in most workloads.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Operation</th><th>ArrayList</th><th>LinkedList</th></tr>
<tr><td>get(i)</td><td>O(1)</td><td>O(n)</td></tr>
<tr><td>add at end</td><td>O(1) amortized</td><td>O(1)</td></tr>
<tr><td>add/remove at head</td><td>O(n) shift</td><td>O(1)</td></tr>
<tr><td>add/remove in middle</td><td>O(n) shift (fast memmove)</td><td>O(n) to find, O(1) to link</td></tr>
<tr><td>Memory</td><td>Compact; spare capacity</td><td>Two pointers plus header per element</td></tr>
<tr><td>CPU cache</td><td>Excellent (contiguous)</td><td>Poor (pointer chasing)</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li>Middle insertion in a <code>LinkedList</code> is only O(1) when you already hold the position (via <code>ListIterator</code>); finding it costs O(n), so it often loses to <code>ArrayList</code>'s array copy.</li>
<li><code>ArrayList</code> grows by about 1.5x; presize with <code>new ArrayList&lt;&gt;(n)</code>.</li>
<li><code>LinkedList</code> implements both <code>List</code> and <code>Deque</code>.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li><code>LinkedList</code>: frequent add/remove at both ends or via iterators while traversing, with little random access. Even then <code>ArrayDeque</code> is usually faster for queues and stacks.</li>
<li><code>ArrayList</code>: everything else, especially read-heavy and iteration-heavy code.</li>
<li>Benchmark before choosing <code>LinkedList</code> for performance reasons.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 5. How do you decide between HashMap and TreeMap in a high-performance scenario?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use <code>HashMap</code> for fastest unordered lookup (O(1) average). Use <code>TreeMap</code> only when you need sorted keys or navigation (range queries, first/last, floor/ceiling), accepting O(log n) per operation.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>HashMap</th><th>TreeMap</th></tr>
<tr><td>Get/put/remove</td><td>O(1) average</td><td>O(log n) guaranteed</td></tr>
<tr><td>Ordering</td><td>None</td><td>Sorted by natural order or Comparator</td></tr>
<tr><td>Navigation</td><td>No</td><td><code>firstKey</code>, <code>floorKey</code>, <code>subMap</code>, <code>headMap</code>, <code>descendingMap</code></td></tr>
<tr><td>Null keys</td><td>One allowed</td><td>Not with natural ordering</td></tr>
<tr><td>Key requirement</td><td>Good <code>hashCode</code>/<code>equals</code></td><td>Consistent <code>compareTo</code>/Comparator</td></tr>
<tr><td>Implementation</td><td>Bucket array, tree bins on collisions</td><td>Red-black tree</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li>HashMap's worst case degrades under bad hashes (O(log n) with tree bins since Java 8) and resizing causes occasional pauses; presize to avoid them.</li>
<li>TreeMap has predictable latency and no hash-quality dependence, but more pointer chasing and allocation per entry.</li>
<li>TreeMap uses the comparator, not <code>equals</code>, to decide key equality; an inconsistent comparator loses entries.</li>
<li>Middle ground: <code>LinkedHashMap</code> for insertion/access order with hash speed; sort once at the end (<code>new TreeMap&lt;&gt;(map)</code> or a sorted stream) if you only need ordered output.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>HashMap: caches, counters, id lookups (the default).</li>
<li>TreeMap: leaderboards, time-window queries, price ladders, "next higher key" logic.</li>
<li>Concurrent sorted needs: <code>ConcurrentSkipListMap</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 6. What are the differences between ConcurrentHashMap and a synchronized Hashtable, and how do you use concurrent data structures?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>Hashtable</code> (legacy) locks the entire map on every method, so threads serialize. <code>ConcurrentHashMap</code> uses lock-free reads, CAS for empty buckets, and per-bucket <code>synchronized</code> locks (Java 8+; earlier versions used segment locks), so many threads proceed in parallel.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Hashtable</th><th>ConcurrentHashMap</th></tr>
<tr><td>Locking</td><td>One lock for the whole map</td><td>Fine-grained (per bin) plus CAS</td></tr>
<tr><td>Reads</td><td>Blocked by writers</td><td>Non-blocking</td></tr>
<tr><td>Null keys/values</td><td>Not allowed</td><td>Not allowed</td></tr>
<tr><td>Iterators</td><td>Fail-fast</td><td>Weakly consistent, never throw <code>ConcurrentModificationException</code></td></tr>
<tr><td>Atomic compound ops</td><td>No (external sync needed)</td><td><code>putIfAbsent</code>, <code>compute</code>, <code>merge</code></td></tr>
<tr><td>Status</td><td>Obsolete</td><td>Preferred</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
ConcurrentHashMap<String, LongAdder> hits = new ConcurrentHashMap<>();
hits.computeIfAbsent("home", k -> new LongAdder()).increment();   // atomic per key

ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();
counts.merge("home", 1, Integer::sum);                            // atomic increment

// BROKEN even on a thread-safe map: check-then-act race
if (!counts.containsKey("x")) counts.put("x", 1);
```

<h3>When to Use</h3>
<ul>
<li>Shared caches, counters, registries. Use the atomic methods for compound operations; the whole check-then-act sequence is not atomic otherwise.</li>
<li>Other concurrent structures: <code>CopyOnWriteArrayList</code> (read-mostly), <code>ConcurrentLinkedQueue</code>, <code>BlockingQueue</code> (producer-consumer), <code>ConcurrentSkipListMap</code> (sorted).</li>
<li>Size is only an estimate under concurrent updates; avoid relying on it for logic.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 7. What is the difference between sequential and parallel streams? When is a parallel stream appropriate in production?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A sequential stream processes elements in one thread. A parallel stream splits the source (Spliterator) and processes chunks on the common <code>ForkJoinPool</code>, then combines the results. It helps only for CPU-bound work on large, cheaply splittable data.</p>

<h3>Why / Context</h3>
<ul>
<li>Overhead (splitting, scheduling, merging) outweighs gains for small data or trivial operations.</li>
<li>Good sources split well: <code>ArrayList</code>, arrays, <code>IntStream.range</code>. Poor: <code>LinkedList</code>, <code>Stream.iterate</code>, I/O-based sources.</li>
<li>All parallel streams share the common pool (default parallelism is cores minus one), so a blocking call in one stream (database, HTTP) can starve every other parallel stream in the JVM, including web request handling.</li>
<li>Operations must be stateless, non-interfering, and associative (for <code>reduce</code>); never mutate shared state in <code>forEach</code>.</li>
<li><code>forEachOrdered</code>, <code>limit</code>, and <code>findFirst</code> constrain parallelism; <code>findAny</code> is cheaper.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
long primes = IntStream.rangeClosed(2, 5_000_000)
        .parallel()                         // CPU-bound, splits cheaply
        .filter(Main::isPrime)
        .count();

// Bad: shared mutable state in a parallel stream
List<Integer> out = new ArrayList<>();
list.parallelStream().forEach(out::add);    // race condition
// Good: use a collector
List<Integer> ok = list.parallelStream().map(x -> x * 2).toList();
```

<h3>When to Use</h3>
<ul>
<li>Use when: large dataset, CPU-heavy per-element work, no blocking, and a measured speedup (JMH).</li>
<li>Avoid in web servers with shared load, for I/O work (use an executor or <code>CompletableFuture</code> or virtual threads), and for small collections.</li>
<li>For isolation, run inside your own <code>ForkJoinPool</code>, though the stream's use of a custom pool is an implementation detail.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/streams/">Learn more: Streams</a></p>

</details>

---

## 8. What is the difference between map and flatMap in streams?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>map</code> transforms each element to exactly one output element (1 to 1). <code>flatMap</code> transforms each element into a stream (or Optional) and flattens all of them into a single stream (1 to many).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>map</th><th>flatMap</th></tr>
<tr><td>Function returns</td><td>A value <code>R</code></td><td>A <code>Stream&lt;R&gt;</code></td></tr>
<tr><td>Result</td><td><code>Stream&lt;R&gt;</code> (may nest if R is a collection)</td><td>Flattened <code>Stream&lt;R&gt;</code></td></tr>
<tr><td>Cardinality</td><td>1 to 1</td><td>1 to 0..n</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
List<List<String>> nested = List.of(List.of("a", "b"), List.of("c"));

nested.stream().map(List::stream).toList();          // List<Stream<String>>: not what you want
nested.stream().flatMap(List::stream).toList();      // [a, b, c]

// Split sentences into distinct words
sentences.stream()
         .flatMap(s -> Arrays.stream(s.split("\\s+")))
         .distinct()
         .toList();

// Optional chain
Optional<String> city = user.flatMap(User::address).map(Address::city);
```

<h3>When to Use</h3>
<ul>
<li><code>map</code>: convert type or extract a field.</li>
<li><code>flatMap</code>: nested collections, splitting, optional chaining, or producing zero results for some elements (return <code>Stream.empty()</code>).</li>
<li>Variants: <code>flatMapToInt</code>, and <code>mapMulti</code> (Java 16) when you want to emit elements without creating small intermediate streams.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/streams/">Learn more: Streams</a></p>

</details>

---

## 9. Java recall: HashMap internals, ConcurrentHashMap vs synchronizedMap, lazy streams, Optional, records

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A quick-reference of the Java facts interviewers probe most often, with the one-line reason behind each.</p>

<h3>Key Facts</h3>
<table>
<tr><th>Topic</th><th>Remember</th></tr>
<tr><td>HashMap internals</td><td>Array of buckets indexed by <code>(n-1) &amp; hash</code>; collisions chain in a list. Default capacity 16, load factor 0.75: resize (double) when size exceeds capacity x 0.75. A bucket treeifies at 8 entries only if the table has at least 64 buckets (otherwise it resizes first); it reverts to a list at 6.</td></tr>
<tr><td>ConcurrentHashMap vs <code>synchronizedMap</code></td><td>CHM (Java 8+) uses CAS plus locking on the head node of a bucket, so reads are mostly lock-free and writers to different buckets do not block each other. <code>synchronizedMap</code> uses one mutex for everything and iteration still needs manual synchronization. CHM rejects null keys and values.</td></tr>
<tr><td>Streams</td><td>Intermediate operations are lazy; nothing runs until a terminal operation. A stream can be consumed once (a second use throws <code>IllegalStateException</code>).</td></tr>
<tr><td>Optional</td><td>A return type for "might be absent". Avoid <code>.get()</code> without a check; prefer <code>orElse</code>, <code>orElseGet</code>, <code>orElseThrow</code>, <code>map</code>. Not for fields or parameters.</td></tr>
<tr><td>Records (Java 16)</td><td>Immutable data carriers with generated constructor, accessors, <code>equals</code>, <code>hashCode</code>, <code>toString</code>; good DTOs. Shallowly immutable: a <code>List</code> component can still be mutated.</td></tr>
<tr><td>Spring</td><td>Constructor injection (final fields, easy tests, fails fast). Singleton beans must be stateless. <code>@Transactional</code> rolls back on unchecked exceptions and <code>Error</code> by default, not checked ones.</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
record UserDto(String id, String name) {}

String name = Optional.ofNullable(findUser(id))
        .map(UserDto::name)
        .orElse("unknown");

var stream = users.stream().filter(u -> u.active());   // nothing executed yet
long n = stream.count();                               // terminal op runs the pipeline
// stream.count();   // IllegalStateException: already operated upon
```

<h3>When to Use</h3>
<ul>
<li>Use these as a last-minute checklist; for each item be ready to explain the "why" and one gotcha.</li>
<li>Pick CHM for shared maps; use <code>computeIfAbsent</code> or <code>merge</code> for atomic updates.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections</a></p>

</details>

---

## 10. How would you assess and improve the time complexity of a slow search operation?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Measure first, then analyse the algorithm with Big-O, then pick a data structure that matches the access pattern, and prove the gain with a benchmark.</p>

<h3>Why / Context</h3>
<ol>
<li><strong>Profile:</strong> confirm the search is the real bottleneck (JFR, async-profiler) and note input size and call frequency.</li>
<li><strong>Analyse:</strong> count the dominant loop and nested work. A linear scan inside a loop is O(n x m).</li>
<li><strong>Choose a structure:</strong> trade space or write cost for faster reads.</li>
<li><strong>Benchmark:</strong> use JMH with realistic data; Big-O ignores constants and cache effects, so small n can favor arrays.</li>
</ol>

<h3>Comparison Table</h3>
<table>
<tr><th>Need</th><th>Structure</th><th>Search cost</th></tr>
<tr><td>Unsorted, one lookup</td><td>List scan</td><td>O(n)</td></tr>
<tr><td>Repeated lookup by key</td><td><code>HashMap</code> / <code>HashSet</code></td><td>O(1) average</td></tr>
<tr><td>Sorted data, range queries</td><td>Sorted array + binary search, <code>TreeMap</code></td><td>O(log n)</td></tr>
<tr><td>Prefix search</td><td>Trie</td><td>O(key length)</td></tr>
<tr><td>Huge data, disk or DB</td><td>Index (B-tree), inverted index</td><td>O(log n) or better</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
// O(n * m): list.contains is a linear scan
for (Order o : orders) if (blocked.contains(o.customerId())) flag(o);

// O(n + m): build a hash set once
Set<String> blockedSet = new HashSet<>(blocked);
for (Order o : orders) if (blockedSet.contains(o.customerId())) flag(o);
```

<h3>When to Use</h3>
<ul>
<li>Hash lookups when keys have good <code>hashCode</code> and order is irrelevant; sorted structures when you need order or ranges.</li>
<li>Other levers: caching results, pre-sorting once, early exit, batching, and an index instead of scanning in the database.</li>
<li>Gotcha: premature optimisation. Do not trade clarity for a gain you have not measured.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>

---

## 11. Which coding-round patterns should you know and when does each apply?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Most coding-round problems reduce to a handful of patterns. Recognise the signal in the problem statement and the pattern usually follows.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Pattern</th><th>Signal</th><th>Typical cost</th></tr>
<tr><td>Hash map / set</td><td>Count, dedupe, "have I seen this", complement lookup (two sum)</td><td>O(n)</td></tr>
<tr><td>Two pointers</td><td>Sorted array, pair or palindrome checks, in-place partition</td><td>O(n)</td></tr>
<tr><td>Sliding window</td><td>Longest or shortest contiguous subarray or substring with a condition</td><td>O(n)</td></tr>
<tr><td>Binary search</td><td>Sorted data, or a monotonic yes/no answer ("minimum capacity that works")</td><td>O(log n)</td></tr>
<tr><td>BFS / DFS</td><td>Graphs, grids, trees, shortest path on unweighted graphs (BFS), permutations and backtracking (DFS)</td><td>O(V + E)</td></tr>
<tr><td>Heap / top-k</td><td>k largest, k most frequent, merge k sorted lists</td><td>O(n log k)</td></tr>
<tr><td>Dynamic programming</td><td>Overlapping subproblems, optimal substructure (Kadane, knapsack, climbing stairs)</td><td>Usually O(n) to O(n x m)</td></tr>
<tr><td>LRU cache</td><td>"Evict least recently used" with O(1) get and put</td><td>O(1)</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
// LRU cache in a few lines: LinkedHashMap in access order
class Lru<K, V> extends LinkedHashMap<K, V> {
    private final int cap;
    Lru(int cap) { super(16, 0.75f, true); this.cap = cap; }
    @Override protected boolean removeEldestEntry(Map.Entry<K, V> e) { return size() > cap; }
}

// Sliding window: length of longest substring without repeating characters
static int longestUnique(String s) {
    Map<Character, Integer> last = new HashMap<>();
    int best = 0, left = 0;
    for (int right = 0; right < s.length(); right++) {
        Integer prev = last.put(s.charAt(right), right);
        if (prev != null && prev >= left) left = prev + 1;
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

<h3>When to Use</h3>
<ul>
<li>Start with the brute force, state its complexity, then name the pattern that removes the redundant work.</li>
<li>Follow-ups: thread-safe LRU (lock or <code>ConcurrentHashMap</code> plus a linked list), space-for-time trade-offs, edge cases (empty input, duplicates, overflow).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>
