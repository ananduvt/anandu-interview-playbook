# Java Core Interview Questions

Fundamentals: OOP concepts, memory management, concurrency basics, language features, and best practices.

---

## 1. Why is String immutable in Java?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>String objects in Java are immutable (cannot be changed after creation). Once a String is created, any modification creates a new String object.</p>

<h3>Why (Reasoning & Benefits)</h3>
<ul>
<li><strong>Thread Safety:</strong> Immutability eliminates synchronization needs. Multiple threads can safely share the same String.</li>
<li><strong>Cacheability:</strong> Strings can be pooled (String pool in heap). Same string literals reuse the same object.</li>
<li><strong>HashMap Keys:</strong> Strings are hashCode-safe. If mutable, the hash would change, breaking HashMap lookups.</li>
<li><strong>Security:</strong> Safe for sensitive data like file paths, database URLs, passwords. Cannot be altered mid-use.</li>
<li><strong>Performance:</strong> String interning optimizes memory usage.</li>
</ul>

<h3>Best Practice</h3>
<p>Use <code>StringBuilder</code> (not <code>String</code>) for string mutations in loops to avoid creating intermediate objects and garbage.</p>

```java
// ❌ Bad: Creates many String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // New String each iteration
}

// ✅ Good: Reuses buffer
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings & Primitives</a></p>

</details>

---

## 2. What's the difference between == and .equals()?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr><th>Operator</th><th>Checks</th><th>Use Case</th></tr>
<tr><td><strong>==</strong></td><td>Identity (object reference)</td><td>Comparing null, singletons</td></tr>
<tr><td><strong>.equals()</strong></td><td>Logical equality (content)</td><td>Comparing values</td></tr>
</table>

<h3>Example</h3>

```java
String a = new String("hello");
String b = new String("hello");
String c = a;

a == b          // false (different objects)
a.equals(b)     // true (same content)
a == c          // true (same reference)
```

<h3>Important: Override Both Together</h3>

<p>If you override <code>.equals()</code>, <strong>always override <code>hashCode()</code></strong> too. Objects equal by <code>.equals()</code> must have the same <code>hashCode()</code>.</p>

```java
class Person {
    String name;
    int age;
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return this.name.equals(other.name) && this.age == other.age;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);  // Same hash for equal objects
    }
}
```

<p>📚 <a href="/anandu-interview-playbook/java-core/oop/">Learn more: OOP</a></p>

</details>

---

## 3. What's the difference between HashMap and ConcurrentHashMap?

<details>
<summary>View Answer</summary>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Feature</th>
<th>HashMap</th>
<th>ConcurrentHashMap</th>
</tr>
<tr>
<td><strong>Thread-safe</strong></td>
<td>No</td>
<td>Yes (bucket-level locking)</td>
</tr>
<tr>
<td><strong>Null keys/values</strong></td>
<td>Allowed (1 null key)</td>
<td>❌ Not allowed</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Faster (no locking)</td>
<td>Slower (with locking)</td>
</tr>
<tr>
<td><strong>Use case</strong></td>
<td>Single-threaded</td>
<td>Multi-threaded</td>
</tr>
<tr>
<td><strong>Iterator</strong></td>
<td>Fail-fast (throws exception)</td>
<td>Fail-safe (weakly consistent)</td>
</tr>
</table>

<h3>When to Use</h3>

<ul>
<li><strong>HashMap:</strong> Single-threaded context or well-controlled synchronization</li>
<li><strong>ConcurrentHashMap:</strong> Multi-threaded environment with concurrent reads/writes</li>
<li><strong>Avoid:</strong> Synchronized collections like <code>Collections.synchronizedMap()</code> (single lock, poor performance)</li>
</ul>

<h3>Key Insight: Bucket-Level Locking</h3>

<p>ConcurrentHashMap locks individual buckets, allowing multiple threads to access different buckets simultaneously. HashMap locks the entire map.</p>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections</a></p>

</details>

---

## 4. What happens when you overflow Integer.MAX_VALUE?

<details>
<summary>View Answer</summary>

<h3>Answer</h3>

<p>Integer overflow wraps silently to <strong>Integer.MIN_VALUE</strong>. No exception is thrown.</p>

```java
Integer.MAX_VALUE       // 2147483647
Integer.MAX_VALUE + 1   // -2147483648 (wrapped to MIN_VALUE)
Integer.MAX_VALUE + 2   // -2147483647
```

<h3>Why This Happens</h3>

<p>Java integers use <strong>two's complement</strong> representation. When the maximum positive value increments, it overflows to the minimum negative value.</p>

<h3>Different Behavior by Type</h3>

<table>
<tr>
<th>Type</th>
<th>MAX_VALUE + 1</th>
<th>Behavior</th>
</tr>
<tr>
<td><code>int</code></td>
<td>Integer.MIN_VALUE</td>
<td>Silently wraps</td>
</tr>
<tr>
<td><code>long</code></td>
<td>Long.MIN_VALUE</td>
<td>Silently wraps</td>
</tr>
<tr>
<td><code>double</code></td>
<td>Infinity</td>
<td>Returns Infinity</td>
</tr>
</table>

<h3>How to Prevent</h3>

<ul>
<li><strong>Use long:</strong> For large numbers</li>
<li><strong>Check bounds:</strong> Before arithmetic operations</li>
<li><strong>Use BigInteger:</strong> For arbitrary precision</li>
</ul>

```java
// Check before adding
if (a > Integer.MAX_VALUE - b) {
    // Handle overflow
}

