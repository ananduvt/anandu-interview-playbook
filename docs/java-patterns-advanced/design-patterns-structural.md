# Structural Design Patterns

Structural patterns describe how classes and objects are composed into larger structures while keeping them flexible. This page covers the 7 structural GoF patterns; see the [Design Patterns catalog](design-patterns.md) for the full list of creational, structural and behavioral patterns.

## Comparison at a glance

| Pattern | Intent | Key difference |
| :-- | :-- | :-- |
| Adapter | Make an incompatible interface fit the one the client expects | Changes the interface; no new behavior |
| Bridge | Split abstraction from implementation so both vary independently | Designed up front; two hierarchies linked by composition |
| Composite | Treat single objects and trees of objects uniformly | Recursive tree structure with one shared interface |
| Decorator | Add responsibilities to an object dynamically | Same interface in and out; stackable behavior |
| Facade | Provide one simple entry point to a complex subsystem | New, simpler interface; subsystem stays accessible |
| Flyweight | Share fine-grained objects to save memory | Splits state into shared (intrinsic) and per-use (extrinsic) |
| Proxy | Control access to an object through a stand-in | Same interface; controls access (lazy, security, remote, cache) |

!!! tip "Adapter vs Decorator vs Proxy vs Facade"
    - **Adapter**: different interface in, expected interface out.
    - **Decorator**: same interface, adds behavior the client wants.
    - **Proxy**: same interface, controls *whether/when/how* the real object is reached.
    - **Facade**: new, simpler interface over *many* objects.

---

## Adapter

### Intent
Convert the interface of an existing class into the interface the client expects, so classes with incompatible interfaces can work together. Also called Wrapper.

### When to use
- Integrating a legacy or third-party class you cannot modify.
- Isolating your domain from an external API (payment gateway, SDK).
- Reusing several existing classes that lack a common interface.

### Example

```java
// Target interface the application expects
interface PaymentProcessor {
    void pay(String orderId, long amountCents);
}

// Adaptee: third-party class with an incompatible API
class LegacyGateway {
    void makeTransaction(String ref, double amount, String currency) {
        System.out.println("Legacy charge " + amount + " " + currency + " for " + ref);
    }
}

// Adapter: implements the target, delegates to the adaptee
class LegacyGatewayAdapter implements PaymentProcessor {
    private final LegacyGateway gateway;

    LegacyGatewayAdapter(LegacyGateway gateway) { this.gateway = gateway; }

    @Override
    public void pay(String orderId, long amountCents) {
        gateway.makeTransaction(orderId, amountCents / 100.0, "USD");
    }
}

public class AdapterDemo {
    public static void main(String[] args) {
        PaymentProcessor p = new LegacyGatewayAdapter(new LegacyGateway());
        p.pay("ORD-1", 2599);   // Legacy charge 25.99 USD for ORD-1
    }
}
```

### Real-world Java/Spring usage
- `Arrays.asList(array)` adapts an array to the `List` interface (fixed-size, writes through to the array).
- `InputStreamReader` adapts a byte `InputStream` to a character `Reader`.
- `Collections.enumeration(c)` / `Enumeration.asIterator()` adapt between `Iterator` and `Enumeration`.
- Spring MVC `HandlerAdapter` lets `DispatcherServlet` invoke different handler types uniformly; Spring's `MessageListenerAdapter` adapts a POJO to a JMS listener.

### Trade-offs
- Isolates change and enables reuse without touching the adaptee.
- Adds a class and a layer of indirection; too many adapters signal a poor-fit design.
- Object adapter (composition, shown above) is preferred over class adapter (inheritance) in Java, which has single class inheritance.

### Interview tip
Say that an Adapter changes the *interface* but not the behavior, then name `InputStreamReader` or `Arrays.asList`. Contrast with Decorator (same interface) to show you know the difference.

---

## Bridge

### Intent
Decouple an abstraction from its implementation so the two can vary independently. Prefer composition over an exploding inheritance hierarchy.

### When to use
- You have two orthogonal dimensions of variation (for example shape x renderer, notification type x channel).
- You want to switch implementations at runtime.
- Subclassing would produce a combinatorial explosion (`RedCircle`, `BlueCircle`, `RedSquare`, ...).

