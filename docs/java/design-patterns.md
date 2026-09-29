# Design Patterns

Grouped into Creational, Structural, Behavioral (Gang of Four).

## Creational
- **Singleton** — one instance. Thread-safe via enum, or double-checked locking with `volatile`.
- **Factory Method** — subclass decides which object to create.
- **Abstract Factory** — families of related objects.
- **Builder** — step-by-step construction of complex/immutable objects (fluent API).
- **Prototype** — clone existing objects.

## Structural
- **Adapter** — convert one interface to another.
- **Decorator** — add behavior dynamically without subclassing (`java.io` streams).
- **Facade** — simplified interface over a subsystem.
- **Proxy** — placeholder controlling access (lazy, security, remoting; Spring AOP).
- **Composite** — tree of uniform objects.

## Behavioral
- **Strategy** — interchangeable algorithms behind an interface.
- **Observer** — publish/subscribe notifications.
- **Template Method** — skeleton in base, steps overridden by subclasses.
- **Command** — encapsulate a request as an object.
- **Iterator**, **State**, **Chain of Responsibility**.

## In Java / Spring
- Singleton (Spring beans), Factory (`BeanFactory`), Proxy (AOP), Template (`JdbcTemplate`),
  Builder (`StringBuilder`, `Stream.Builder`), Strategy (`Comparator`).

## Thread-safe singleton
```java
public enum Config { INSTANCE; }          // simplest, serialization-safe
// or double-checked locking:
private static volatile Config instance;
public static Config get() {
    if (instance == null) synchronized (Config.class) {
        if (instance == null) instance = new Config();
    }
    return instance;
}
```
