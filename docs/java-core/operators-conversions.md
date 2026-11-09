# Bitwise Operations, Conversions and Lambda Capture

Three small topics that show up in coding rounds: bit tricks, converting between collections/arrays/strings, and why lambdas only capture "effectively final" variables.

## Bitwise Operations

| Operator | Meaning | Example (`a=0b1100`, `b=0b1010`) |
|----------|---------|-----------------------------------|
| `&` | AND | `a & b` = `0b1000` |
| `\|` | OR | `a \| b` = `0b1110` |
| `^` | XOR | `a ^ b` = `0b0110` |
| `~` | NOT (flip all bits) | `~a` = `...11110011` (-13) |
| `<<` | Left shift (multiply by 2^n) | `a << 1` = 24 |
| `>>` | Signed right shift (keeps sign bit) | `-8 >> 1` = -4 |
| `>>>` | Unsigned right shift (fills with 0) | `-8 >>> 28` = 15 |

- Operands are promoted to `int` (or `long`); shift distance is masked (`& 31` for int, `& 63` for long), so `1 << 32 == 1`.
- `byte`/`short` values are sign-extended when promoted, so `(b & 0xFF)` is the idiom to read a byte as unsigned.

### Common tricks

```java
int n = 40;

boolean isOdd      = (n & 1) == 1;                 // parity
boolean isPow2     = n > 0 && (n & (n - 1)) == 0;  // exactly one bit set
int clearLowest    = n & (n - 1);                  // drop the lowest set bit
int lowestSetBit   = n & -n;                       // isolate lowest set bit
boolean bitIsSet   = ((n >> 3) & 1) == 1;          // test bit 3
int setBit         = n | (1 << 2);                 // set bit 2
int clearBit       = n & ~(1 << 3);                // clear bit 3
int toggleBit      = n ^ (1 << 5);                 // toggle bit 5

int a = 5, b = 9;
a ^= b; b ^= a; a ^= b;                            // swap without temp (a=9, b=5)

int count = Integer.bitCount(n);                   // population count
int single = 0;
for (int x : new int[]{4, 1, 2, 1, 2}) single ^= x; // XOR cancels pairs -> 4
```

### Flags and masks

```java
final int READ = 1, WRITE = 1 << 1, EXEC = 1 << 2;
int perms = READ | WRITE;
boolean canWrite = (perms & WRITE) != 0;
perms &= ~WRITE;                                    // revoke write
```

Prefer `EnumSet` over int flags in application code; it is type-safe and backed by a bit vector.

### Useful JDK helpers

`Integer.toBinaryString`, `Integer.highestOneBit`, `Integer.numberOfTrailingZeros`, `Integer.reverse`, `Integer.rotateLeft`, `Long.bitCount`, `BitSet`.

!!! tip "Interview tips"
    - `x & (x - 1)` and `x & -x` solve most "count/isolate bits" questions.
    - `>>` vs `>>>` on negative numbers is a classic trap. `(byte) -1 >>> 4` still gives -1 because the byte is promoted to int first, then narrowed back.
    - For a power-of-two `n`, `x & (n - 1)` equals `x % n` for non-negative `x`. `HashMap` uses this trick for bucket indexing.

## Conversions

### Array, List and Stream

```java
import java.util.*;
import java.util.stream.*;

String[] arr = {"a", "b", "c"};
int[] prim   = {3, 1, 2};

// Array -> List
List<String> view  = Arrays.asList(arr);          // fixed-size view backed by the array
List<String> copy  = new ArrayList<>(Arrays.asList(arr));
List<String> imm   = List.of(arr);                // immutable (Java 9+), no nulls
List<Integer> boxed = Arrays.stream(prim).boxed().toList(); // Java 16+; Collectors.toList() before

// List -> Array
String[] a1 = copy.toArray(new String[0]);        // idiomatic
String[] a2 = copy.toArray(String[]::new);        // Java 11+
int[] p1    = boxed.stream().mapToInt(Integer::intValue).toArray();

// Array / List -> Stream
Stream<String> s1 = Arrays.stream(arr);
IntStream s2      = Arrays.stream(prim);
Stream<String> s3 = copy.stream();

// Stream -> List / Set / array / String
List<String> l  = copy.stream().collect(Collectors.toList()); // mutable in practice, not guaranteed
List<String> l2 = copy.stream().toList();                     // Java 16+, unmodifiable
Set<String> set = Stream.of(arr).collect(Collectors.toSet());
String[] back   = Stream.of(arr).toArray(String[]::new);
String joined   = Stream.of(arr).collect(Collectors.joining(","));
```