### Example

```java
// Implementor hierarchy
interface Channel {
    void deliver(String text);
}

class EmailChannel implements Channel {
    public void deliver(String text) { System.out.println("EMAIL: " + text); }
}

class SmsChannel implements Channel {
    public void deliver(String text) { System.out.println("SMS: " + text); }
}

// Abstraction hierarchy holds a reference to the implementor (the "bridge")
abstract class Notification {
    protected final Channel channel;

    protected Notification(Channel channel) { this.channel = channel; }

    abstract void send(String message);
}

class AlertNotification extends Notification {
    AlertNotification(Channel channel) { super(channel); }

    void send(String message) { channel.deliver("[ALERT] " + message); }
}

class ReminderNotification extends Notification {
    ReminderNotification(Channel channel) { super(channel); }

    void send(String message) { channel.deliver("Reminder: " + message); }
}

public class BridgeDemo {
    public static void main(String[] args) {
        new AlertNotification(new SmsChannel()).send("Server down");
        new ReminderNotification(new EmailChannel()).send("Standup at 10");
    }
}
```

With 2 notification types and 2 channels you write 4 classes instead of the 4 combinations; add a third channel and you add 1 class, not 2.

### Real-world Java/Spring usage
- **JDBC**: `java.sql.Connection`/`Statement` (abstraction) are separate from vendor `Driver` implementations; `DriverManager` picks the implementation.
- **SLF4J**: the `Logger` API is bridged to Logback, Log4j 2 or `java.util.logging` bindings.
- AWT: `Component` peers (historical example in the GoF book).

### Trade-offs
- Both hierarchies evolve independently; implementation can be swapped at runtime.
- More upfront design and more classes; overkill when only one dimension varies.

### Interview tip
Bridge is decided at design time (two hierarchies), while Adapter is applied after the fact to make existing classes fit. Use the JDBC example.

---

## Composite

### Intent
Compose objects into tree structures and let clients treat individual objects (leaves) and compositions (nodes) through the same interface.

### When to use
- Part-whole hierarchies: file systems, UI components, menus, org charts, expression trees.
- Client code should not distinguish between one item and a group of items.

### Example

```java
import java.util.ArrayList;
import java.util.List;

interface FileSystemItem {
    long size();
    void print(String indent);
}

class FileItem implements FileSystemItem {
    private final String name;
    private final long bytes;

    FileItem(String name, long bytes) { this.name = name; this.bytes = bytes; }

    public long size() { return bytes; }
    public void print(String indent) { System.out.println(indent + name + " (" + bytes + ")"); }
}

class Folder implements FileSystemItem {
    private final String name;
    private final List<FileSystemItem> children = new ArrayList<>();

    Folder(String name) { this.name = name; }

    Folder add(FileSystemItem item) { children.add(item); return this; }

    public long size() {
        return children.stream().mapToLong(FileSystemItem::size).sum();
    }

    public void print(String indent) {
        System.out.println(indent + name + "/ (" + size() + ")");
        children.forEach(c -> c.print(indent + "  "));
    }
}

public class CompositeDemo {
    public static void main(String[] args) {
        Folder root = new Folder("root")
            .add(new FileItem("a.txt", 100))
            .add(new Folder("docs").add(new FileItem("b.md", 250)));
        root.print("");
        System.out.println(root.size());   // 350
    }
}
```

### Real-world Java/Spring usage
- Swing/AWT: `Container` is a `Component` that holds other `Component`s.
- Jakarta Faces `UIComponent` trees; XML/HTML DOM (`org.w3c.dom.Node`).
- `java.nio.file.FileVisitor` walks a tree; Spring Security's `CompositeFilter` and Spring's `CompositeCacheManager` treat a group like a single instance.

### Trade-offs
- Simple client code and easy to add new node types.
- The shared interface can be too general (should a leaf have `add()`?). The "safe" variant keeps child-management methods only on the composite; the "transparent" variant puts them on the interface.
- Deep trees need care with recursion (stack depth) and cycles.

### Interview tip
Explain the recursion: composite operations delegate to children. Mention the transparency vs safety trade-off for child-management methods.

---

## Decorator

### Intent
Attach additional responsibilities to an object dynamically by wrapping it in an object with the same interface. A flexible alternative to subclassing.

