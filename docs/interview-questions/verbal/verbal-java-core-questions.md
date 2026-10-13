# Java Core Interview Questions

Fundamentals: OOP concepts, memory management, concurrency basics, language features, and best practices.

---

## Question Categories

### Object-Oriented Programming
- Classes, objects, inheritance, polymorphism
- Interfaces vs abstract classes
- Encapsulation, coupling, cohesion

### Memory Management
- Stack vs heap
- Garbage collection basics
- Memory leaks in Java

### Strings & Primitives
- String immutability
- String interning, StringBuilder vs StringBuffer
- Primitive vs wrapper classes

### Exceptions
- Checked vs unchecked exceptions
- Exception handling best practices
- Custom exceptions

### Collections & Generics
- ArrayList vs LinkedList
- HashMap vs TreeMap vs ConcurrentHashMap
- Generics, type erasure, wildcards

### Concurrency Basics
- Threads, runnable, callable
- Synchronized keyword, volatile
- Thread safety, race conditions

### Java Features
- Inheritance hierarchies
- Method overloading vs overriding
- Immutability and final keyword

---

## Common Questions & Answers

### String Immutability
**Q: Why is String immutable in Java?**

Immutability provides: **thread-safety** (no synchronization needed), **cacheability** (string pool), safe as HashMap keys, and security (e.g., file paths, passwords).

Use `StringBuilder` for mutations to avoid creating intermediate String objects and garbage.

---

### == vs equals()
**Q: What's the difference between == and .equals()?**

- **==**: Checks **identity** (same object reference in memory)
- **.equals()**: Checks **logical equality** (content comparison)

Always override `equals()` and `hashCode()` together. Example:
```java
String a = new String("hello");
String b = new String("hello");
a == b          // false (different objects)
a.equals(b)     // true (same content)
```

---

### HashMap vs ConcurrentHashMap
**Q: What's the difference between HashMap and ConcurrentHashMap?**

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread-safe | No | Yes (segment/bucket locking) |
| Null keys | Allowed (1 null key) | **Not allowed** |
| Performance | Faster (no locking) | Slightly slower |
| Use case | Single-threaded | Multi-threaded |
| Fail-safe iterator | No (fail-fast) | Yes |

ConcurrentHashMap uses bucket-level locking for better concurrency than synchronized collections.

---

### Integer Overflow
**Q: What happens when you overflow Integer.MAX_VALUE?**

```java
Integer.MAX_VALUE + 1  // Wraps to Integer.MIN_VALUE (-2147483648)
```

Integer arithmetic in Java uses two's complement; overflow wraps silently. Use `long` or check bounds before arithmetic.

---

### Increment Operators (Pre vs Post)
**Q: What's the output of a++ + ++a for a=5?**

```java
int a = 5;
int result = a++ + ++a;  // 5 + 7 = 12; a becomes 7
```

- `a++`: Returns 5, then increments a to 6 (post-increment)
- `++a`: Increments a to 7, then returns 7 (pre-increment)
- Result: 5 + 7 = 12

**Avoid mixing operators in same expression** — very error-prone.

---

### Division by Zero
**Q: What's the difference between 0.0/0.0 and 1/0 in Java?**

```java
0.0 / 0.0     // NaN (Not a Number)
1.0 / 0.0     // Infinity
1 / 0         // ArithmeticException (integer division)
```

Floating-point division by zero returns `Infinity` or `NaN`; integer division throws exception.

---

### Fail-Fast vs Fail-Safe Iterators
**Q: What's the difference between fail-fast and fail-safe iterators?**

- **Fail-fast**: Throws `ConcurrentModificationException` if collection modified during iteration
  - Example: `ArrayList`, `HashMap`
  - Faster, detects bugs early
- **Fail-safe**: Creates copy of collection, iteration continues even if modified
  - Example: `CopyOnWriteArrayList`, `ConcurrentHashMap`
  - Slower (copies data), but safe for concurrent modifications

---

### Comparable vs Comparator
**Q: What's the difference between Comparable and Comparator?**

- **Comparable**: Natural ordering; object compares itself (`compareTo()`)
  - Implement in the class: `class Student implements Comparable<Student>`
  - Only one natural order per class
- **Comparator**: Custom ordering; external comparator defines order
  - Separate class: `Comparator<Student>` to compare by name, age, etc.
  - Multiple orderings possible

```java
// Comparable (natural order)
class Student implements Comparable<Student> {
    public int compareTo(Student other) { return this.age - other.age; }
}

// Comparator (custom order)
Comparator<Student> byName = (s1, s2) -> s1.name.compareTo(s2.name);
students.sort(byName);
```

---

## Sample Topics (To expand further)

- Explain the difference between class and object
- What are the 4 pillars of OOP?
- What's the difference between ArrayList and LinkedList?
- Explain interfaces vs abstract classes
- How does garbage collection work?
- What causes a memory leak in Java?
- Explain method overloading vs overriding
- When should you use enum?

---

[More content to be added progressively]

**See also:** [Java Core Basics](../../java-core/basics.md), [OOP](../../java-core/oop.md), [Interview Questions Overview](../index.md)