// Or use Math.addExact (throws ArithmeticException)
int result = Math.addExact(a, b);
```

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 5. What's the output of a++ + ++a for a=5?

<details>
<summary>View Answer</summary>

<h3>Answer</h3>

```java
int a = 5;
int result = a++ + ++a;
System.out.println("result: " + result + ", a: " + a);
// Output: result: 12, a: 7
```

<h3>Step-by-Step Trace</h3>

<ol>
<li><strong>a++:</strong> Returns 5 (current value), then increments a to 6 (post-increment)</li>
<li><strong>++a:</strong> Increments a to 7, then returns 7 (pre-increment)</li>
<li><strong>5 + 7 = 12</strong></li>
<li><strong>a becomes 7</strong></li>
</ol>

<h3>Why This Is Confusing</h3>

<p>The order of evaluation for <code>a++ + ++a</code> is undefined behavior in many languages. The outcome depends on compiler optimization and evaluation order, making it unreliable.</p>

<h3>Best Practice</h3>

<p><strong>❌ Avoid mixing increment operators in the same expression.</strong></p>

```java
// ❌ Bad: Confusing, undefined behavior
int result = a++ + ++a;

// ✅ Good: Clear intent
int temp1 = a++;       // a is 6
int temp2 = ++a;       // a is 7
int result = temp1 + temp2;  // 5 + 7 = 12
```

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 6. What's the difference between 0.0/0.0 and 1/0 in Java?

<details>
<summary>View Answer</summary>

<h3>Answer</h3>

<table>
<tr><th>Expression</th><th>Result</th><th>Type</th></tr>
<tr><td><code>0.0 / 0.0</code></td><td>NaN (Not a Number)</td><td>float/double</td></tr>
<tr><td><code>1.0 / 0.0</code></td><td>Infinity</td><td>float/double</td></tr>
<tr><td><code>1 / 0</code></td><td>ArithmeticException</td><td>int/long</td></tr>
</table>

<h3>Key Difference: Floating-Point vs Integer</h3>

<ul>
<li><strong>Floating-point division by zero:</strong> Returns <code>Infinity</code> or <code>NaN</code> (IEEE 754 standard)</li>
<li><strong>Integer division by zero:</strong> Throws <code>ArithmeticException</code></li>
</ul>

<h3>Example</h3>

```java
// Floating-point (no exception)
System.out.println(1.0 / 0.0);      // Infinity
System.out.println(-1.0 / 0.0);     // -Infinity
System.out.println(0.0 / 0.0);      // NaN

// Integer (throws exception)
System.out.println(1 / 0);          // ArithmeticException: / by zero

// Check for NaN
double result = 0.0 / 0.0;
if (Double.isNaN(result)) {
    System.out.println("Result is NaN");
}
```

<h3>Handling NaN and Infinity</h3>

```java
// NaN comparisons
double x = 0.0 / 0.0;
x == Double.NaN   // false (NaN != NaN always)
Double.isNaN(x)   // true (correct way)

// Infinity comparisons
double y = 1.0 / 0.0;
y == Double.POSITIVE_INFINITY  // true
Double.isInfinite(y)           // true
```

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 7. What's the difference between fail-fast and fail-safe iterators?

<details>
<summary>View Answer</summary>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Property</th>
<th>Fail-Fast</th>
<th>Fail-Safe</th>
</tr>
<tr>
<td><strong>Behavior on modification</strong></td>
<td>Throws ConcurrentModificationException</td>
<td>Continues with copy</td>
</tr>
<tr>
<td><strong>Examples</strong></td>
<td>ArrayList, HashMap, HashSet</td>
<td>CopyOnWriteArrayList, ConcurrentHashMap</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Faster (no copying)</td>
<td>Slower (creates copy)</td>
</tr>
<tr>
<td><strong>Concurrency</strong></td>
<td>Not thread-safe for modifications</td>
<td>Thread-safe</td>
</tr>
<tr>
<td><strong>Memory</strong></td>
<td>O(1)</td>
<td>O(n) - full copy</td>
</tr>
</table>

<h3>Fail-Fast Example</h3>

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
Iterator<Integer> it = list.iterator();

it.next();      // 1
list.add(4);    // Modifying list during iteration

it.next();      // ❌ ConcurrentModificationException
```

<h3>Fail-Safe Example</h3>

```java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(1, 2, 3));
Iterator<Integer> it = list.iterator();

it.next();      // 1
list.add(4);    // Modifying list during iteration

it.next();      // ✅ Works fine (sees original copy)
```

<h3>When to Use</h3>