### When to use
- Add or remove behavior at runtime, per instance.
- Combine optional features in many combinations (compression + buffering + encryption).
- Extending by subclassing would cause class explosion or the class is `final`.

### Example

```java
interface TextSource {
    String read();
}

class PlainText implements TextSource {
    private final String text;

    PlainText(String text) { this.text = text; }

    public String read() { return text; }
}

// Base decorator: same interface, holds a wrapped instance
abstract class TextDecorator implements TextSource {
    protected final TextSource inner;

    protected TextDecorator(TextSource inner) { this.inner = inner; }
}

class UpperCase extends TextDecorator {
    UpperCase(TextSource inner) { super(inner); }

    public String read() { return inner.read().toUpperCase(); }
}

class Trimmed extends TextDecorator {
    Trimmed(TextSource inner) { super(inner); }

    public String read() { return inner.read().trim(); }
}

public class DecoratorDemo {
    public static void main(String[] args) {
        TextSource src = new UpperCase(new Trimmed(new PlainText("  hello  ")));
        System.out.println(src.read());   // HELLO
    }
}
```

### Real-world Java/Spring usage
- **`java.io`**: `new BufferedInputStream(new GZIPInputStream(new FileInputStream(f)))`; `BufferedReader`, `DataOutputStream` and `PushbackInputStream` all extend `FilterInputStream`/`FilterOutputStream` or wrap a `Reader`/`Writer`.
- **Collections wrappers**: `Collections.unmodifiableList`, `synchronizedList`, `checkedList` wrap a list with the same `List` interface.
- Servlet `HttpServletRequestWrapper`; Spring `ServerHttpRequestDecorator`, `BeanPostProcessor`-based wrapping, `TransactionAwareDataSourceProxy`.

### Trade-offs
- Behavior composed at runtime; follows open/closed and single responsibility.
- Many small objects; stack traces and debugging get harder; order of wrapping matters (`Trimmed` then `UpperCase` differs from the reverse in general).
- Identity checks break: the decorator is not `==` to the wrapped object.

### Interview tip
Open with `java.io`: "`new BufferedReader(new InputStreamReader(System.in))` is a decorator on top of an adapter." Then contrast: Decorator adds behavior, Proxy controls access.

---

## Facade

### Intent
Provide a unified, simplified interface to a set of interfaces in a subsystem, making the subsystem easier to use.

### When to use
- A subsystem has many classes and clients need only a common use case.
- You want to layer a system and define entry points (service layer over repositories and clients).
- You want to reduce coupling between clients and subsystem internals.

### Example

```java
class InventoryService {
    boolean reserve(String sku, int qty) { System.out.println("Reserved " + qty + " x " + sku); return true; }
}

class PaymentService {
    boolean charge(String customerId, long cents) { System.out.println("Charged " + cents); return true; }
}

class ShippingService {
    void schedule(String orderId) { System.out.println("Shipping scheduled for " + orderId); }
}

// Facade: one method hides the orchestration
class OrderFacade {
    private final InventoryService inventory = new InventoryService();
    private final PaymentService payment = new PaymentService();
    private final ShippingService shipping = new ShippingService();

    public boolean placeOrder(String orderId, String customerId, String sku, int qty, long cents) {
        if (!inventory.reserve(sku, qty)) return false;
        if (!payment.charge(customerId, cents)) return false;
        shipping.schedule(orderId);
        return true;
    }
}

public class FacadeDemo {
    public static void main(String[] args) {
        boolean ok = new OrderFacade().placeOrder("ORD-1", "C-9", "SKU-42", 2, 4999);
        System.out.println(ok);   // true
    }
}
```

This sketch omits compensation (releasing stock if payment fails); a real facade across services needs a rollback or saga strategy.

### Real-world Java/Spring usage
- A Spring `@Service` that orchestrates several repositories and clients is a facade for the controller layer.
- `JdbcTemplate` hides connection, statement and exception handling; `RestClient`/`WebClient` hide HTTP plumbing.
- `java.net.http.HttpClient` (Java 11) and `javax.faces.context.FacesContext` are facades over larger subsystems.
- SLF4J's `LoggerFactory` is a facade over logging frameworks.

