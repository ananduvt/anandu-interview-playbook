# Functional Programming (Lambdas & Method References)

## Lambda Expressions

**(argument) -> (body)**

Before Java 8, we would usually create a class for every case where we needed to encapsulate a single piece of functionality. This implied a lot of unnecessary boilerplate code to define something that served as a primitive function representation.

A lambda expression is a short block of code which takes in parameters and returns a value. Lambda expressions are similar to methods, but they do not need a name and they can be implemented right in the body of a method.

* Enable to treat functionality as a method argument, or code as data.
* A function that can be created without belonging to any class.
* A lambda expression can be passed around as if it was an object and executed on demand.

**Java lambda functions can be only used with functional interfaces.Lambda expressions are just like functions and they accept parameters just like functions.**

**Advantages**

* Reduced Lines of Code
* Sequential and Parallel Execution Support
* Passing Behaviors into methods
* Higher Efficiency with Laziness

There are three Lambda Expression Parameters

1. Zero Parameter
   () -> System.out.println("Zero parameter lambda");

2. Single Parameter
   (p) -> System.out.println("One parameter: " + p);

3. Multiple Parameters
   (p1, p2) -> System.out.println("Multiple parameters: " + p1 + ", " + p2);

Eg:
interface StringFunction {
   String run(String str);
 }
  public class Main {
   public static void main(String[] args) {
     StringFunction exclaim = (s) -> s + "!";
     StringFunction ask = (s) -> s + "?";
     printFormatted("Hello", exclaim);
     printFormatted("Hello", ask);
   }
   public static void printFormatted(String str, StringFunction format) {
     String result = format.run(str);
     System.out.println(result);
   }
 }
—-----------------------------------------------------------------------------
//Traditional approach
private static boolean isPrime(int number) {
   if(number < 2) return false;
   for(int i=2; i<number; i++){
       if(number % i == 0) return false;
   }
   return true;
}

//Declarative approach
private static boolean isPrime(int number) {
   return number > 1
           && IntStream.range(2, number).noneMatch(
                   index -> number % index == 0);
}
private static boolean isPrime(int number) {
   IntPredicate isDivisible = index -> number % index == 0;

   return number > 1
           && IntStream.range(2, number).noneMatch(
                   isDivisible);
}
—-----------------------------------------------------------------------------
public static int sumWithCondition(List<Integer> numbers, Predicate<Integer> predicate) {
   return numbers.parallelStream()
           .filter(predicate)
           .mapToInt(i -> i)
           .sum();
}

//sum of all numbers
sumWithCondition(numbers, n -> true)
//sum of all even numbers
sumWithCondition(numbers, i -> i%2==0)
//sum of all numbers greater than 5
sumWithCondition(numbers, i -> i>5)

## Method References (Double Colon :: Operator)

**<Class name>::<method name>**

The double colon (::) operator, also known as method reference operator in Java, is used to call a **method by referring** to it with the help of its class directly. They behave exactly as the lambda expressions. The only difference it has from lambda expressions is that this uses **direct reference to the method** by name instead of providing a delegate to the method.

There are four kinds of method references:

* Reference to a static method ClassName::staticMethodName
* Reference to an instance method of a particular object Object::instanceMethodName
* Reference to an instance method of an arbitrary object of a particular type ContainingType::methodName–
* Reference to a constructor ClassName::new

* (ClassName::methodName)
* (objectOfClass::methodName)
* (super::methodName)
* (ClassName::methodName)
* (ClassName::new)

Eg:

* **System::getProperty**
* **System.out::println**
* **"abc"::length**
* **ArrayList::new**
* **Int[]::new**

## Functional Interfaces

Any interface with a **SAM(Single Abstract Method)** is a functional interface, and its implementation may be treated as lambda expressions. A functional interface can have any number of default methods.
Eg:

* **Runnable** –> This interface only contains the run() method.
* **Comparable** –> This interface only contains the compareTo() method.
* **ActionListener** –> This interface only contains the actionPerformed() method.
* **Callable** –> This interface only contains the call() method.

Functional interfaces are included in Java SE 8 with Lambda expressions and Method references in order to make code more readable, clean, and straightforward. Functional interfaces are interfaces that ensure that they include precisely only one abstract method. Functional interfaces are used and executed by representing the interface with an annotation called **@FunctionalInterface**. Functional interfaces can contain only one abstract method. However, they can include any quantity of default and static methods.

### Basic Functional Interface with Default Methods

```java
@FunctionalInterface
public interface StringProcessor {
    // Single Abstract Method (SAM)
    String process(String input);
    
    // Default methods - multiple allowed
    default String toUpperCase(String input) {
        return process(input).toUpperCase();
    }
    
    default String toLowerCase(String input) {
        return process(input).toLowerCase();
    }
    
    // Static methods - can have multiple
    static StringProcessor identity() {
        return s -> s;  // Lambda implementing SAM
    }
    
    // Default method with composition
    default StringProcessor andThen(StringProcessor after) {
        return input -> after.process(process(input));
    }
}
```