<ul>
<li><strong>Fail-Fast:</strong> Single-threaded, want early detection of concurrent modifications</li>
<li><strong>Fail-Safe:</strong> Multi-threaded environment, safe iteration with concurrent modifications</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections</a></p>

</details>

---

## 8. What's the difference between Comparable and Comparator?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr>
<th>Feature</th>
<th>Comparable</th>
<th>Comparator</th>
</tr>
<tr>
<td><strong>Purpose</strong></td>
<td>Natural ordering of class</td>
<td>Custom ordering logic</td>
</tr>
<tr>
<td><strong>Implementation</strong></td>
<td>Implement in class: <code>class X implements Comparable&lt;X&gt;</code></td>
<td>External class: <code>class XComparator implements Comparator&lt;X&gt;</code></td>
</tr>
<tr>
<td><strong>Method</strong></td>
<td><code>compareTo(Object obj)</code></td>
<td><code>compare(Object a, Object b)</code></td>
</tr>
<tr>
<td><strong>Orderings per class</strong></td>
<td>One natural order</td>
<td>Multiple custom orders</td>
</tr>
<tr>
<td><strong>Use with</strong></td>
<td>Arrays.sort(), Collections.sort()</td>
<td>Arrays.sort(array, comparator)</td>
</tr>
</table>

<h3>Comparable Example</h3>

```java
class Student implements Comparable<Student> {
    String name;
    int age;
    
    @Override
    public int compareTo(Student other) {
        return this.age - other.age;  // Natural: sort by age
    }
}

Student[] students = { ... };
Arrays.sort(students);  // Uses natural order (age)
```

<h3>Comparator Example</h3>

```java
// Multiple sorting options
Comparator<Student> byName = (s1, s2) -> s1.name.compareTo(s2.name);
Comparator<Student> byAge = (s1, s2) -> s1.age - s2.age;
Comparator<Student> byNameThenAge = byName.thenComparing(byAge);

Student[] students = { ... };
Arrays.sort(students, byName);              // Sort by name
Arrays.sort(students, byAge);               // Sort by age
Arrays.sort(students, byNameThenAge);       // Sort by name, then age
```

<h3>When to Use</h3>

<ul>
<li><strong>Comparable:</strong> When there's an obvious natural order (e.g., ID, age, timestamp)</li>
<li><strong>Comparator:</strong> When you need multiple orderings or the class doesn't implement Comparable</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/oop/">Learn more: OOP</a></p>

</details>

---

## 9. Which Collections data structure stores unique employee objects, and what requirements must they fulfill?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Use HashSet for most cases; TreeSet for sorted order.</strong></p>

<h3>HashSet vs TreeSet Comparison</h3>

<table>
<tr><th>Feature</th><th>HashSet</th><th>TreeSet</th></tr>
<tr><td><strong>Order</strong></td><td>No guaranteed order</td><td>Sorted (natural or custom)</td></tr>
<tr><td><strong>Performance</strong></td><td>O(1) add/remove/lookup</td><td>O(log n) add/remove/lookup</td></tr>
<tr><td><strong>Memory</strong></td><td>Lower</td><td>Higher (maintains tree)</td></tr>
<tr><td><strong>Null handling</strong></td><td>Allows one null</td><td>❌ No nulls allowed</td></tr>
<tr><td><strong>Implementation</strong></td><td>Hash table</td><td>Red-Black tree</td></tr>
<tr><td><strong>Use Case</strong></td><td>Fast uniqueness check</td><td>Ordered iteration needed</td></tr>
</table>

<h3>Entity Requirements for Set Uniqueness</h3>

```java
public class Employee {
    private Long id;
    private String name;
    private String email;
    
    // ✅ MUST override equals()
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Employee)) return false;
        Employee other = (Employee) obj;
        return this.id.equals(other.id);  // or email if email is unique
    }
    
    // ✅ MUST override hashCode() in the same way
    @Override
    public int hashCode() {
        return Objects.hash(id);  // Same fields as equals()
    }
    
    // For TreeSet: implement Comparable or provide Comparator
    @Override
    public int compareTo(Employee other) {
        return Long.compare(this.id, other.id);
    }
}

// Usage
Set<Employee> employees = new HashSet<>();
employees.add(new Employee(1L, "John", "john@example.com"));
employees.add(new Employee(1L, "Jane", "jane@example.com"));
// Set size: 1 (duplicates removed by equals/hashCode)

// For sorted access
Set<Employee> sorted = new TreeSet<>(employees);
// Iterates in order by ID
```

<h3>Critical Rules</h3>

<ul>
<li><strong>✅ Override equals() and hashCode() together</strong> — Violations break Set contracts</li>
<li><strong>✅ hashCode() must return same value for equal objects</strong> — Required by Set contract</li>
<li><strong>✅ For TreeSet: implement Comparable or provide Comparator</strong> — Needed for ordering</li>
<li><strong>❌ Don't use mutable fields in equals()/hashCode()</strong> — Changing them breaks Set lookups</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 10. How do you efficiently sort and filter collections of objects using modern Java features?

<details>
<summary>View Answer</summary>