### Trade-offs
- Simpler client code, loose coupling, easier layering.
- Risk of a "god object" that accumulates unrelated logic.
- Does not prevent clients from using the subsystem directly (unless you enforce it with package visibility or modules).

### Interview tip
Distinguish from Adapter: a Facade *simplifies* many classes with a new interface, an Adapter *converts* one interface to another. The Spring service layer is an easy real example.

---

## Flyweight

### Intent
Use sharing to support large numbers of fine-grained objects efficiently. Split state into **intrinsic** (immutable, shared) and **extrinsic** (supplied by the caller per use).

### When to use
- An application creates huge numbers of similar objects and memory is the bottleneck.
- Most object state can be made extrinsic and the shared part is immutable.
- Object identity does not matter.

### Example

```java
import java.util.HashMap;
import java.util.Map;

// Intrinsic state (shared, immutable): the glyph's font data
record GlyphStyle(String font, int size, boolean bold) {}

// Flyweight factory: returns the shared instance for equal state
class GlyphStyleFactory {
    private static final Map<GlyphStyle, GlyphStyle> CACHE = new HashMap<>();

    static GlyphStyle get(String font, int size, boolean bold) {
        return CACHE.computeIfAbsent(new GlyphStyle(font, size, bold), s -> s);
    }

    static int cacheSize() { return CACHE.size(); }
}

// Extrinsic state (x, y, char) lives in the context object
record PlacedGlyph(char ch, int x, int y, GlyphStyle style) {}

public class FlyweightDemo {
    public static void main(String[] args) {
        PlacedGlyph a = new PlacedGlyph('A', 0, 0, GlyphStyleFactory.get("Arial", 12, false));
        PlacedGlyph b = new PlacedGlyph('B', 10, 0, GlyphStyleFactory.get("Arial", 12, false));
        System.out.println(a.style() == b.style());          // true: shared
        System.out.println(GlyphStyleFactory.cacheSize());   // 1
    }
}
```

The `HashMap` cache is not thread-safe; use `ConcurrentHashMap` in multi-threaded code.

### Real-world Java/Spring usage
- `Integer.valueOf(int)` caches -128 to 127 (also `Byte`, `Short`, `Long`, `Character` up to 127, and `Boolean.TRUE/FALSE`). Hence `Integer.valueOf(127) == Integer.valueOf(127)` is `true` but `Integer.valueOf(128) == Integer.valueOf(128)` is `false` in default settings.
- The **String pool** (`"abc"` literals, `String.intern()`).
- `Enum` constants and `Collections.emptyList()` are shared immutable instances.
- Connection or thread pools share objects but are the *Object Pool* pattern, not Flyweight, because pooled objects are borrowed exclusively and are mutable.

### Trade-offs
- Large memory savings when many duplicates exist.
- More complexity; extrinsic state must be passed around; shared objects must be immutable and thread-safe.
- Lookup cost in the factory; premature use adds complexity with little gain.

### Interview tip
The classic trap is `Integer` caching with `==`. Explain it as a Flyweight, then add that you should compare boxed values with `equals`. Also be ready to contrast Flyweight with Object Pool.

---

## Proxy

### Intent
Provide a surrogate or placeholder for another object to control access to it. The proxy has the same interface as the real subject.

### When to use
- **Virtual proxy**: lazy initialization of an expensive object.
- **Protection proxy**: access control or authorization.
- **Remote proxy**: local stand-in for a remote service (RMI stubs, Feign clients).
- **Caching, logging, metrics, transactions**: cross-cutting behavior without touching the target.

### Example 1: static proxy (lazy loading)

```java
interface Image {
    void display();
}

class HighResImage implements Image {
    private final String file;

    HighResImage(String file) {
        this.file = file;
        System.out.println("Loading " + file + " from disk...");   // expensive
    }

    public void display() { System.out.println("Displaying " + file); }
}

class LazyImageProxy implements Image {
    private final String file;
    private HighResImage real;   // created on first use

    LazyImageProxy(String file) { this.file = file; }

    public void display() {
        if (real == null) real = new HighResImage(file);
        real.display();
    }
}

public class StaticProxyDemo {
    public static void main(String[] args) {
        Image img = new LazyImageProxy("photo.png");   // nothing loaded yet
        img.display();   // loads, then displays
        img.display();   // displays only
    }
}
```