**Usage**:
```java
// Lambda expression implementing the SAM
StringProcessor addExclamation = s -> s + "!";

// Using default methods
String result1 = addExclamation.toUpperCase("hello");  // "HELLO!"
String result2 = addExclamation.andThen(s -> ">> " + s).process("hi");  // ">> HI!"

// Using static method
StringProcessor identity = StringProcessor.identity();
String result3 = identity.process("test");  // "test"
```

### Four Main Categories of Functional Interfaces

**1. Consumer** — Takes input, returns nothing (void)
```java
// Built-in Consumer
Consumer<String> printConsumer = name -> System.out.println("Hello, " + name);
List<String> names = Arrays.asList("John", "Freddy", "Samuel");
names.forEach(printConsumer);

// Custom Consumer with default methods
@FunctionalInterface
interface LogConsumer<T> {
    void consume(T item);
    
    default LogConsumer<T> andThen(LogConsumer<T> next) {
        return item -> {
            consume(item);
            next.consume(item);
        };
    }
}

LogConsumer<String> logger = s -> System.out.println("Logging: " + s);
LogConsumer<String> persister = s -> System.out.println("Persisting: " + s);
LogConsumer<String> combined = logger.andThen(persister);
combined.consume("Important message");
// Output:
// Logging: Important message
// Persisting: Important message
```

**2. Predicate** — Takes input, returns boolean
```java
// Built-in Predicate
public interface Predicate<T> {
    boolean test(T t);
    
    // Default methods in Predicate
    default Predicate<T> and(Predicate<T> other) {
        return t -> test(t) && other.test(t);
    }
    
    default Predicate<T> or(Predicate<T> other) {
        return t -> test(t) || other.test(t);
    }
    
    default Predicate<T> negate() {
        return t -> !test(t);
    }
}

// Usage with composition
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);

List<Integer> numbers = Arrays.asList(-2, -1, 0, 1, 2, 3, 4);
List<Integer> result = numbers.stream()
    .filter(isEvenAndPositive)
    .collect(Collectors.toList());  // [2, 4]

// Using negate
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> doesNotStartWithA = startsWithA.negate();
List<String> names = Arrays.asList("Angela", "Aaron", "Bob", "Claire");
List<String> filteredNames = names.stream()
    .filter(doesNotStartWithA)
    .collect(Collectors.toList());  // [Bob, Claire]
```

**3. Function** — Takes input, returns output (transformation)
```java
// Built-in Function
public interface Function<T, R> {
    R apply(T t);
    
    // Composition methods
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        return v -> apply(before.apply(v));
    }
    
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        return t -> after.apply(apply(t));
    }
    
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

// Custom Function with default methods
@FunctionalInterface
interface DataTransformer<T, R> {
    R transform(T input);
    
    default <V> DataTransformer<V, R> compose(DataTransformer<? super V, ? extends T> before) {
        return v -> transform(before.transform(v));
    }
    
    default <V> DataTransformer<T, V> andThen(DataTransformer<? super R, ? extends V> after) {
        return t -> after.transform(transform(t));
    }
    
    default DataTransformer<T, R> repeat(int times) {
        return input -> {
            R result = transform(input);
            for (int i = 1; i < times; i++) {
                result = transform((T) result);
            }
            return result;
        };
    }
}

// Usage
DataTransformer<Integer, Integer> double_fn = n -> n * 2;
DataTransformer<Integer, Integer> addTen = n -> n + 10;

// Composition: (n * 2) + 10
DataTransformer<Integer, Integer> combined = double_fn.andThen(addTen);
int result = combined.transform(5);  // (5 * 2) + 10 = 20

// Map usage
Map<String, Integer> nameMap = new HashMap<>();
Function<String, Integer> lengthFunction = String::length;
Integer value = nameMap.computeIfAbsent("John", lengthFunction);  // 4
```

**4. Supplier** — Takes nothing, returns output (source)
```java
// Built-in Supplier
public interface Supplier<T> {
    T get();
    
    // Can have default methods
    default Supplier<T> memoize() {
        return new Supplier<T>() {
            private T value;
            private boolean computed = false;
            
            @Override
            public T get() {
                if (!computed) {
                    value = Supplier.this.get();
                    computed = true;
                }
                return value;
            }
        };
    }
}

// Custom Supplier with default methods
@FunctionalInterface
interface DataSource<T> {
    T provide();
    
    default Supplier<T> asSupplier() {
        return this::provide;
    }
    
    default DataSource<T> withDefault(T defaultValue) {
        return () -> {
            T value = provide();
            return value != null ? value : defaultValue;
        };
    }
    
    static <T> DataSource<T> constant(T value) {
        return () -> value;
    }
}

// Usage
Supplier<Integer> randomSupplier = () -> new Random().nextInt(100);
int value = randomSupplier.get();  // Random value each time

DataSource<String> dataSource = () -> "Hello World";
DataSource<String> withDefault = dataSource.withDefault("Default");
String result = withDefault.provide();  // "Hello World"

// Constant supplier
DataSource<String> constant = DataSource.constant("Fixed Value");
String value1 = constant.provide();  // "Fixed Value"
String value2 = constant.provide();  // "Fixed Value"
```

