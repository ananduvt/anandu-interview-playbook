# Strings & Primitives

## Primitive Data Types

[https://www.programiz.com/java-programming/variables-primitive-data-types](https://www.programiz.com/java-programming/variables-primitive-data-types)
[https://www.baeldung.com/java-primitives#:\~:text=2.-,Primitive%20Data%20Types,objects%20and%20represent%20raw%20values](https://www.baeldung.com/java-primitives#:~:text=2.-,Primitive%20Data%20Types,objects%20and%20represent%20raw%20values).
[https://www.geeksforgeeks.org/data-types-in-java/](https://www.geeksforgeeks.org/data-types-in-java/)
[https://medium.com/@AlexanderObregon/java-data-types-primitive-vs-non-primitive-417925cee746](https://medium.com/@AlexanderObregon/java-data-types-primitive-vs-non-primitive-417925cee746)

| Type | Size (bits) | Minimum | Maximum | Range | Example | Use Case |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| byte | 8 | -27 | 27– 1 | -128 to 127 | byte b = 100; | Very small integers, memory-efficient in arrays |
| short | 16 | -215 | 215– 1 | 32,768 to 32,767 | short s = 30_000; | Small integers, more memory-efficient than int |
| int | 32 | -231 | 231– 1 | -2,147,483,648 to 2,147,483,647 | int i = 100_000_000; | Default integer type |
| long | 64 | -263 | 263– 1 | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | long l = 100_000_000_000_000; | Large integer values |
| float | 32 | -2-149 | (2-2-23)·2127 | Sufficient for storing 6 to 7 decimal digitsIEEE 754 floating-point | float f = 1.456f; | Fractional numbers, less precise than double |
| double | 64 | -2-1074 | (2-2-52)·21023 | Sufficient for storing 15 to 16 decimal digitsIEEE 754 floating-point | double f = 1.456789012345678; | Default for decimal values, high precision |
| char | 16 | 0 | 216– 1 | 0 to 65,535 a single character/letter or ASCII values | char c = ‘c’; | Single Unicode characters |
| boolean | 1 | – | – | true or false values | boolean b = true; | Logical values for conditions and flags |

## Integer Implementation

In Java, the int data type is a primitive type that represents a 32-bit signed two's complement integer. This means that each int variable occupies 4 bytes (32 bits) of memory. One bit is used to represent the sign of the integer (positive or negative), and the remaining 31 bits are used to represent the magnitude of the integer.

Java also provides the Integer class, which is a wrapper class for the primitive int type. Integer objects are stored on the heap and have additional overhead compared to primitive int variables. Each Integer object typically occupies 16 bytes of memory, which includes the object header, the int value, and some padding.

## Double Implementation

In Java, the double data type is a primitive type that represents a double-precision 64-bit IEEE 754 floating-point number. It is used to store decimal numbers with a higher degree of precision compared to the float data type.

When a double variable is declared, the Java Virtual Machine (JVM) allocates 64 bits (8 bytes) of memory to store its value. This memory is typically allocated on the heap, which is the area of memory used for dynamic memory allocation in Java.

The 64 bits are divided into three parts:
Sign bit (1 bit): Indicates whether the number is positive or negative.
Exponent bits (11 bits): Represents the exponent of the number.
Significand bits (52 bits): Represents the fractional part of the number.

The value of a double is calculated as follows:

**value = (-1)^sign \* 2^(exponent - bias) \* (1 + significand)**

where:
sign is the sign bit (0 for positive, 1 for negative).
exponent is the value of the exponent bits.
bias is a constant value (1023 for double).
significand is the value of the significand bits, normalized to be between 0 and 1.

The double data type can represent a wide range of values, from approximately 4.9e-324 to 1.7e+308, with a precision of about 15 decimal digits.

When performing operations on double values, the JVM uses floating-point arithmetic, which can sometimes result in rounding errors. This is because not all decimal numbers can be represented exactly in binary.

## Integer Pool

To optimize memory usage, Java uses an Integer Pool (or cache) for Integer objects with values in the range of -128 to 127. When an Integer object is created within this range using autoboxing or the valueOf() method, the JVM reuses existing objects from the pool instead of creating new ones. This can significantly reduce memory consumption when working with frequently used integer values.

Integer a = 100;
Integer b = 100;
System.out.println(a == b); // Output: true (same object from the pool)

Integer c = 200;
Integer d = 200;
System.out.println(c == d); // Output: false (different objects)

When an int variable is declared, the memory is allocated directly on the stack. When an Integer object is created using the new keyword, the memory is allocated on the heap. The garbage collector automatically reclaims memory occupied by Integer objects that are no longer in use.

## String Pool

* The string pool in Java is a special area of memory within the heap that stores unique string literals. When you create a string using string literals (e.g., "hello"), Java first checks if an identical string already exists in the string pool. If it does, Java reuses the existing string from the pool instead of creating a new one. This helps conserve memory and improve performance by reducing the number of duplicate strings.
* String literals in Java are automatically interned, meaning they are added to the string pool by default. Additionally, you can **explicitly intern strings using the intern() method**, which adds the string to the pool if it's not already present and returns a reference to the canonical representation.
  Eg:
  String s1 = "hello";  // Added to the string pool
  String s2 = "hello";  // Reuses the existing string from the pool

  System.out.println(s1 == s2);  // Output: true (same reference)

  String s3 = new String("hello");  // Creates a new string object in the heap
  String s4 = s3.intern();           // Adds s3 to the string pool and returns a reference to it

  System.out.println(s1 == s3);  // Output: false (different references)
  System.out.println(s1 == s4);  // Output: true (s4 references the interned string from the pool)

## Data Structures

## Difference Between String, StringBuilder, and StringBuffer

[https://www.geeksforgeeks.org/string-vs-stringbuilder-vs-stringbuffer-in-java/](https://www.geeksforgeeks.org/string-vs-stringbuilder-vs-stringbuffer-in-java/)
[https://medium.com/@salvipriya97/string-vs-stringbuilder-vs-stringbuffer-which-one-to-choose-4308dbcc3022](https://medium.com/@salvipriya97/string-vs-stringbuilder-vs-stringbuffer-which-one-to-choose-4308dbcc3022)
[https://www.digitalocean.com/community/tutorials/string-vs-stringbuffer-vs-stringbuilder](https://www.digitalocean.com/community/tutorials/string-vs-stringbuffer-vs-stringbuilder)

| Feature | String | StringBuilder | StringBuffer |
| :---- | :---- | :---- | :---- |
| **Introduction** | Introduced in JDK 1.0 | Introduced in JDK 1.5 | Introduced in JDK 1.0 |
| **Mutability** | Immutable | Mutable | Mutable |
| **Thread Safety** | Thread Safe | Not Thread Safe | Thread Safe |
| **Memory Efficiency** | High | Efficient | Less Efficient |
| **Performance** | High(No-Synchronization) | High(No-Synchronization) | Low(Due to Synchronization) |
| **Usage** | This is used when we want immutability. | This is used when Thread safety is not required. | This is used when Thread safety is required. |
| **Pros** | Thread-safe (immutable), making it suitable for use in multi-threaded environments. Predictable behavior in terms of memory usage. | Mutable, making it suitable for building or modifying strings dynamically. Efficient for concatenating or manipulating strings in a loop or when dealing with frequent modifications. | Mutable and thread-safe, making it suitable for multi-threaded applications. Efficient for concatenating or modifying strings when thread safety is required. |
| **Cons** | Inefficient for frequent string manipulation, as it creates new objects with each modification, leading to performance overhead. | Not thread-safe. If used in a multi-threaded environment, additional synchronization is required (use StringBuffer for thread safety). | Slightly less efficient than StringBuilder due to thread safety overhead. It's generally recommended for multi-threaded scenarios. |

**Which to Choose:**

1. If you need to manipulate strings dynamically and thread safety is not a concern, **StringBuilder** is the preferred choice due to its efficiency.
2. If you require thread safety, especially in a multi-threaded environment, use **StringBuffer**.
3. Use **String** when you need an immutable string, and you don't anticipate frequent modifications.
