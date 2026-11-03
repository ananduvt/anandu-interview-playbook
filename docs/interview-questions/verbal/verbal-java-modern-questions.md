# Java Modern Features and Generics Interview Questions

Language features added since Java 8 (var, records, Optional, java.time, Java 17) plus generics and effectively final variables.

---

## 1. What are the key features introduced in Java 17?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Java 17 (LTS, September 2021) finalized sealed classes and strongly encapsulated JDK internals, and consolidated features previewed in 12 to 16: records, text blocks, switch expressions, and pattern matching for <code>instanceof</code>.</p>

<h3>Why / Context</h3>
<table>
<tr><th>Feature</th><th>Final in</th><th>What it gives you</th></tr>
<tr><td>Sealed classes and interfaces</td><td>17</td><td>Restrict who can extend (<code>permits</code>)</td></tr>
<tr><td>Records</td><td>16</td><td>Concise immutable data carriers</td></tr>
<tr><td>Pattern matching for instanceof</td><td>16</td><td>Test and cast in one step</td></tr>
<tr><td>Text blocks</td><td>15</td><td>Multi-line strings with <code>"""</code></td></tr>
<tr><td>Switch expressions</td><td>14</td><td>Arrow syntax, returns a value</td></tr>
<tr><td>Strong encapsulation of JDK internals</td><td>17</td><td>Illegal reflective access to <code>sun.*</code> and <code>java.*</code> internals now fails</td></tr>
<tr><td>Other</td><td>17</td><td>Always-strict floating point, enhanced pseudo-random generators, new macOS rendering pipeline</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double side) implements Shape {}

static double area(Shape s) {
    if (s instanceof Circle c) return Math.PI * c.r() * c.r();   // pattern matching
    if (s instanceof Square q) return q.side() * q.side();
    throw new IllegalStateException();
}
```

<h3>When to Use</h3>
<ul>
<li>Migrating 11 to 17 often breaks on strong encapsulation (add <code>--add-opens</code> or upgrade libraries).</li>
<li>Pattern matching in <code>switch</code> is final in Java 21, not 17 (preview in 17).</li>
<li>Current LTS is Java 25; earlier LTS are 8, 11, 17, 21.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/versions-features/">Learn more: Java Versions & Features</a></p>

</details>

---

## 2. What is the var keyword? Can it be used as a generic type argument?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>var</code> (Java 10) is local variable type inference: the compiler infers the static type from the initializer. It is not dynamic typing. It cannot be used as a generic type argument (<code>List&lt;var&gt;</code> is a compile error).</p>

<h3>Why / Context</h3>
<ul>
<li>Allowed: local variables with an initializer, for-loop variables, try-with-resources, and lambda parameters (Java 11, so annotations can be applied).</li>
<li>Not allowed: fields, method parameters, return types, uninitialized or <code>null</code>-initialized variables, array initializer shorthand (<code>var a = {1,2}</code>), or bare lambdas (<code>var f = () -&gt; 1</code>).</li>
<li><code>var</code> is a reserved type name, not a keyword, so <code>var</code> can still be used as a variable name.</li>
<li>Diamond with var infers <code>Object</code>: <code>var list = new ArrayList&lt;&gt;()</code> is <code>ArrayList&lt;Object&gt;</code>.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
var names = new ArrayList<String>();          // ArrayList<String>
for (var entry : map.entrySet()) { }          // Map.Entry<K, V>
// List<var> bad = new ArrayList<>();         // compile error
// var x;                                      // compile error: no initializer
```

<h3>When to Use</h3>
<ul>
<li>Use when the type is obvious from the right-hand side or very long.</li>
<li>Avoid when it hides the type (<code>var result = service.process()</code>) and hurts readability.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 3. What is an effectively final variable in Java?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A local variable or parameter that is never reassigned after initialization, even without the <code>final</code> keyword (Java 8+). Lambdas and anonymous/local classes may only capture local variables that are final or effectively final.</p>

<h3>Why / Context</h3>
<ul>
<li>Captured locals are <strong>copied</strong> into the lambda. If the variable could change, the two copies would diverge, and locals live on a thread's stack, so sharing a mutable one across threads would be unsafe.</li>
<li>The rule restricts the variable, not the object: you can still mutate the object it references (<code>list.add</code>).</li>
<li>Instance and static fields are not subject to the rule (they are read through <code>this</code> or the class).</li>
<li>Test: add <code>final</code>; if it still compiles, the variable is effectively final.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
int base = 10;                                  // effectively final
Function<Integer, Integer> add = x -> x + base; // OK

int counter = 0;
// Runnable r = () -> counter++;                // compile error: counter is modified

AtomicInteger safe = new AtomicInteger();       // workaround: mutable holder
Runnable r2 = safe::incrementAndGet;
```

<h3>When to Use</h3>
<ul>
<li>Workarounds: single-element array, <code>AtomicInteger</code>, or better, restructure with streams (<code>reduce</code>, <code>count</code>).</li>
<li>Follow-up: a variable assigned once in both branches of an if/else (definite assignment) is still effectively final.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/functional-programming/">Learn more: Functional Programming</a></p>

</details>

---

## 4. What is a Record class? How can query results be mapped to records, and how do you fetch the first record?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A record (Java 16) is a concise, shallowly immutable data class. The compiler generates the canonical constructor, private final fields, accessors (<code>name()</code>, not <code>getName()</code>), <code>equals</code>, <code>hashCode</code>, and <code>toString</code>.</p>

<h3>Why / Context</h3>
<ul>
<li>Implicitly final, cannot extend a class, but can implement interfaces; can have static members and a compact constructor for validation.</li>
<li>Shallow immutability: a record holding a <code>List</code> can still have that list mutated; copy defensively with <code>List.copyOf</code>.</li>
<li>Good for DTOs, projections, and map keys.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
record Employee(long id, String name) {
    Employee {                                    // compact constructor
        if (name == null || name.isBlank()) throw new IllegalArgumentException("name");
    }
}

// JDBC: map each row to a record, take the first
List<Employee> rows = jdbcTemplate.query(
    "SELECT id, name FROM employee WHERE dept = ?",
    (rs, i) -> new Employee(rs.getLong("id"), rs.getString("name")),
    dept);
Optional<Employee> first = rows.stream().findFirst();

// Spring Data JPA: interface method returning a record projection
// Optional<EmployeeView> findFirstByDeptOrderByIdAsc(String dept);
```

<h3>When to Use</h3>
<ul>
<li>Mapping options: a RowMapper lambda as above, <code>DataClassRowMapper</code> (Spring), or constructor expressions in JPQL (<code>SELECT new com.app.Employee(e.id, e.name) ...</code>) and class-based projections.</li>
<li>Fetching the first record: add <code>LIMIT 1</code> (or <code>findFirstBy...</code>) in the query rather than loading all rows, then return <code>Optional</code>.</li>
<li>Records are not suitable as JPA entities (entities need a no-arg constructor and mutable state).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 5. What is the Optional class and when should you use it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>Optional&lt;T&gt;</code> (Java 8) is a container that holds a value or is empty. It makes "no result" explicit in a return type instead of returning <code>null</code>.</p>

<h3>Why / Context</h3>
<ul>
<li>Intended mainly for <strong>return types</strong> of methods that may have no result (<code>findById</code>, <code>stream.findFirst</code>).</li>
<li>Do not use it for fields, method parameters, collections elements, or serialization; return an empty collection instead of <code>Optional&lt;List&gt;</code>.</li>
<li>Never call <code>get()</code> blindly; use <code>orElseThrow()</code> (Java 10+) which is explicit.</li>
<li><code>orElse(x)</code> always evaluates <code>x</code>; use <code>orElseGet(() -&gt; ...)</code> for expensive defaults.</li>
<li>Primitive variants avoid boxing: <code>OptionalInt</code>, <code>OptionalLong</code>, <code>OptionalDouble</code>.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
String city = userRepo.findById(id)               // Optional<User>
        .map(User::address)                       // Optional<Address>
        .map(Address::city)
        .filter(c -> !c.isBlank())
        .orElse("Unknown");

User u = userRepo.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));

userRepo.findById(id).ifPresentOrElse(this::welcome, this::register); // Java 9+
```

<h3>When to Use</h3>
<ul>
<li>Return type for lookups that may legitimately be absent; chain with <code>map</code>, <code>flatMap</code>, <code>filter</code>.</li>
<li>Avoid <code>if (opt.isPresent()) opt.get()</code>; it is just a null check in disguise.</li>
<li>An <code>Optional</code> itself must never be <code>null</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/functional-programming/">Learn more: Functional Programming</a></p>

</details>

---

## 6. What are the date and time APIs introduced in Java 8 (java.time)?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>java.time</code> (JSR-310) replaces the mutable, thread-unsafe <code>java.util.Date</code>, <code>Calendar</code>, and <code>SimpleDateFormat</code> with immutable, thread-safe types and a clear model.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Class</th><th>Represents</th><th>Example</th></tr>
<tr><td><code>LocalDate</code></td><td>Date without time or zone</td><td>2026-11-03</td></tr>
<tr><td><code>LocalTime</code></td><td>Time without date or zone</td><td>14:30</td></tr>
<tr><td><code>LocalDateTime</code></td><td>Date and time, no zone</td><td>Appointment slot</td></tr>
<tr><td><code>ZonedDateTime</code></td><td>Date-time with zone and DST rules</td><td>Meeting across regions</td></tr>
<tr><td><code>OffsetDateTime</code></td><td>Date-time with fixed UTC offset</td><td>API timestamps</td></tr>
<tr><td><code>Instant</code></td><td>Point on the UTC timeline</td><td>Event time, audit column</td></tr>
<tr><td><code>Duration</code> / <code>Period</code></td><td>Time-based / date-based amount</td><td>PT2H / P3D</td></tr>
<tr><td><code>DateTimeFormatter</code></td><td>Thread-safe parsing and formatting</td><td>ISO or custom pattern</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
LocalDate due = LocalDate.of(2026, 11, 3).plusDays(30);      // new instance each time
ZonedDateTime ny = ZonedDateTime.now(ZoneId.of("America/New_York"));
Duration d = Duration.between(start, end);
String s = ny.format(DateTimeFormatter.ISO_OFFSET_DATE_TIME);
Instant now = Instant.now();                                  // store in UTC
```

<h3>When to Use</h3>
<ul>
<li>Store and exchange instants in UTC (<code>Instant</code>); convert to a zone only for display.</li>
<li>Inject a <code>Clock</code> for testable code (<code>LocalDate.now(clock)</code>).</li>
<li>Convert legacy types with <code>Date.toInstant()</code> and <code>Date.from(instant)</code>.</li>
<li>Months are 1-based, unlike <code>Calendar</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

## 7. Explain how Java generics work and their pitfalls

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Generics add compile-time type parameters (<code>List&lt;String&gt;</code>) so type errors are caught early and casts are unnecessary. The compiler then erases the type arguments (type erasure), so no generic type information exists at runtime.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Type erasure:</strong> <code>T</code> becomes its bound (or <code>Object</code>) and casts are inserted. Consequences: no <code>new T()</code>, no <code>new T[n]</code>, no <code>instanceof List&lt;String&gt;</code>, no primitive type arguments, and overloads differing only by type argument clash.</li>
<li><strong>Invariance:</strong> <code>List&lt;Integer&gt;</code> is not a <code>List&lt;Number&gt;</code> (unlike arrays, which are covariant and fail at runtime with <code>ArrayStoreException</code>).</li>
<li><strong>Wildcards (PECS):</strong> <code>? extends T</code> to read (producer), <code>? super T</code> to write (consumer).</li>
<li><strong>Heap pollution:</strong> raw types and unchecked casts defeat the checks and produce <code>ClassCastException</code> far from the cause; heed unchecked warnings.</li>
<li>Generic varargs can pollute the heap; <code>@SafeVarargs</code> only on safe, final/static/private methods.</li>
</ul>

<h3>Example / Real-World Use</h3>

```java
static <T extends Comparable<T>> T max(List<? extends T> items) {   // bounded type
    T best = items.get(0);
    for (T t : items) if (t.compareTo(best) > 0) best = t;
    return best;
}

static void addIntegers(List<? super Integer> sink) { sink.add(42); }   // consumer

List raw = new ArrayList<String>();   // raw type: unchecked warnings
raw.add(1);                           // compiles, heap pollution
String s = (String) raw.get(0);       // ClassCastException at runtime
```

<h3>When to Use</h3>
<ul>
<li>Avoid raw types; keep unchecked casts minimal and documented.</li>
<li>Pass a <code>Class&lt;T&gt;</code> token or a supplier when you need the type at runtime.</li>
<li>Bytecode compatibility with pre-Java 5 code is the reason erasure was chosen.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/advanced/">Learn more: Advanced Java</a></p>

</details>