<h3>Modern Approach: Stream API (Java 8+)</h3>

<p><strong>Streams provide declarative, functional filtering and sorting.</strong></p>

```java
List<Employee> employees = Arrays.asList(
    new Employee(1L, "John", 50000),
    new Employee(2L, "Jane", 60000),
    new Employee(3L, "Bob", 45000)
);

// ✅ Filter: Salary > 50000
List<Employee> highEarners = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .collect(Collectors.toList());

// ✅ Sort: By salary descending
List<Employee> sorted = employees.stream()
    .sorted((e1, e2) -> Double.compare(e2.getSalary(), e1.getSalary()))
    .collect(Collectors.toList());

// ✅ Chain: Filter + Sort + Limit
List<Employee> top2HighEarners = employees.stream()
    .filter(e -> e.getSalary() > 45000)
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .limit(2)
    .collect(Collectors.toList());
```

<h3>Comparison: Stream vs Traditional Loop</h3>

<table>
<tr><th>Traditional Loop</th><th>Stream API</th></tr>
<tr>
<td><pre>
List<Employee> result = new ArrayList<>();
for (Employee e : employees) {
    if (e.getSalary() > 50000) {
        result.add(e);
    }
}
Collections.sort(result, 
    (e1, e2) -> Double.compare(
        e2.getSalary(),
        e1.getSalary()
    )
);
</pre></td>
<td><pre>
List<Employee> result = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .sorted(Comparator
        .comparingDouble(
            Employee::getSalary
        ).reversed()
    )
    .collect(Collectors.toList());
</pre></td>
</tr>
</table>

<h3>Common Stream Operations</h3>

```java
// Filter by multiple conditions
employees.stream()
    .filter(e -> e.getSalary() > 50000 && e.getName().startsWith("J"))
    .collect(Collectors.toList());

// Sort by multiple fields
employees.stream()
    .sorted(Comparator.comparingDouble(Employee::getSalary)
        .reversed()
        .thenComparingLong(Employee::getId))
    .collect(Collectors.toList());

// Transform (map)
List<String> names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// Distinct by field
employees.stream()
    .distinct()  // Uses equals/hashCode
    .collect(Collectors.toList());

// Group by
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Reduce to single value
double totalSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .sum();
```

<h3>Performance Considerations</h3>

<ul>
<li><strong>Streams are lazy:</strong> Operations don't execute until a terminal operation (collect, forEach, count)</li>
<li><strong>Single pass:</strong> Streams consume data once; can't reuse</li>
<li><strong>Parallel streams:</strong> Use <code>.parallelStream()</code> for large datasets on multi-core systems</li>
<li><strong>Memory:</strong> Streams can be memory-efficient with intermediate operations</li>
</ul>

```java
// For large datasets, use parallelStream
List<Employee> result = employees.parallelStream()
    .filter(e -> e.getSalary() > 50000)
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .collect(Collectors.toList());
```

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections & Stream API</a></p>

</details>

---

## 11. What is the difference between StringBuilder and StringBuffer, and when do you prefer StringBuilder?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Both are mutable character sequences with the same API. <code>StringBuffer</code> synchronizes every method (thread-safe), while <code>StringBuilder</code> (since Java 5) does not, so it is faster. Prefer <code>StringBuilder</code> almost always.</p>

<h3>Why / Context</h3>
<ul>
<li>Most string building happens in a local variable confined to one thread, so synchronization is pure overhead.</li>
<li>Even when shared, per-method locking rarely makes a compound sequence of operations atomic (<code>append</code> then <code>insert</code> can still interleave).</li>
<li>Both start with a default capacity of 16 and grow; pass an expected size to the constructor to avoid copying.</li>
</ul>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>String</th><th>StringBuilder</th><th>StringBuffer</th></tr>
<tr><td>Mutable</td><td>No</td><td>Yes</td><td>Yes</td></tr>
<tr><td>Thread-safe</td><td>Yes (immutable)</td><td>No</td><td>Yes (synchronized)</td></tr>
<tr><td>Speed for repeated edits</td><td>Slow</td><td>Fastest</td><td>Slower than StringBuilder</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
StringBuilder sb = new StringBuilder(64);
for (String part : parts) {
    sb.append(part).append(',');
}
if (sb.length() > 0) sb.setLength(sb.length() - 1); // drop trailing comma
```

<h3>When to Use</h3>
<ul>
<li><code>StringBuilder</code>: loops and single-threaded building (the default choice).</li>
<li><code>StringBuffer</code>: legacy code or a buffer genuinely shared across threads (even then, consider external locking or a different design).</li>
<li>A plain <code>+</code> outside loops is fine; the compiler optimizes it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings & Primitives</a></p>

</details>

---

## 12. How do String and StringBuffer handle memory differently?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>String</code> is immutable: every modification allocates a new object, and literals are shared through the String pool. <code>StringBuffer</code> is mutable: it keeps an internal array and edits it in place, reallocating only when capacity is exceeded.</p>

<h3>Why / Context</h3>
<ul>
<li>Literals (<code>"abc"</code>) live in the pool on the heap and are reused; <code>new String("abc")</code> creates an extra object, and <code>intern()</code> returns the pooled one.</li>
<li><code>s += x</code> in a loop creates a new String each iteration, giving O(n^2) copying and lots of garbage.</li>
<li><code>StringBuffer</code> objects are never pooled. When the array is full, a larger one (about 2x + 2) is allocated and the contents copied, so a good initial capacity matters.</li>
<li>Since Java 9, String uses compact strings (Latin-1 byte[] when possible), reducing memory use.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
String a = "hi";
String b = "hi";
System.out.println(a == b);                         // true: same pooled object
System.out.println(a == new String("hi"));          // false
System.out.println(a == new String("hi").intern()); // true

StringBuffer buf = new StringBuffer(1000);          // pre-sized, no regrowth
buf.append("hi");
```

