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

<p>📚 <a href="/anandu-interview-playbook/java-core/oop.md">Learn more: OOP</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/basics.md">Learn more: Java Basics</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/basics.md">Learn more: Java Basics</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/basics.md">Learn more: Java Basics</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/java-core/oop.md">Learn more: OOP</a></p>

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

## Additional Topics

See [Sample Topics](../index.md) for more areas to explore:
- Explain the difference between class and object
- What are the 4 pillars of OOP?
- What's the difference between ArrayList and LinkedList?
- Explain interfaces vs abstract classes
- How does garbage collection work?
- What causes a memory leak in Java?
- Explain method overloading vs overriding
- When should you use enum?

---

**See also:** [Java Core Basics](../../java-core/basics.md), [OOP](../../java-core/oop.md), [Interview Questions Overview](../index.md)
