# Stream API Coding Problems

Modern Java functional programming with Stream API - filter, map, reduce, and complex stream operations.

---

## 1. Convert a list of strings to uppercase and sort them

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given a list of words, convert all to uppercase and return sorted. Handle case-insensitive sorting.</p>

<h3>Approach 1: Stream API (Recommended)</h3>

```java
List<String> words = Arrays.asList("banana", "apple", "Cherry", "date");

// Convert to uppercase, sort, and collect
List<String> result = words.stream()
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());

System.out.println(result);  // [APPLE, BANANA, CHERRY, DATE]
```

<h3>Approach 2: Case-Insensitive Sort</h3>

```java
List<String> result = words.stream()
    .map(String::toUpperCase)
    .sorted(String::compareToIgnoreCase)  // Case-insensitive
    .collect(Collectors.toList());

// Or with explicit comparator
List<String> result = words.stream()
    .map(String::toUpperCase)
    .sorted(Comparator.comparing(String::toString, String.CASE_INSENSITIVE_ORDER))
    .collect(Collectors.toList());
```

<h3>Approach 3: With Duplicates Removal</h3>

```java
List<String> result = words.stream()
    .map(String::toUpperCase)
    .distinct()  // Remove duplicates
    .sorted()
    .collect(Collectors.toList());
```

<h4>Complexity Comparison</h4>

<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Use Case</th></tr>
<tr><td>Stream map + sort</td><td>O(n log n)</td><td>O(n)</td><td>Simple, clean code</td></tr>
<tr><td>Stream + distinct + sort</td><td>O(n log n)</td><td>O(n)</td><td>Remove duplicates first</td></tr>
<tr><td>Traditional loop</td><td>O(n log n)</td><td>O(n)</td><td>When stream unavailable</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Stream API & Collections</a></p>

</details>

---

## 2. Filter a list of integers and collect even numbers greater than 10

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given a list of integers, filter for even numbers greater than 10, and return as a list.</p>

<h3>Approach 1: Stream Filter (Recommended)</h3>

```java
List<Integer> numbers = Arrays.asList(5, 12, 8, 15, 20, 3, 18, 7, 25);

// Filter even numbers > 10
List<Integer> result = numbers.stream()
    .filter(n -> n > 10)           // Greater than 10
    .filter(n -> n % 2 == 0)       // Even numbers
    .collect(Collectors.toList());

System.out.println(result);  // [12, 20, 18]
```

<h3>Approach 2: Combined Filter Condition</h3>

```java
List<Integer> result = numbers.stream()
    .filter(n -> n > 10 && n % 2 == 0)  // Single filter
    .collect(Collectors.toList());
```

<h3>Approach 3: Filter to Set (Remove Duplicates)</h3>

```java
Set<Integer> result = numbers.stream()
    .filter(n -> n > 10 && n % 2 == 0)
    .collect(Collectors.toSet());
```

<h3>Approach 4: Sorted and Limited</h3>

```java
List<Integer> result = numbers.stream()
    .filter(n -> n > 10 && n % 2 == 0)
    .sorted(Comparator.reverseOrder())  // Descending
    .limit(2)                            // Top 2
    .collect(Collectors.toList());

// Result: [20, 18]
```

<h4>Complexity Comparison</h4>

<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Best For</th></tr>
<tr><td>Two filters</td><td>O(n)</td><td>O(k)</td><td>Readability</td></tr>
<tr><td>Combined filter</td><td>O(n)</td><td>O(k)</td><td>Performance</td></tr>
<tr><td>Filter to Set</td><td>O(n)</td><td>O(k)</td><td>Uniqueness</td></tr>
<tr><td>Filter + sort + limit</td><td>O(n log n)</td><td>O(k)</td><td>Top-K results</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Stream API & Collections</a></p>

</details>

---

## 3. Use Stream to count occurrences of each string in a list

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given a list of words, count how many times each word appears and return a map of word→count.</p>

<h3>Approach 1: groupingBy with Counting (Recommended)</h3>

