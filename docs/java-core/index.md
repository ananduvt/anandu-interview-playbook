# Java Core


## Brief History

Java was created in **1991** by **James Gosling** and colleagues at Sun Microsystems, designed with the principle "Write Once, Run Anywhere" (WORA). The language officially launched as Java 1.0 in **1996**, introducing the JVM, classes, and the revolutionary concept of platform independence through bytecode compilation. Over three decades, Java evolved from applets and server-side applications to becoming the dominant language for enterprise systems, microservices, and distributed computing. Today, maintained by Oracle (since acquiring Sun in 2010), Java remains one of the most widely deployed programming languages globally, powering millions of systems across finance, healthcare, e-commerce, and cloud infrastructure.

## Authors & Contributors

- **James Gosling** — Inventor of Java; visionary behind WORA and JVM design
- **Mike Sheridan, Patrick Naughton** — Co-creators at Sun Microsystems
- **Oracle Corporation** — Owner and maintainer since 2010; drives JDK enhancements via JEP (Java Enhancement Proposals)
- **OpenJDK Community** — Open-source implementation; enables transparency and community contributions

**Major Impact**: Java's portability revolutionized software distribution; the JVM became the foundation for modern languages (Kotlin, Scala, Clojure). Enterprise adoption of Java middleware (Spring, Jakarta EE) set the standard for scalable backend systems. Java's garbage collection, concurrency libraries, and ecosystem maturity remain unmatched in production environments.

## Topics

Foundational Java concepts: the platform (JVM/JRE/JDK), language semantics, object-oriented principles, exception handling, and version-specific features. Essential for any Java interview.

---
### [Basics](basics.md)
**JDK - JRE - JVM - Bytecode - ClassLoader - Editions**  
Platform architecture: JDK/JRE/JVM distinctions, bytecode execution, ClassLoader lifecycle, Java SE/EE/ME/JavaFX editions, JIT compilation.

### [OOP (Object-Oriented Programming)](oop.md)  
**Classes - Inheritance - Polymorphism - Encapsulation - SOLID - Abstraction**  
Object-oriented principles: method overriding/overloading, access modifiers, super/this, abstract classes vs. interfaces, composition patterns.

### [Strings & Primitives](strings-primitives.md)  
**String Pool - Immutability - Primitives - Autoboxing - Type Conversion - Collections**  
String mechanics: immutability, StringBuilder vs. StringBuffer, wrapper types, autoboxing/unboxing, int ↔ String conversions, array/list/stream transformations.

### [Exceptions](exceptions.md)  
**Exception Hierarchy - Try-Catch-Finally - Checked vs. Unchecked - Custom Exceptions - Try-With-Resources**  
Error handling: exception propagation, custom exceptions, try-with-resources (AutoCloseable), stack traces, recovery patterns.

### [Versions & Features](versions-features.md)  
**Java 8 - 11 - 17 - 21 - LTS - Feature Timeline - Virtual Threads**  
Version history: Java 1.0–24 timeline, LTS releases (8/11/17/21), marquee features (lambdas → HTTP/2 → sealed classes → virtual threads), support windows.

---

## Quick Interview Prep

**Know these for any Java role:**
- **JVM basics**: Bytecode, ClassLoader phases, garbage collection (see [Java Patterns & Advanced](../java-patterns-advanced/index.md))
- **OOP**: 4 pillars, SOLID principles, composition over inheritance
- **Core APIs**: String, Collections (see [Java Collections & Concurrency](../java-collections-concurrency/index.md)), Streams (see [Java Collections & Concurrency](../java-collections-concurrency/index.md))
- **Exception handling**: Checked vs. unchecked, try-with-resources
- **LTS versions**: Java 8 (lambdas/streams), 11 (HTTP/2, var), 17 (sealed classes), 21 (virtual threads)