### Example 2: JDK dynamic proxy (interface-based)

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

interface Greeter {
    String greet(String name);
}

public class DynamicProxyDemo {
    public static void main(String[] args) {
        Greeter target = name -> "Hello, " + name;

        InvocationHandler timing = (proxy, method, methodArgs) -> {
            long start = System.nanoTime();
            try {
                return method.invoke(target, methodArgs);
            } finally {
                System.out.println(method.getName() + " took " + (System.nanoTime() - start) + " ns");
            }
        };

        Greeter proxy = (Greeter) Proxy.newProxyInstance(
            Greeter.class.getClassLoader(), new Class<?>[] { Greeter.class }, timing);

        System.out.println(proxy.greet("Ann"));
    }
}
```

The handler is written once and applies to every method of the interface. Note that `method.invoke` wraps exceptions thrown by the target in `InvocationTargetException`, so production handlers usually unwrap it.

### Real-world Java/Spring usage
- **Spring AOP** (`@Transactional`, `@Cacheable`, `@Async`, `@Secured`) wraps your bean in a proxy. It uses a **JDK dynamic proxy** if the bean implements an interface and `proxyTargetClass` is false, otherwise a **CGLIB** subclass proxy. Spring Boot defaults to CGLIB (`spring.aop.proxy-target-class=true`).
- **Hibernate/JPA** lazy associations and `getReference()` return proxies that load on first access.
- Spring Data repository interfaces are implemented at runtime by JDK proxies.
- `Collections.unmodifiableList` and `Collections.synchronizedList` behave like protection proxies (they are also commonly cited as decorators).
- Java RMI stubs and OpenFeign clients are remote proxies; Mockito mocks are generated proxies.

### Trade-offs
- Adds cross-cutting behavior transparently; target stays focused on business logic.
- Extra indirection and some runtime cost; stack traces get longer.
- **Self-invocation gotcha**: calling `this.otherMethod()` inside a Spring bean bypasses the proxy, so `@Transactional` or `@Cacheable` on `otherMethod` does not apply.
- JDK proxies need an interface; CGLIB cannot proxy `final` classes or `final` methods.

### Interview tip
Be ready to explain how `@Transactional` works: Spring creates a proxy that opens a transaction, delegates to the target, then commits or rolls back. Mention the self-invocation problem and JDK vs CGLIB. Contrast with Decorator: a decorator is chosen by the client to *add* behavior, a proxy usually decides *access* to the subject and often creates or locates it itself.

---

## Common interview questions

- **How do Adapter, Decorator, Proxy and Facade differ?** Adapter changes the interface; Decorator keeps it and adds behavior; Proxy keeps it and controls access; Facade defines a new, simpler interface over several classes.
- **Which structural patterns does `java.io` use?** Decorator (`BufferedInputStream`, `GZIPInputStream`, `DataInputStream`) and Adapter (`InputStreamReader`, `OutputStreamWriter`).
- **How does Spring AOP implement `@Transactional`?** Through a runtime proxy (JDK dynamic proxy or CGLIB) that wraps the bean; self-invocation bypasses it.
- **JDK dynamic proxy vs CGLIB?** JDK proxies implement interfaces via `java.lang.reflect.Proxy`; CGLIB generates a subclass, so the class and methods cannot be `final`.
- **Why does `Integer.valueOf(127) == Integer.valueOf(127)` but not for 128?** Flyweight cache for -128 to 127 (upper bound configurable with `-XX:AutoBoxCacheMax`).
- **Bridge vs Adapter?** Bridge is planned up front to let abstraction and implementation vary; Adapter is a retrofit to make incompatible existing classes cooperate.
- **Flyweight vs Object Pool?** Flyweight shares immutable state among many users at once; a pool lends out mutable objects exclusively.
- **Where is Composite used?** UI component trees, file systems, DOM, expression trees; one interface for leaf and group.

Practice more in the interview question banks:

- [Design patterns questions](../interview-questions/verbal/verbal-design-patterns-questions.md)
- [Design principles and patterns questions](../interview-questions/verbal/verbal-design-patterns-principles-questions.md)