```java
List<String> words = Arrays.asList("apple", "banana", "apple", "cherry", "banana", "apple");

// Group by word and count
Map<String, Long> wordCount = words.stream()
    .collect(Collectors.groupingBy(
        w -> w,                           // Group by word itself
        Collectors.counting()              // Count occurrences
    ));

System.out.println(wordCount);
// {apple=3, banana=2, cherry=1}
```

<h3>Approach 2: Case-Insensitive Counting</h3>

```java
Map<String, Long> wordCount = words.stream()
    .map(String::toLowerCase)  // Normalize
    .collect(Collectors.groupingBy(
        w -> w,
        Collectors.counting()
    ));

// "Apple" and "apple" counted together
```

<h3>Approach 3: Sort by Count (Descending)</h3>

```java
Map<String, Long> wordCount = words.stream()
    .collect(Collectors.groupingBy(
        w -> w,
        Collectors.counting()
    ));

// Sort by count descending
List<Map.Entry<String, Long>> sorted = wordCount.entrySet()
    .stream()
    .sorted((e1, e2) -> Long.compare(e2.getValue(), e1.getValue()))
    .collect(Collectors.toList());

// or using comparingByValue
List<Map.Entry<String, Long>> sorted = wordCount.entrySet()
    .stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .collect(Collectors.toList());

sorted.forEach(e -> System.out.println(e.getKey() + ": " + e.getValue()));
// apple: 3
// banana: 2
// cherry: 1
```

<h3>Approach 4: Traditional HashMap (for comparison)</h3>

```java
Map<String, Integer> wordCount = new HashMap<>();
for (String word : words) {
    wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
}
```

<h4>Complexity Comparison</h4>

<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Best For</th></tr>
<tr><td>groupingBy().counting()</td><td>O(n)</td><td>O(k)</td><td>Clean, functional</td></tr>
<tr><td>HashMap loop</td><td>O(n)</td><td>O(k)</td><td>Performance critical</td></tr>
<tr><td>groupingBy + sort</td><td>O(n + k log k)</td><td>O(k)</td><td>Need sorted results</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Stream API & Collections</a></p>

</details>

---

## 4. Use Stream reduce() to calculate sum, product, and concatenation

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Use Stream.reduce() to compute sum, product, and string concatenation of collection elements.</p>

<h3>Approach 1: Sum with reduce()</h3>

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum using reduce
int sum = numbers.stream()
    .reduce(0, Integer::sum);  // 0 is identity/initial value

System.out.println(sum);  // 15

// Or equivalent
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
```

<h3>Approach 2: Product with reduce()</h3>

```java
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 1 is identity for multiplication

System.out.println(product);  // 120 (1*2*3*4*5)
```

<h3>Approach 3: String Concatenation</h3>

```java
List<String> words = Arrays.asList("Hello", " ", "World");

// Concatenate strings
String result = words.stream()
    .reduce("", String::concat);  // "" is identity for strings

System.out.println(result);  // "Hello World"

// Or with custom separator
String result = words.stream()
    .reduce((a, b) -> a + "-" + b)
    .orElse("");  // Handle empty stream

System.out.println(result);  // "Hello- -World"
```

<h3>Approach 4: Optional reduce (no identity)</h3>

```java
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);  // No identity value

if (sum.isPresent()) {
    System.out.println(sum.get());  // 15
} else {
    System.out.println("Empty stream");
}

// Shorthand
numbers.stream()
    .reduce((a, b) -> a + b)
    .ifPresent(System.out::println);
```

<h3>Approach 5: Complex Reduction - Find Max</h3>

```java
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);

System.out.println(max.get());  // 5

// Equivalent but clearer
Optional<Integer> max = numbers.stream()
    .max(Integer::compare);
```

<h3>Approach 6: Collect vs Reduce</h3>

```java
// DON'T: Inefficient string concat with reduce
String result = words.stream()
    .reduce("", (a, b) -> a + b);  // Creates many intermediate strings ❌