<h3>When to Use</h3>
<ul>
<li>String for constants, keys, and values shared widely (safe and poolable).</li>
<li>A builder (<code>StringBuilder</code> preferred) for assembling text from many pieces.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings & Primitives</a></p>

</details>

---

## 13. What is the difference between Collection and Collections?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>Collection</code> is the root interface (<code>java.util.Collection</code>) of List, Set, and Queue. <code>Collections</code> is a final utility class with static helper methods that operate on collections.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Collection</th><th>Collections</th></tr>
<tr><td>Kind</td><td>Interface</td><td>Final class, private constructor</td></tr>
<tr><td>Purpose</td><td>Contract: add, remove, size, iterator, stream</td><td>Utilities: sort, reverse, shuffle, binarySearch, min/max, unmodifiableX, synchronizedX, emptyList</td></tr>
<tr><td>Instances</td><td>Implemented by ArrayList, HashSet, etc.</td><td>Never instantiated</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li>Map is <strong>not</strong> a <code>Collection</code>; it has its own hierarchy.</li>
<li>The same naming pattern exists in <code>Arrays</code> (utility) vs arrays, and <code>Objects</code> vs <code>Object</code>.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
List<String> names = new ArrayList<>(List.of("bob", "amy"));
Collections.sort(names);                                   // utility on a list
List<String> ro = Collections.unmodifiableList(names);     // read-only view
List<String> sync = Collections.synchronizedList(new ArrayList<>());
```

<h3>When to Use</h3>
<ul>
<li>Program against <code>Collection</code>/<code>List</code> in signatures; use <code>Collections</code> for sorting and wrappers.</li>
<li>Prefer <code>List.of</code> (Java 9+) for truly immutable lists; <code>unmodifiableList</code> is only a view over the original.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 14. Why is Comparable not a functional interface even though it has only one abstract method?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Technically it qualifies: <code>Comparable&lt;T&gt;</code> has a single abstract method (<code>compareTo</code>), so it can be the target of a lambda. It is simply not annotated with <code>@FunctionalInterface</code>, because its intent is different.</p>

<h3>Why / Context</h3>
<ul>
<li>A functional interface needs exactly one abstract method; the annotation is optional and only makes the compiler enforce that rule.</li>
<li><code>Comparable</code> describes a <strong>type's natural order</strong>: the object implements it on itself (<code>this.compareTo(other)</code>). It is not meant to be passed around as behavior.</li>
<li><code>Comparator</code> is the behavior-style counterpart and is annotated <code>@FunctionalInterface</code>.</li>
<li>Annotating <code>Comparable</code> would invite lambdas for what should be a stable class contract and would commit the API to staying single-method.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
Comparable<String> c = s -> s.length() - 3;   // compiles: valid lambda target
System.out.println(c.compareTo("abcd"));      // 1

Comparator<String> byLen = (a, b) -> a.length() - b.length(); // intended usage
```

<h3>When to Use</h3>
<ul>
<li><code>Comparable</code> for one natural ordering in the class itself; <code>Comparator</code> (often a lambda or <code>Comparator.comparing</code>) for alternative orderings.</li>
<li>Interview tip: any interface with one abstract method works as a lambda target; the annotation is documentation plus a compile-time check.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/functional-programming/">Learn more: Functional Programming</a></p>

</details>

---

## 15. What are the four pillars of OOP in Java, and how do they support scalable design?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Encapsulation, inheritance, polymorphism, and abstraction.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Encapsulation:</strong> hide state behind private fields and expose behavior through methods, so invariants are enforced in one place and internals can change safely.</li>
<li><strong>Inheritance:</strong> a subclass reuses and specializes a parent (<code>extends</code>). Powerful but tightly coupled; prefer composition when the relationship is not truly "is-a".</li>
<li><strong>Polymorphism:</strong> one reference type, many behaviors. Compile-time (overloading) and runtime (overriding, dynamic dispatch).</li>
<li><strong>Abstraction:</strong> expose what an object does, not how (interfaces, abstract classes), so callers depend on contracts.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
interface PaymentGateway { void pay(long cents); }            // abstraction

