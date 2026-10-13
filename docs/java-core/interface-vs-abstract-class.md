# Interface vs Abstract Class

Interfaces and abstract classes are both used for abstraction in Java, but they serve different purposes and have different constraints. Understanding when to use each is critical for good design.

---

## Quick Comparison

| Feature | Interface | Abstract Class |
|---------|-----------|-----------------|
| **Inheritance** | A class can implement multiple interfaces | A class can extend only one abstract class |
| **Methods** | Default: public abstract; can have default/static/private methods (Java 8+) | Can have abstract and concrete methods |
| **State** | Can't have instance variables (only constants) | Can have instance variables (any visibility) |
| **Constructor** | Can't have constructors | Can have constructors (to initialize fields) |
| **Access Modifiers** | Methods are public; fields are public static final | Can use any modifier (public, private, protected) |
| **When to use** | Define a contract/capability | Share code and state across related classes |
| **Relationship** | "Can do" (capability) | "Is a" (inheritance hierarchy) |

---

## Detailed Comparison

### **Interfaces: Defining Contracts**

An interface defines **what a class should do**, not how it does it. It's a contract.

```java
// Interface: defines capability
public interface PaymentProcessor {
    
    // Abstract method (must be implemented)
    PaymentResponse processPayment(Order order);
    
    // Default method (optional override)
    default void logTransaction(Order order) {
        System.out.println("Processed: " + order.getId());
    }
    
    // Static method (utility)
    static PaymentProcessor getDefault() {
        return new CreditCardProcessor();
    }
    
    // Constant (public static final)
    String CURRENCY = "USD";
}

// Multiple implementations
public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public PaymentResponse processPayment(Order order) {
        // Credit card logic
        return new PaymentResponse("SUCCESS");
    }
}

public class PayPalProcessor implements PaymentProcessor {
    @Override
    public PaymentResponse processPayment(Order order) {
        // PayPal logic
        return new PaymentResponse("SUCCESS");
    }
}

// A class can implement multiple interfaces
public class HybridProcessor implements PaymentProcessor, RefundProcessor, NotificationProcessor {
    // Must implement all three interfaces
}
```

**Key Points:**
- ✅ Multiple implementation (polymorphism)
- ✅ Loose coupling (depends on interface, not concrete class)
- ✅ Mixin behavior (class can be many things)
- ❌ Can't share state (only constants)

---

### **Abstract Classes: Sharing Implementation**

An abstract class defines **what subclasses should be** and provides shared code/state.

```java
// Abstract class: defines hierarchy and shared behavior
public abstract class Animal {
    
    // Shared state (instance variable)
    private String name;
    private int age;
    
    // Constructor to initialize state
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Abstract method (subclasses must implement)
    public abstract void makeSound();
    
    // Concrete method (shared by all subclasses)
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Getter (access to shared state)
    public String getName() {
        return name;
    }
}

// Single inheritance
public class Dog extends Animal {
    public Dog(String name, int age) {
        super(name, age);  // Initialize parent state
    }
    
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

// ❌ This won't work (can't extend multiple classes)
public class Hybrid extends Dog, Cat { }  // ERROR
```

**Key Points:**
- ✅ Share state and behavior
- ✅ Constructor to initialize shared state
- ✅ Protects internal implementation
- ❌ Single inheritance only
- ❌ Tighter coupling between parent and child

---

## When to Use Each

### **Use Interface When:**

```java
// 1. Define a capability/behavior unrelated to class hierarchy
public interface Serializable {
    byte[] toBytes();
    void fromBytes(byte[] data);
}

// Any class can implement it (String, List, Order, etc.)
public class Order implements Serializable {
    // ...
}

public class User implements Serializable {
    // ...
}

// 2. Multiple inheritance of behavior
public class PaymentService implements PaymentProcessor, AuditLogger, NotificationSender {
    // Does multiple things
}

// 3. Strategy pattern (plug-and-play implementations)
public interface ReportGenerator {
    Report generate(Data data);
}

public class PdfReportGenerator implements ReportGenerator { }
public class ExcelReportGenerator implements ReportGenerator { }
public class HtmlReportGenerator implements ReportGenerator { }

// Client doesn't care which implementation
public class ReportingService {
    private ReportGenerator generator;  // Can be any implementation
    
    public Report buildReport(Data data) {
        return generator.generate(data);  // Polymorphic call
    }
}
```

### **Use Abstract Class When:**

```java
// 1. Share code and state across related classes
public abstract class Vehicle {
    private int speed;  // Shared state
    private String color;
    
    public abstract void accelerate();
    
    public void stop() {
        speed = 0;  // Shared behavior using shared state
        System.out.println("Stopped");
    }
}

public class Car extends Vehicle {
    @Override
    public void accelerate() {
        speed += 20;
    }
}

// 2. Define a class hierarchy with "is-a" relationship
public abstract class Shape {
    public abstract double getArea();
    public abstract double getPerimeter();
}

public class Circle extends Shape {
    private double radius;
    
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}

// 3. Enforce consistent initialization
public abstract class Database {
    protected Connection conn;
    
    public Database(String url) {
        // Guaranteed initialization path
        this.conn = DriverManager.getConnection(url);
        initialize();
    }
    
    protected abstract void initialize();
}

public class MySQLDatabase extends Database {
    public MySQLDatabase(String url) {
        super(url);  // Parent constructor ensures setup
    }
    
    @Override
    protected void initialize() {
        // MySQL-specific init
    }
}

// 4. Access control (protected methods)
public abstract class BaseService {
    
    protected void log(String msg) {
        System.out.println(msg);  // Subclasses can call
    }
    
    public abstract void execute();
}

public class MyService extends BaseService {
    @Override
    public void execute() {
        log("Starting...");  // Can use protected method
    }
}
```