// DO: Use Collectors.joining (optimized)
String result = words.stream()
    .collect(Collectors.joining(""));  // ✅ Uses StringBuilder internally
```

<h4>reduce() Summary Table</h4>

<table>
<tr><th>Function</th><th>Identity</th><th>Example</th><th>Result</th></tr>
<tr><td>Sum</td><td>0</td><td>reduce(0, Integer::sum)</td><td>15</td></tr>
<tr><td>Product</td><td>1</td><td>reduce(1, (a,b) → a*b)</td><td>120</td></tr>
<tr><td>Max</td><td>N/A</td><td>reduce((a,b) → max(a,b))</td><td>5</td></tr>
<tr><td>Concat</td><td>""</td><td>reduce("", String::concat)</td><td>"12345"</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Stream API & Collections</a></p>

</details>

---

## 5. Combine multiple Stream operations: filter, map, and collect

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given a list of employees with salary, filter by salary > 50k, extract names, and collect as comma-separated string.</p>

<h3>Approach 1: Chained Stream Operations (Recommended)</h3>

```java
class Employee {
    String name;
    double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public double getSalary() { return salary; }
}

List<Employee> employees = Arrays.asList(
    new Employee("Alice", 75000),
    new Employee("Bob", 45000),
    new Employee("Charlie", 95000),
    new Employee("Diana", 55000),
    new Employee("Eve", 40000)
);

// Filter salary > 50k, map to names, collect as CSV
String result = employees.stream()
    .filter(e -> e.getSalary() > 50000)      // Filter
    .map(Employee::getName)                  // Transform
    .collect(Collectors.joining(", "));      // Collect as string

System.out.println(result);  // "Alice, Charlie, Diana"
```

<h3>Approach 2: Collect to List First</h3>

```java
List<String> highEarners = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .map(Employee::getName)
    .collect(Collectors.toList());

System.out.println(highEarners);  // [Alice, Charlie, Diana]
```

<h3>Approach 3: Multiple Filters</h3>

```java
String result = employees.stream()
    .filter(e -> e.getSalary() > 50000)  // Salary filter
    .filter(e -> e.getName().length() > 3)  // Name length filter
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// Result: "Alice, Charlie, Diana"
```

<h3>Approach 4: Map to Different Object</h3>

```java
class EmployeeDTO {
    String name;
    double salary;
    
    public EmployeeDTO(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
}

List<EmployeeDTO> dtos = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .map(e -> new EmployeeDTO(e.getName(), e.getSalary()))
    .collect(Collectors.toList());
```

<h3>Approach 5: Sorted Collection</h3>

```java
String result = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .sorted((e1, e2) -> Double.compare(e2.getSalary(), e1.getSalary()))  // Descending
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// Result: "Charlie, Alice, Diana" (sorted by salary desc)
```

<h3>Approach 6: Group and Collect</h3>

```java
Map<Boolean, List<Employee>> grouped = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

List<Employee> highEarners = grouped.get(true);
List<Employee> lowEarners = grouped.get(false);

System.out.println("High: " + highEarners.size());  // 3
System.out.println("Low: " + lowEarners.size());    // 2
```

<h4>Stream Pipeline Performance</h4>

<table>
<tr><th>Operation</th><th>Type</th><th>Lazy?</th><th>Impact</th></tr>
<tr><td>filter()</td><td>Intermediate</td><td>✅ Yes</td><td>Processes only needed elements</td></tr>
<tr><td>map()</td><td>Intermediate</td><td>✅ Yes</td><td>Transforms elements</td></tr>
<tr><td>sorted()</td><td>Intermediate</td><td>❌ No</td><td>Must consume entire stream</td></tr>
<tr><td>distinct()</td><td>Intermediate</td><td>❌ No</td><td>Requires buffering</td></tr>
<tr><td>collect()</td><td>Terminal</td><td>N/A</td><td>Triggers execution</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Stream API & Collections</a></p>

</details>

---

**See also:** [Collections Framework](../../java-collections-concurrency/collections.md), [Interview Questions Overview](../index.md)