| From | To | Notes |
|------|----|-------|
| `T[]` | `List<T>` | `Arrays.asList` is a view: no add/remove, writes go through to the array |
| `int[]` | `List<Integer>` | `Arrays.asList(prim)` gives `List<int[]>` (one element); box via a stream |
| `List` | array | `toArray()` returns `Object[]`; pass a typed array or generator |
| `Stream.toList()` | | Unmodifiable, allows nulls; `List.copyOf`/`List.of` reject nulls |

### int and String

```java
int n = Integer.parseInt("42");          // NumberFormatException on bad input
Integer m = Integer.valueOf("42");       // returns Integer (cached -128..127)
int h = Integer.parseInt("ff", 16);      // 255 (radix)

String s1 = String.valueOf(42);          // preferred
String s2 = Integer.toString(42);
String s3 = "" + 42;                     // works, but least explicit
String bin = Integer.toBinaryString(42); // "101010"
String hex = Integer.toHexString(255);   // "ff"
String pad = String.format("%05d", 42);  // "00042"

char c = '7';
int digit = c - '0';                     // 7 (Integer.parseInt takes a String, not a char)
char back = (char) ('0' + 7);            // '7'
int digitSafe = Character.getNumericValue(c);
```

- Use `Long.parseLong`/`BigInteger` for values beyond `int`; `Integer.parseInt("2147483648")` throws.
- `Integer.parseInt(" 1")` fails: trim first. `null` throws `NumberFormatException`.
- Compare boxed values with `equals`, not `==` (cache only covers -128..127).

See also [Strings and primitives](strings-primitives.md).

## Effectively Final Variables and Lambda Capture

A lambda or anonymous/inner class can read a local variable of the enclosing method only if it is `final` or **effectively final**: assigned exactly once and never changed afterwards (Java 8+).

```java
int base = 10;                                   // effectively final
Function<Integer, Integer> add = x -> x + base;  // OK

int counter = 0;
Runnable r = () -> counter++;                    // compile error: must be final or effectively final
```

Also an error if the variable is modified *after* the lambda, since it then stops being effectively final:

```java
String prefix = "a";
Runnable r = () -> System.out.println(prefix);   // error
prefix = "b";
```

### Why the rule exists

- Locals live on the thread stack; a lambda may run later or on another thread, so it captures a **copy of the value**, not the variable.
- Allowing mutation would imply shared mutable stack state, which is unsafe and misleading (copy and original would diverge).
- Fields (instance and static) are not restricted: they live on the heap, and the lambda captures `this` or the class.

### Workarounds

```java
// 1. Single-element array or holder (not thread-safe)
int[] count = {0};
list.forEach(x -> count[0]++);

// 2. AtomicInteger / LongAdder (thread-safe)
AtomicInteger total = new AtomicInteger();
list.forEach(x -> total.addAndGet(x));

// 3. Best: avoid mutation, use a stream reduction
int sum = list.stream().mapToInt(Integer::intValue).sum();

// 4. Copy into a final variable inside a loop
for (int i = 0; i < 3; i++) {
    final int id = i;                            // 'i' itself is not effectively final
    executor.submit(() -> process(id));
}
```

| Capture | Allowed? |
|---------|----------|
| Local / parameter, never reassigned | Yes |
| Local reassigned anywhere | No |
| Instance or static field | Yes (reads and writes) |
| Enhanced-for variable | Yes (new variable per iteration) |
| Classic `for` counter | No |

!!! tip "Interview tips"
    - Say "captures the value, not the variable". A reference stays effectively final even if the object it points to is mutated (`list.add` is fine).
    - Inside a lambda, `this` refers to the enclosing instance, unlike an anonymous class, where it refers to the anonymous object. A lambda also cannot redeclare an enclosing local's name.