class CardGateway implements PaymentGateway {                  // polymorphism
    private final String merchantId;                           // encapsulation
    CardGateway(String merchantId) { this.merchantId = merchantId; }
    public void pay(long cents) { /* call card network */ }
}

class Checkout {
    private final PaymentGateway gateway;                      // depends on abstraction
    Checkout(PaymentGateway g) { this.gateway = g; }
    void complete(long cents) { gateway.pay(cents); }
}
```

<h3>When to Use</h3>
<ul>
<li>Scalability comes from loose coupling: new gateways are added without touching <code>Checkout</code> (open/closed principle).</li>
<li>Favor interfaces and composition; use inheritance sparingly for genuine specialization.</li>
<li>Common follow-up: Java has no multiple class inheritance, but a class can implement many interfaces.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/oop/">Learn more: OOP Concepts</a></p>

</details>

---

## 16. What is the difference between a class and an object?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A class is a blueprint (fields, methods, constructors) defined once at compile time. An object is a runtime instance of that class with its own state, usually created with <code>new</code> and stored on the heap.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Class</th><th>Object</th></tr>
<tr><td>Nature</td><td>Template / type definition</td><td>Concrete instance</td></tr>
<tr><td>Memory</td><td>Metadata in Metaspace; static fields once per class</td><td>Heap; instance fields per object</td></tr>
<tr><td>Count</td><td>One per class loader</td><td>Many</td></tr>
<tr><td>Created by</td><td>Declaration, loaded by a class loader</td><td><code>new</code>, reflection, deserialization, clone</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
class Account {                       // class
    static int created = 0;           // shared across all objects
    private long balance;             // per object
    Account(long b) { balance = b; created++; }
}
Account a = new Account(100);         // objects
Account b = new Account(500);         // independent balances
```

<h3>When to Use</h3>
<ul>
<li>Gotcha: a variable holds a <em>reference</em> to an object, not the object itself; <code>==</code> compares references.</li>
<li>Records and enums are also classes; their instances are objects.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/oop/">Learn more: OOP Concepts</a></p>

</details>

---

## 17. What is the difference between method overloading and overriding?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Overloading: same method name, different parameter lists in the same class (compile-time polymorphism). Overriding: a subclass provides its own implementation of an inherited method with the same signature (runtime polymorphism).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Overloading</th><th>Overriding</th></tr>
<tr><td>Signature</td><td>Must differ in parameters</td><td>Same name and parameters</td></tr>
<tr><td>Return type</td><td>Can differ (but not alone)</td><td>Same or covariant subtype</td></tr>
<tr><td>Resolved</td><td>Compile time, by static types</td><td>Runtime, by actual object type</td></tr>
<tr><td>Access</td><td>Any</td><td>Cannot be more restrictive</td></tr>
<tr><td>Checked exceptions</td><td>Any</td><td>Cannot throw broader checked exceptions</td></tr>
<tr><td>static / private / final</td><td>Can be overloaded</td><td>Cannot be overridden (static is hidden)</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
class Printer {
    void print(Object o) { System.out.println("object"); }
    void print(String s) { System.out.println("string"); }
}
class Animal { String sound() { return "..."; } }
class Dog extends Animal {
    @Override String sound() { return "woof"; }
}

Object o = "hi";
new Printer().print(o);        // "object": chosen from the static type
Animal a = new Dog();
a.sound();                     // "woof": chosen from the runtime type
```

<h3>When to Use</h3>
<ul>
<li>Always add <code>@Override</code> so the compiler catches signature typos.</li>
<li>Avoid overloads that become ambiguous with boxing or varargs.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/oop/">Learn more: OOP Concepts</a></p>

</details>

---

## 18. Explain interfaces vs abstract classes

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>An interface defines a contract (a capability) that unrelated classes can implement; an abstract class is a partially implemented base class for closely related types and can hold state.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Interface</th><th>Abstract class</th></tr>
<tr><td>Inheritance</td><td>A class implements many</td><td>A class extends one</td></tr>
<tr><td>State</td><td>Only public static final constants</td><td>Instance fields of any visibility</td></tr>
<tr><td>Constructors</td><td>No</td><td>Yes (called by subclasses)</td></tr>
<tr><td>Methods</td><td>Abstract, default, static, private (Java 8/9+)</td><td>Abstract and concrete, any access modifier</td></tr>
<tr><td>Typical role</td><td>"can do" (Comparable, Runnable)</td><td>"is a" with shared code (template method)</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li>Default methods let interfaces evolve without breaking implementers, which blurred the old "interfaces have no code" line.</li>
<li>Lambdas only target interfaces, not abstract classes.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li>Interface: the default for APIs and for multiple capabilities.</li>
<li>Abstract class: shared fields, constructors, or protected helper logic across a family of subclasses.</li>
<li>Common combo: an interface plus an abstract skeleton (<code>List</code> and <code>AbstractList</code>).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/interface-vs-abstract-class/">Learn more: Interface vs Abstract Class</a></p>

</details>

---

## 19. What types of methods can an interface contain, and can it have private methods?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Yes, private methods are allowed since Java 9. An interface can contain abstract methods, default methods (Java 8), static methods (Java 8), and private and private static methods (Java 9).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Kind</th><th>Since</th><th>Body</th><th>Visibility</th><th>Purpose</th></tr>
<tr><td>Abstract</td><td>1.0</td><td>No</td><td>Public (implicit)</td><td>The contract</td></tr>
<tr><td>default</td><td>8</td><td>Yes</td><td>Public</td><td>Optional behavior, evolve APIs</td></tr>
<tr><td>static</td><td>8</td><td>Yes</td><td>Public (private from 9)</td><td>Utility/factory methods; not inherited</td></tr>
<tr><td>private / private static</td><td>9</td><td>Yes</td><td>Interface only</td><td>Share code between default/static methods</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
interface Validator {
    boolean isValid(String s);                       // abstract

    default boolean isValidOrBlank(String s) {       // default
        return isBlank(s) || isValid(s);
    }
    static Validator notEmpty() { return s -> !s.isEmpty(); } // static
    private boolean isBlank(String s) { return s == null || s.isBlank(); } // private
}
```