### BiFunction and Other Binary Operations

```java
// BiFunction - takes 2 inputs, returns 1 output
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        return (t, u) -> after.apply(apply(t, u));
    }
}

// Usage
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;

int sum = add.apply(5, 3);  // 8
int product = multiply.apply(5, 3);  // 15

// Composition
BiFunction<Integer, Integer, String> addAndFormat = add.andThen(r -> "Result: " + r);
String formatted = addAndFormat.apply(5, 3);  // "Result: 8"

// Map usage
Map<String, Integer> nameMap = new HashMap<>();
nameMap.compute("age", (key, value) -> value == null ? 0 : value + 1);
```

### Advanced: Custom Functional Interface with Multiple Default Methods

```java
@FunctionalInterface
interface Operation<T> {
    // Single Abstract Method
    T execute(T input);
    
    // Default methods for composition
    default Operation<T> andThen(Operation<T> next) {
        return input -> next.execute(execute(input));
    }
    
    default Operation<T> repeat(int times) {
        return input -> {
            T result = input;
            for (int i = 0; i < times; i++) {
                result = execute(result);
            }
            return result;
        };
    }
    
    default Operation<T> orElse(Operation<T> fallback) {
        return input -> {
            try {
                return execute(input);
            } catch (Exception e) {
                return fallback.execute(input);
            }
        };
    }
    
    // Static factory method
    static <T> Operation<T> identity() {
        return t -> t;
    }
    
    // Chaining multiple operations
    static <T> Operation<T> chain(Operation<T>... operations) {
        return input -> {
            T result = input;
            for (Operation<T> op : operations) {
                result = op.execute(result);
            }
            return result;
        };
    }
}

// Usage
Operation<Integer> double_op = n -> n * 2;
Operation<Integer> addFive = n -> n + 5;
Operation<Integer> square = n -> n * n;

// Chain: ((n * 2) + 5) * (n * 2) + 5
Operation<Integer> chained = double_op
    .andThen(addFive)
    .andThen(square);

int result = chained.execute(3);  // ((3 * 2) + 5)^2 = 121

// Repeat: n * 2 four times
Operation<Integer> repeated = double_op.repeat(4);
int doubled = repeated.execute(1);  // 16 (1 * 2 * 2 * 2 * 2)
```

### Key Points About Functional Interfaces

| Aspect | Details |
|--------|---------|
| **Single Abstract Method (SAM)** | Must have exactly 1 abstract method |
| **Default Methods** | Can have 0 or more default methods |
| **Static Methods** | Can have 0 or more static methods |
| **@FunctionalInterface** | Optional annotation for compile-time check |
| **Inheritance** | Can extend other interfaces if they don't add new abstract methods |
| **Lambda Compatible** | Can be instantiated with lambda expressions |
| **Method References** | Can be instantiated with method references (::) |
| **Composition** | Default/static methods enable method chaining |

### Common Pitfalls

```java
// ❌ NOT a functional interface - 2 abstract methods
@FunctionalInterface
interface BadInterface<T> {
    T process(T input);
    void validate(T input);  // 2nd abstract method!
}

// ❌ NOT a functional interface - overrides don't count
@FunctionalInterface
interface AlsoBad<T> {
    T process(T input);
    @Override
    boolean equals(Object o);  // Overrides Object.equals, OK
    int hashCode();  // Overrides Object.hashCode, OK
    String toString();  // Overrides Object.toString, but this is abstract!
}

// ✅ Correct - only 1 abstract method
@FunctionalInterface
interface Good<T> {
    T process(T input);  // 1 abstract method
    
    default void log() {  // Default OK
        System.out.println("Processing...");
    }
    
    static Good<String> identity() {  // Static OK
        return s -> s;
    }
    
    @Override
    boolean equals(Object o);  // Override Object method OK
}
```

## References

1. [Baeldung - Java Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces)
2. [DigitalOcean - Java Functional Interfaces Tutorial](https://www.digitalocean.com/community/tutorials/java-8-functional-interfaces)
3. [Baeldung - Java BiFunction Interface](https://www.baeldung.com/java-bifunction-interface)

**Related**: See [Streams](streams.md) for data processing with functional interfaces using Stream API.