---

## Real-World Scenarios

### Scenario 1: Payment Processing

```java
// ✅ Use Interface: Different payment methods (credit card, PayPal, crypto)
public interface PaymentMethod {
    PaymentResponse pay(Order order);
}

public class CreditCardPayment implements PaymentMethod { }
public class PayPalPayment implements PaymentMethod { }
public class CryptoPayment implements PaymentMethod { }

// Multiple implementations, no shared state
```

### Scenario 2: Data Access

```java
// ✅ Use Abstract Class: Shared connection logic, different databases
public abstract class BaseRepository {
    protected Connection conn;  // Shared state
    
    public BaseRepository(String url) {
        this.conn = connect(url);  // Shared initialization
    }
    
    protected abstract List<?> query(String sql);
}

public class UserRepository extends BaseRepository {
    public UserRepository(String url) {
        super(url);
    }
    
    @Override
    protected List<?> query(String sql) {
        // MySQL-specific querying
    }
}

public class OrderRepository extends BaseRepository {
    // Shares connection logic with UserRepository
}
```

### Scenario 3: Plugin System

```java
// ✅ Use Interface: Plugins should be loosely coupled
public interface Plugin {
    void onLoad();
    void onEvent(Event e);
    void onUnload();
}

public class AuthPlugin implements Plugin { }
public class CachePlugin implements Plugin { }
public class MonitoringPlugin implements Plugin { }

// Main app doesn't care about plugin internals
public class PluginManager {
    private List<Plugin> plugins = new ArrayList<>();
    
    public void loadPlugins(String path) {
        // Discover and load any Plugin implementation
    }
}
```

### Scenario 4: Template Method Pattern

```java
// ✅ Use Abstract Class: Define algorithm skeleton, let subclasses fill in steps
public abstract class DataProcessor {
    
    public final void process(Data data) {
        // Template: algorithm skeleton (final prevents override)
        validate(data);
        transform(data);
        save(data);
        notifyCompletion(data);
    }
    
    protected abstract void validate(Data data);
    protected abstract void transform(Data data);
    protected abstract void save(Data data);
    
    protected void notifyCompletion(Data data) {
        System.out.println("Done: " + data.getId());
    }
}

public class ReportProcessor extends DataProcessor {
    @Override
    protected void validate(Data data) { }
    
    @Override
    protected void transform(Data data) { }
    
    @Override
    protected void save(Data data) { }
}
```

---

## Java 8+ Default Methods: Blurring the Line

Java 8 introduced default methods in interfaces, making them more like abstract classes:

```java
// Modern interface with default implementation
public interface Repository<T> {
    
    // Abstract: subclasses must implement
    T findById(Long id);
    
    // Default: subclasses can override, but don't have to
    default List<T> findAll() {
        return Collections.emptyList();
    }
    
    // Static utility method
    static <T> Repository<T> getEmpty() {
        return new Repository<T>() {
            @Override
            public T findById(Long id) { return null; }
        };
    }
}

// Subclass can skip implementing findAll() if not needed
public class UserRepository implements Repository<User> {
    @Override
    public User findById(Long id) { }
    
    // Inherits default findAll()
}
```

**This doesn't make interfaces obsolete:**
- Interfaces still can't have state (instance variables)
- Interfaces still express "can do" (capability)
- Abstract classes still express "is a" (hierarchy)

---

## Decision Tree

```
                ┌─ Do you need shared STATE?
                │  (instance variables)
                │  ├─ YES → Abstract Class
                │  └─ NO → Could be interface
                │
     Problem    ├─ Do you need CONSTRUCTORS?
       Design   │  (to initialize objects)
         ?      │  ├─ YES → Abstract Class
                │  └─ NO → Could be interface
                │
                ├─ Do you need SINGLE inheritance?
                │  (only one parent)
                │  ├─ YES → Could be either
                │  └─ NO (multiple parents) → Interface
                │
                └─ Is this a CLASS HIERARCHY?
                   (X is-a Y)
                   ├─ YES → Abstract Class
                   └─ NO (just capabilities) → Interface
```

---

## Interview Talking Points

- **Key difference?** Interfaces define "what can be done"; abstract classes define "what things are"
- **State?** Interfaces have no instance variables; abstract classes do
- **Inheritance?** Multiple interfaces, single abstract class (per class)
- **Constructors?** Abstract classes have them (initialize state); interfaces don't
- **When to use which?** State + hierarchy = abstract class; capability + multiple types = interface
- **Default methods?** Java 8+ interfaces can have default implementations, but still no state
- **Mixin behavior?** Use interfaces; abstract classes are for inheritance hierarchies
- **Template method?** Abstract classes (define skeleton); interfaces just define contract

---

**See also:** [OOP Concepts](oop.md), [Design Patterns](../java-patterns-advanced/design-patterns.md)