<h3>When to Use</h3>
<ul>
<li>Private methods remove duplication among default methods without exposing helpers publicly.</li>
<li>Interfaces cannot have instance fields or constructors; fields are implicitly <code>public static final</code>.</li>
<li>Static interface methods are called as <code>Validator.notEmpty()</code>, never through an instance.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/interface-vs-abstract-class/">Learn more: Interface vs Abstract Class</a></p>

</details>

---

## 20. What happens when a class implements two interfaces with identical default methods?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The code does not compile unless the class overrides the method. Inside the override it can pick a parent with <code>InterfaceName.super.method()</code>.</p>

<h3>Why / Context</h3>
<ul>
<li>This is Java's answer to the diamond problem: ambiguity is an explicit compile error, never silently resolved.</li>
<li>Resolution rules, in order: (1) a class method (including one inherited from a superclass) always wins over an interface default; (2) a more specific interface (one that extends the other) wins; (3) otherwise the class must override.</li>
<li>If one interface declares the method abstract and the other default, the class still must implement it.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
interface Walker { default String move() { return "walk"; } }
interface Swimmer { default String move() { return "swim"; } }

class Duck implements Walker, Swimmer {
    @Override
    public String move() {
        return Walker.super.move() + " and " + Swimmer.super.move();
    }
}
// new Duck().move() -> "walk and swim"
```

<h3>When to Use</h3>
<ul>
<li>Override and delegate to one parent (or combine both), or write fresh behavior.</li>
<li>Follow-up: interface fields are constants and must be referenced by qualified name, so they cause no silent ambiguity.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/interface-vs-abstract-class/">Learn more: Interface vs Abstract Class</a></p>

</details>

---

## 21. When should you use an enum?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use an enum whenever a value comes from a fixed, known set of constants (statuses, days, roles, types). It is type-safe, readable, and can carry fields and behavior.</p>

<h3>Why / Context</h3>
<ul>
<li>Beats <code>int</code>/<code>String</code> constants: the compiler rejects invalid values, and switch expressions must cover all constants.</li>
<li>Each constant is a singleton, so compare with <code>==</code>. An enum is also the simplest thread-safe, serialization-safe singleton.</li>
<li>Enums can have constructors, fields, methods, implement interfaces, and have constant-specific bodies. They cannot extend other classes.</li>
<li>Utilities: <code>values()</code>, <code>valueOf()</code>, <code>ordinal()</code>, plus <code>EnumMap</code> and <code>EnumSet</code> (fast and compact).</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
enum OrderStatus {
    NEW(false), PAID(false), SHIPPED(true), CANCELLED(true);

    private final boolean terminal;
    OrderStatus(boolean terminal) { this.terminal = terminal; }
    boolean isTerminal() { return terminal; }
}

String label = switch (status) {          // switch expression: must be exhaustive
    case NEW, PAID -> "open";
    case SHIPPED, CANCELLED -> "closed";
};
```

<h3>When to Use</h3>
<ul>
<li>Do: fixed sets, strategy-per-constant, map keys (<code>EnumMap</code>).</li>
<li>Do not: values that change at runtime or come from data (use a table or class).</li>
<li>Gotcha: persist the name, not <code>ordinal()</code>, since reordering constants silently corrupts data (JPA: <code>EnumType.STRING</code>).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 22. Describe Java's exception handling mechanism and the difference between checked and unchecked exceptions

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Exceptions are objects thrown with <code>throw</code> and handled with <code>try/catch/finally</code>. All derive from <code>Throwable</code>: <code>Error</code> (JVM problems, do not catch), <code>Exception</code> (checked), and <code>RuntimeException</code> (unchecked).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Property</th><th>Checked</th><th>Unchecked</th></tr>
<tr><td>Superclass</td><td>Exception (not RuntimeException)</td><td>RuntimeException, Error</td></tr>
<tr><td>Compiler enforces</td><td>Catch or declare with <code>throws</code></td><td>No</td></tr>
<tr><td>Typical cause</td><td>Recoverable external failure (IOException, SQLException)</td><td>Programming bug (NPE, IllegalArgumentException)</td></tr>
</table>

<h3>Why / Context</h3>
<ul>
<li><strong>Propagation:</strong> an uncaught exception unwinds the stack to the nearest matching catch.</li>
<li><strong>finally</strong> runs on both paths (except <code>System.exit</code> or a JVM crash); prefer try-with-resources for <code>AutoCloseable</code> (suppressed exceptions are retained).</li>
<li>Catch the most specific type first; multi-catch uses <code>|</code>.</li>
<li>Wrap with the cause to keep the stack trace: <code>new ServiceException("msg", e)</code>.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
class OrderNotFoundException extends RuntimeException {
    OrderNotFoundException(long id, Throwable cause) {
        super("Order " + id + " not found", cause);
    }
}

try (var in = Files.newBufferedReader(path)) {
    return in.readLine();
} catch (IOException e) {
    throw new UncheckedIOException(e);       // translate at the boundary
}
```

<h3>When to Use</h3>
<ul>
<li>Custom exceptions: extend <code>RuntimeException</code> for business/programming errors; use checked only when the caller can reasonably recover.</li>
<li>Do not swallow exceptions, log and rethrow the same one, or use exceptions for normal flow control.</li>
<li>Handle centrally at boundaries (for example, a REST exception handler).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/exceptions/">Learn more: Exception Handling</a></p>

</details>

---

## 23. What are checked exceptions? What are their advantages and disadvantages?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Checked exceptions are subclasses of <code>Exception</code> (excluding <code>RuntimeException</code>) that the compiler forces you to catch or declare with <code>throws</code>. They model failures the caller is expected to anticipate and possibly recover from.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Advantages</th><th>Disadvantages</th></tr>
<tr><td>Failure modes are part of the method signature</td><td>Boilerplate try/catch and <code>throws</code> clutter</td></tr>
<tr><td>Compiler ensures callers consider recovery</td><td>Leaks implementation details up the layers (SQLException in a service API)</td></tr>
<tr><td>Good documentation for I/O, network, DB calls</td><td>Awkward with lambdas and streams: standard functional interfaces do not declare them</td></tr>
<tr><td>Hard to ignore silently</td><td>Often swallowed with empty catch blocks; changing a signature breaks every caller</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
// Lambdas cannot throw checked exceptions directly
paths.stream()
     .map(p -> {
         try { return Files.readString(p); }
         catch (IOException e) { throw new UncheckedIOException(e); }
     })
     .toList();
```

<h3>When to Use</h3>
<ul>
<li>Use checked for recoverable conditions the caller can act on (retry, fallback, user message).</li>
<li>Modern frameworks (Spring, Hibernate) favor unchecked exceptions; translate checked ones at layer boundaries and keep the cause.</li>
<li>Rule of thumb: if the caller cannot do anything useful, make it unchecked.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/exceptions/">Learn more: Exception Handling</a></p>

</details>

---

## 24. What is the difference between final, finally, and finalize?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>They are unrelated: <code>final</code> is a modifier that prevents change, <code>finally</code> is a block that runs after try/catch for cleanup, and <code>finalize()</code> is a deprecated <code>Object</code> method the GC could call before reclaiming an object.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Item</th><th>Kind</th><th>Meaning</th></tr>
<tr><td><code>final</code></td><td>Modifier</td><td>Variable: assigned once. Method: cannot be overridden. Class: cannot be extended.</td></tr>
<tr><td><code>finally</code></td><td>Block</td><td>Runs whether or not an exception was thrown.</td></tr>
<tr><td><code>finalize()</code></td><td>Method</td><td>Cleanup hook called at most once by the GC. Deprecated for removal since Java 18 (JEP 421).</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
final List<String> names = new ArrayList<>();
names.add("a");            // OK: the reference is final, the object is still mutable
// names = new ArrayList<>();   // compile error

try (var in = Files.newBufferedReader(path)) {   // preferred over try/finally
    return in.readLine();
} finally {
    metrics.increment("read.attempts");           // always runs
}
```

<h3>Gotchas</h3>
<ul>
<li><code>final</code> on a reference does not make the object immutable.</li>
<li>A <code>return</code> inside <code>finally</code> overrides a return or exception from the try block; avoid it.</li>
<li><code>finally</code> does not run if the JVM exits (<code>System.exit</code>) or the thread is killed.</li>
<li><code>finalize()</code> has no timing guarantee, can resurrect objects and hurts GC. Use try-with-resources, <code>Cleaner</code> or <code>PhantomReference</code> instead.</li>
</ul>

<h3>When to Use</h3>
<ul>
<li><code>final</code> for constants, fields set once, and immutable classes.</li>
<li>try-with-resources for <code>AutoCloseable</code>; <code>finally</code> for other must-run cleanup.</li>
<li>Never write new <code>finalize()</code> methods.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/exceptions/">Learn more: Exception Handling</a></p>

</details>

---

**See also:** [Java Core Basics](../../java-core/basics.md), [OOP](../../java-core/oop.md), [Interview Preparation](../index.md)
