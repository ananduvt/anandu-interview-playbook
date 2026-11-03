# Design Patterns Interview Questions

Creational, Structural, and Behavioral design patterns - Gang of Four patterns and their real-world applications.

---

## 1. What is the Factory Design Pattern and when should you use it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Factory Pattern:</strong> Creates objects without specifying their exact classes. Returns one of several subclasses based on input parameters.</p>

<h3>When to Use</h3>

<ul>
<li><strong>Object type unknown at compile time:</strong> Decide at runtime which class to instantiate</li>
<li><strong>Decouple client from concrete classes:</strong> Client doesn't know about implementation</li>
<li><strong>Complex object creation logic:</strong> Multiple steps or conditional logic in construction</li>
<li><strong>Adding new types frequently:</strong> Easy to extend without modifying client code</li>
</ul>

<h3>Example</h3>

```java
// Product Interface
public interface Notification {
    void send(String to, String message);
}

// Concrete Products
public class EmailNotification implements Notification {
    @Override
    public void send(String to, String message) {
        System.out.println("Sending EMAIL to " + to + ": " + message);
    }
}

public class SmsNotification implements Notification {
    @Override
    public void send(String to, String message) {
        System.out.println("Sending SMS to " + to + ": " + message);
    }
}

// Factory
public class NotificationFactory {
    public static Notification create(String channel) {
        if (channel == null) throw new IllegalArgumentException("Channel required");
        
        switch (channel.toLowerCase()) {
            case "email": return new EmailNotification();
            case "sms":   return new SmsNotification();
            case "push":  return new PushNotification();
            default: throw new IllegalArgumentException("Unknown channel");
        }
    }
}

// Usage
Notification notification = NotificationFactory.create("email");
notification.send("user@example.com", "Welcome!");
```

<h3>Advantages vs Disadvantages</h3>

<table>
<tr><th>Advantages</th><th>Disadvantages</th></tr>
<tr><td>✅ Decouples client from concrete classes</td><td>❌ Added complexity for simple cases</td></tr>
<tr><td>✅ Easy to add new product types</td><td>❌ Factory class grows with each new type</td></tr>
<tr><td>✅ Centralizes object creation logic</td><td>❌ May violate Open/Closed principle if not careful</td></tr>
<tr><td>✅ Encapsulates instantiation</td><td></td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 2. What's the difference between Factory and Abstract Factory patterns?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr><th>Aspect</th><th>Factory Pattern</th><th>Abstract Factory</th></tr>
<tr><td><strong>Creates</strong></td><td>One product type</td><td>Families of related products</td></tr>
<tr><td><strong>Returns</strong></td><td>Single interface</td><td>Multiple interfaces</td></tr>
<tr><td><strong>Use Case</strong></td><td>Simple object creation</td><td>Coordinated product families</td></tr>
<tr><td><strong>Complexity</strong></td><td>Low</td><td>High</td></tr>
<tr><td><strong>Example</strong></td><td>Notification factory (email/SMS)</td><td>UI factory (Windows/Mac buttons, menus)</td></tr>
</table>

<h3>Abstract Factory Example</h3>

```java
// Abstract products
public interface Button { void click(); }
public interface Checkbox { void toggle(); }

// Concrete products for Windows
public class WindowsButton implements Button {
    @Override
    public void click() { System.out.println("Windows Button clicked"); }
}

public class WindowsCheckbox implements Checkbox {
    @Override
    public void toggle() { System.out.println("Windows Checkbox toggled"); }
}

// Concrete products for Mac
public class MacButton implements Button {
    @Override
    public void click() { System.out.println("Mac Button clicked"); }
}

public class MacCheckbox implements Checkbox {
    @Override
    public void toggle() { System.out.println("Mac Checkbox toggled"); }
}

// Abstract Factory
public interface UIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factories
public class WindowsUIFactory implements UIFactory {
    @Override
    public Button createButton() { return new WindowsButton(); }
    
    @Override
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

public class MacUIFactory implements UIFactory {
    @Override
    public Button createButton() { return new MacButton(); }
    
    @Override
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Usage
UIFactory factory = osType.equals("windows") ? new WindowsUIFactory() : new MacUIFactory();
Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();
```

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 3. Explain the Builder Design Pattern with a real example

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Builder Pattern:</strong> Constructs complex objects step-by-step, separating construction from representation. Useful for objects with many optional parameters.</p>

<h3>When to Use</h3>

<ul>
<li><strong>Many optional parameters:</strong> Avoid constructor explosion</li>
<li><strong>Readable object creation:</strong> Fluent, readable syntax</li>
<li><strong>Immutable objects:</strong> Build then set final state</li>
<li><strong>Complex initialization:</strong> Multi-step setup process</li>
</ul>

<h3>Real-World Example: HTTP Request Builder</h3>

```java
public class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;
    private final int timeout;
    private final boolean followRedirects;
    
    // Private constructor - only builder can create
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
        this.body = builder.body;
        this.timeout = builder.timeout;
        this.followRedirects = builder.followRedirects;
    }
    
    // Builder class
    public static class Builder {
        private final String url;  // Required
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private String body = "";
        private int timeout = 30000;
        private boolean followRedirects = true;
        
        public Builder(String url) {
            this.url = url;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public Builder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }
        
        public Builder followRedirects(boolean follow) {
            this.followRedirects = follow;
            return this;
        }
        
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// Usage: Clean, readable, type-safe
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
    .method("POST")
    .header("Authorization", "Bearer token123")
    .header("Content-Type", "application/json")
    .body("{\"name\": \"John\"}")
    .timeout(5000)
    .followRedirects(false)
    .build();
```

<h3>Benefits</h3>

<ul>
<li>✅ Fluent, readable syntax</li>
<li>✅ Handles optional parameters cleanly</li>
<li>✅ Immutability (all fields final)</li>
<li>✅ Single Responsibility (Builder separate from HttpRequest)</li>
<li>✅ Validation can be done in build() method</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 4. What is the Singleton Pattern and when is it problematic?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Singleton:</strong> Restricts a class to have only one instance, providing global access to that instance.</p>

<h3>Thread-Safe Implementation (Bill Pugh)</h3>

```java
public class DatabaseConnection {
    
    // Private constructor - prevents instantiation
    private DatabaseConnection() {
    }
    
    // Static inner class - loaded only when needed
    private static class SingletonHelper {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }
    
    // Public accessor
    public static DatabaseConnection getInstance() {
        return SingletonHelper.INSTANCE;
    }
    
    public void connect() {
        System.out.println("Database connected");
    }
}

// Usage
DatabaseConnection db = DatabaseConnection.getInstance();
db.connect();
```

<h3>When Problematic ⚠️</h3>

<table>
<tr><th>Problem</th><th>Impact</th><th>Solution</th></tr>
<tr><td>Hard to test</td><td>Cannot mock or inject; couples tests to singleton</td><td>Use dependency injection instead</td></tr>
<tr><td>Hidden dependencies</td><td>Global state makes code hard to understand</td><td>Make dependencies explicit in constructor</td></tr>
<tr><td>Multi-threading issues</td><td>Race conditions if not implemented carefully</td><td>Use Bill Pugh or enum approach</td></tr>
<tr><td>Concurrency scaling</td><td>Single instance bottleneck in distributed systems</td><td>Use connection pool or service registry</td></tr>
</table>

<h3>Better Alternative: Dependency Injection</h3>

```java
// ✅ PREFERRED: Inject dependency
@Service
public class OrderService {
    private final DatabaseConnection db;
    
    public OrderService(DatabaseConnection db) {
        this.db = db;  // Injected - testable, mockable
    }
    
    public void createOrder(Order order) {
        db.connect();
        // Process order
    }
}

// In tests: Easy to mock
@Test
public void testCreateOrder() {
    DatabaseConnection mockDb = mock(DatabaseConnection.class);
    OrderService service = new OrderService(mockDb);
    // Test without touching real DB
}
```

<h3>When Singleton is Appropriate</h3>

<ul>
<li>✅ Logger factories (Log4j, SLF4J)</li>
<li>✅ Thread pools and executors</li>
<li>✅ Database connection pools</li>
<li>✅ Cache managers</li>
<li>❌ Service objects (use DI instead)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 5. Explain the Strategy Pattern and provide a use case

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Strategy Pattern:</strong> Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Lets clients choose the algorithm at runtime.</p>

<h3>Real-World Example: Payment Processing</h3>

```java
// Strategy interface
public interface PaymentStrategy {
    boolean pay(double amount);
    String getPaymentMethod();
}

// Concrete strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "Credit Card";
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "PayPal";
    }
}

public class CryptoCurrencyPayment implements PaymentStrategy {
    private String walletAddress;
    
    public CryptoCurrencyPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public boolean pay(double amount) {
        System.out.println("Processing cryptocurrency payment: $" + amount);
        return true;
    }
    
    @Override
    public String getPaymentMethod() {
        return "Cryptocurrency";
    }
}

// Context: Uses strategy
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public void setPaymentMethod(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout() {
        double total = items.stream().mapToDouble(Item::getPrice).sum();
        
        if (paymentStrategy.pay(total)) {
            System.out.println("✅ Payment successful via " + paymentStrategy.getPaymentMethod());
        } else {
            System.out.println("❌ Payment failed");
        }
    }
}

// Usage: Choose strategy at runtime
ShoppingCart cart = new ShoppingCart();
cart.addItem(new Item("Book", 25.99));
cart.addItem(new Item("Pen", 5.50));

// Customer chooses payment method
PaymentStrategy paymentMethod = new CreditCardPayment("1234-5678-9012-3456");
cart.setPaymentMethod(paymentMethod);
cart.checkout();

// Later, customer switches method
paymentMethod = new PayPalPayment("customer@paypal.com");
cart.setPaymentMethod(paymentMethod);
cart.checkout();
```

<h3>Benefits</h3>

<ul>
<li>✅ Runtime algorithm selection</li>
<li>✅ Encapsulates behavior variations</li>
<li>✅ Easy to add new strategies</li>
<li>✅ Eliminates if/else chains</li>
<li>✅ Open/Closed Principle compliant</li>
</ul>

<h3>Real-World Uses</h3>

<ul>
<li>Sorting algorithms (Collections.sort with Comparator)</li>
<li>Payment processing (credit card, PayPal, crypto)</li>
<li>Compression strategies (ZIP, GZIP, BZIP2)</li>
<li>Export formats (PDF, Excel, CSV)</li>
<li>Authentication methods (JWT, OAuth2, Basic Auth)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 6. What is the Observer Pattern and when would you use it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Observer Pattern:</strong> Defines one-to-many relationship where many observer objects watch a subject object. When subject changes state, all observers are notified automatically.</p>

<h3>Real-World Example: Event Publishing System</h3>

```java
// Subject
public class OrderService {
    private List<OrderObserver> observers = new ArrayList<>();
    
    public void subscribe(OrderObserver observer) {
        observers.add(observer);
    }
    
    public void unsubscribe(OrderObserver observer) {
        observers.remove(observer);
    }
    
    public void createOrder(Order order) {
        // Save order to database
        System.out.println("✅ Order created: " + order.getId());
        
        // Notify all observers
        notifyOrderCreated(order);
    }
    
    private void notifyOrderCreated(Order order) {
        for (OrderObserver observer : observers) {
            observer.onOrderCreated(order);
        }
    }
}

// Observer interface
public interface OrderObserver {
    void onOrderCreated(Order order);
}

// Concrete observers
public class EmailNotificationObserver implements OrderObserver {
    @Override
    public void onOrderCreated(Order order) {
        System.out.println("📧 Sending confirmation email to customer");
    }
}

public class InventoryObserver implements OrderObserver {
    @Override
    public void onOrderCreated(Order order) {
        System.out.println("📦 Updating inventory for items in order");
    }
}

public class PaymentObserver implements OrderObserver {
    @Override
    public void onOrderCreated(Order order) {
        System.out.println("💳 Processing payment for order");
    }
}

public class AnalyticsObserver implements OrderObserver {
    @Override
    public void onOrderCreated(Order order) {
        System.out.println("📊 Logging order event for analytics");
    }
}

// Usage
OrderService orderService = new OrderService();

// Subscribe observers
orderService.subscribe(new EmailNotificationObserver());
orderService.subscribe(new InventoryObserver());
orderService.subscribe(new PaymentObserver());
orderService.subscribe(new AnalyticsObserver());

// Create order - all observers notified automatically
Order order = new Order("ORD-123");
orderService.createOrder(order);
// Output:
// ✅ Order created: ORD-123
// 📧 Sending confirmation email to customer
// 📦 Updating inventory for items in order
// 💳 Processing payment for order
// 📊 Logging order event for analytics
```

<h3>Java's Built-in Observer Mechanism</h3>

```java
// Java provides PropertyChangeListener for Observer pattern
public class Order extends Observable {
    private String status;
    
    public void setStatus(String newStatus) {
        String oldStatus = this.status;
        this.status = newStatus;
        
        // Notify observers
        setChanged();
        notifyObservers(new StatusChangeEvent(oldStatus, newStatus));
    }
}

// Observer
public class OrderStatusListener implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        StatusChangeEvent event = (StatusChangeEvent) arg;
        System.out.println("Order status changed from " + event.oldStatus + " to " + event.newStatus);
    }
}
```

<h3>Observer vs Pub/Sub</h3>

<table>
<tr><th>Aspect</th><th>Observer</th><th>Pub/Sub (Event-Driven)</th></tr>
<tr><td>Coupling</td><td>Tightly coupled (direct references)</td><td>Loosely coupled (via broker)</td></tr>
<tr><td>Communication</td><td>Direct method calls</td><td>Through message broker</td></tr>
<tr><td>Scale</td><td>Good for small apps</td><td>Better for microservices</td></tr>
<tr><td>Performance</td><td>Faster (in-process)</td><td>Higher latency (network)</td></tr>
<tr><td>Example</td><td>UI listeners, event handlers</td><td>Kafka, RabbitMQ, SNS/SQS</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 7. Explain the Decorator Pattern with an example

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>

<p><strong>Decorator Pattern:</strong> Attaches responsibilities to objects dynamically. Provides a flexible alternative to subclassing for extending functionality.</p>

<h3>Real-World Example: Coffee Ordering System</h3>

```java
// Component interface
public interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 2.00;
    }
    
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
}

// Abstract decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.10;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
}

public class CaramelDecorator extends CoffeeDecorator {
    public CaramelDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.75;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Caramel";
    }
}

// Usage: Stack decorators dynamically
Coffee coffee = new SimpleCoffee();  // $2.00

coffee = new MilkDecorator(coffee);      // $2.50
coffee = new SugarDecorator(coffee);     // $2.60
coffee = new CaramelDecorator(coffee);   // $3.35

System.out.println(coffee.getDescription());  // Simple Coffee, Milk, Sugar, Caramel
System.out.println("Cost: $" + coffee.getCost());  // Cost: $3.35
```

<h3>Decorator vs Inheritance</h3>

<table>
<tr><th>Aspect</th><th>Decorator</th><th>Inheritance</th></tr>
<tr><td>Flexibility</td><td>✅ Runtime composition</td><td>❌ Compile-time</td></tr>
<tr><td>Combination</td><td>✅ Any combinations</td><td>❌ Limited hierarchy</td></tr>
<tr><td>Complexity</td><td>❌ More objects</td><td>✅ Simpler</td></tr>
<tr><td>Maintenance</td><td>✅ Open/Closed</td><td>❌ Explosion of classes</td></tr>
<tr><td>Example</td><td>MilkDecorator(SugarDecorator(coffee))</td><td>MilkSugarCoffee extends Coffee</td></tr>
</table>

<h3>Real-World Use Cases</h3>

<ul>
<li>Java I/O Streams (BufferedInputStream, DataInputStream)</li>
<li>Servlet filters (request/response decoration)</li>
<li>UI component styling (add borders, colors, shadows)</li>
<li>Logging decorators (add logging to any object)</li>
<li>Security decorators (add authentication/authorization)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 8. What is the Prototype Design Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Prototype</strong> creates new objects by copying an existing instance (the prototype) instead of building one from scratch. The client asks the object to clone itself, so it does not need to know the concrete class.</p>

<h3>Why / Context</h3>
<ul>
<li>Useful when construction is expensive (parsing, DB/network load) and copies differ only slightly.</li>
<li>Avoids a parallel hierarchy of factories; a registry of ready-made prototypes can be cloned on demand.</li>
</ul>

<h3>Java Specifics and Gotchas</h3>
<ul>
<li><code>Object.clone()</code> with <code>Cloneable</code> is widely considered broken: it is a shallow copy, bypasses constructors, and <code>Cloneable</code> has no <code>clone()</code> method.</li>
<li><strong>Shallow vs deep copy:</strong> shallow copy shares mutable fields (lists, arrays) between original and clone, so a change in one shows up in the other.</li>
<li>Prefer a <strong>copy constructor</strong> or copy factory method; records and immutable objects usually need no cloning.</li>
</ul>

<h3>Example</h3>

```java
public class Report {
    private final String title;
    private final List<String> sections;

    public Report(String title, List<String> sections) {
        this.title = title;
        this.sections = new ArrayList<>(sections);
    }

    // Copy constructor = deep copy of the mutable list
    public Report(Report other) {
        this(other.title, other.sections);
    }
}

Report template = loadExpensiveTemplate();
Report copy = new Report(template);   // cheap, independent copy
```

<h3>When to Use</h3>
<p>Costly initialization with many near-identical objects, or when you need a snapshot of an object. Skip it when objects are cheap to create.</p>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 9. Explain the Adapter Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Adapter</strong> converts the interface of an existing class into the interface the client expects, so classes with incompatible interfaces can work together without changing either one.</p>

<h3>Why / Context</h3>
<ul>
<li>Typical for legacy code, third-party libraries, or swapping vendors (for example payment gateways) behind your own interface.</li>
<li><strong>Object adapter</strong> (composition, preferred) wraps the adaptee; <strong>class adapter</strong> (inheritance) extends it and needs multiple inheritance, so it is rare in Java.</li>
</ul>

<h3>Example</h3>

```java
interface PaymentProcessor {
    void pay(long amountCents);
}

// Third-party class we cannot change
class LegacyGateway {
    void makePayment(double dollars) { /* ... */ }
}

class LegacyGatewayAdapter implements PaymentProcessor {
    private final LegacyGateway gateway;
    LegacyGatewayAdapter(LegacyGateway gateway) { this.gateway = gateway; }

    @Override
    public void pay(long amountCents) {
        gateway.makePayment(amountCents / 100.0);
    }
}
```

<h3>JDK Examples</h3>
<ul>
<li><code>Arrays.asList()</code> (array to List), <code>InputStreamReader</code> (byte stream to character stream), <code>Collections.enumeration()</code>.</li>
</ul>

<h3>Comparison</h3>
<table>
<tr><th>Pattern</th><th>Intent</th></tr>
<tr><td>Adapter</td><td>Change an interface to match what the client wants</td></tr>
<tr><td>Decorator</td><td>Same interface, adds behavior</td></tr>
<tr><td>Facade</td><td>New simpler interface over many classes</td></tr>
<tr><td>Proxy</td><td>Same interface, controls access</td></tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 10. What is the Facade Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Facade</strong> provides one simple, high-level interface over a complex subsystem of several classes. Clients call the facade instead of coordinating many collaborators.</p>

<h3>Why / Context</h3>
<ul>
<li>Reduces coupling between clients and the subsystem and gives a single place for the orchestration logic.</li>
<li>It does not hide the subsystem: advanced users can still use the underlying classes directly.</li>
<li>A Spring <code>@Service</code> that coordinates several repositories and clients is often a facade in practice.</li>
</ul>

<h3>Example</h3>

```java
class OrderFacade {
    private final InventoryService inventory;
    private final PaymentService payment;
    private final ShippingService shipping;

    OrderFacade(InventoryService i, PaymentService p, ShippingService s) {
        this.inventory = i; this.payment = p; this.shipping = s;
    }

    public void placeOrder(Order order) {
        inventory.reserve(order.items());
        payment.charge(order.customerId(), order.total());
        shipping.schedule(order);
    }
}
```

<h3>Gotchas</h3>
<ul>
<li>Avoid a "god facade" that grows into a catch-all class; keep one facade per use-case area.</li>
<li>Facade vs Adapter: a facade simplifies many classes; an adapter makes one incompatible interface fit.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 11. Explain the Bridge Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Bridge</strong> splits a class into two independent hierarchies, an <em>abstraction</em> and an <em>implementation</em>, connected by composition. Each hierarchy can vary on its own.</p>

<h3>Why / Context</h3>
<ul>
<li>Prevents a class explosion: with M abstractions and N implementations, inheritance needs M x N classes, Bridge needs M + N.</li>
<li>The abstraction holds a reference to an implementor interface and delegates to it. The choice can even change at runtime.</li>
</ul>

<h3>Example</h3>

```java
interface MessageSender {            // implementation side
    void deliver(String text);
}
class EmailSender implements MessageSender { public void deliver(String t) { /* smtp */ } }
class SmsSender   implements MessageSender { public void deliver(String t) { /* sms  */ } }

abstract class Notification {        // abstraction side
    protected final MessageSender sender;
    protected Notification(MessageSender sender) { this.sender = sender; }
    abstract void notifyUser(String msg);
}
class UrgentNotification extends Notification {
    UrgentNotification(MessageSender s) { super(s); }
    void notifyUser(String msg) { sender.deliver("[URGENT] " + msg); }
}
```

<h3>Bridge vs Adapter vs Strategy</h3>
<ul>
<li><strong>Adapter</strong> is applied after the fact to make things fit; <strong>Bridge</strong> is designed up front.</li>
<li><strong>Strategy</strong> has the same structure but a narrower goal (swap one algorithm); Bridge separates two whole dimensions of a class. JDBC (<code>DriverManager</code> + drivers) is a classic Bridge.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 12. What is the Composite Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Composite</strong> lets you treat individual objects (leaves) and groups of objects (composites) through the same interface, which is natural for tree structures such as file systems, UI components, or org charts.</p>

<h3>Structure</h3>
<ul>
<li><strong>Component:</strong> common interface.</li>
<li><strong>Leaf:</strong> does the real work, has no children.</li>
<li><strong>Composite:</strong> holds children (as Components) and delegates to them, often aggregating results.</li>
</ul>

<h3>Example</h3>

```java
interface FileNode {
    long size();
}
record FileLeaf(String name, long bytes) implements FileNode {
    public long size() { return bytes; }
}
class Directory implements FileNode {
    private final List<FileNode> children = new ArrayList<>();
    void add(FileNode node) { children.add(node); }
    public long size() {
        return children.stream().mapToLong(FileNode::size).sum();   // recursion through the tree
    }
}
```

<h3>Gotchas</h3>
<ul>
<li>Where to declare <code>add/remove</code>: on the Component (transparent, but leaves must throw) or only on Composite (safe, but clients need to know the type).</li>
<li>Deep trees can cause stack overflow with recursion; watch for cycles.</li>
<li>JDK/Spring examples: Swing <code>Container</code>/<code>Component</code>, JSON tree models.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 13. Explain the Chain of Responsibility Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Chain of Responsibility</strong> passes a request along a chain of handlers. Each handler either processes it, passes it on, or both. The sender does not know which handler will act.</p>

<h3>Why / Context</h3>
<ul>
<li>Decouples sender from receivers and lets you add, remove, or reorder handlers by configuration.</li>
<li>Variants: first-handler-wins (exception handlers) or all-handlers-run (servlet filters, middleware).</li>
</ul>

<h3>Example</h3>

```java
abstract class Validator {
    private Validator next;
    Validator linkWith(Validator next) { this.next = next; return next; }

    final void validate(Request r) {
        check(r);                          // throws on failure
        if (next != null) next.validate(r);
    }
    protected abstract void check(Request r);
}

Validator head = new AuthValidator();
head.linkWith(new RateLimitValidator()).linkWith(new PayloadValidator());
head.validate(request);
```

<h3>Real-World Use</h3>
<ul>
<li>Servlet <code>Filter</code> chain, Spring Security filter chain, Spring MVC <code>HandlerInterceptor</code>, logging frameworks' handler levels.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>A request might fall off the end unhandled; define a default.</li>
<li>Long chains are hard to debug; keep handler order explicit and logged.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 14. What is the Command Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Command</strong> turns a request into an object holding the action and its parameters. That lets you queue, log, schedule, undo, or pass requests around like data.</p>

<h3>Structure</h3>
<ul>
<li><strong>Command</strong> interface (<code>execute()</code>, optionally <code>undo()</code>), <strong>Receiver</strong> (does the work), <strong>Invoker</strong> (triggers commands), <strong>Client</strong> (wires them).</li>
</ul>

<h3>Example</h3>

```java
interface Command {
    void execute();
    void undo();
}

class AddTextCommand implements Command {
    private final StringBuilder doc; private final String text;
    AddTextCommand(StringBuilder doc, String text) { this.doc = doc; this.text = text; }
    public void execute() { doc.append(text); }
    public void undo()    { doc.setLength(doc.length() - text.length()); }
}

class Editor {
    private final Deque<Command> history = new ArrayDeque<>();
    void run(Command c) { c.execute(); history.push(c); }
    void undoLast()     { if (!history.isEmpty()) history.pop().undo(); }
}
```

<h3>Real-World Use</h3>
<ul>
<li><code>Runnable</code>/<code>Callable</code> submitted to an <code>ExecutorService</code>, job queues, undo/redo, transactional scripts, event sourcing (commands and events).</li>
</ul>

<h3>Command vs Strategy</h3>
<p>Command encapsulates <em>a request</em> (what to do, with its data, often once); Strategy encapsulates <em>an interchangeable algorithm</em> chosen for a context.</p>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 15. Explain the Iterator Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Iterator</strong> gives sequential access to the elements of a collection without exposing its internal structure. In Java this is <code>java.util.Iterator</code> and the <code>Iterable</code> interface behind the for-each loop.</p>

<h3>Example</h3>

```java
class Range implements Iterable<Integer> {
    private final int start, end;
    Range(int start, int end) { this.start = start; this.end = end; }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            private int current = start;
            public boolean hasNext() { return current < end; }
            public Integer next() {
                if (!hasNext()) throw new NoSuchElementException();
                return current++;
            }
        };
    }
}

for (int n : new Range(0, 3)) System.out.println(n);   // 0 1 2
```

<h3>Gotchas and Follow-ups</h3>
<ul>
<li><strong>Fail-fast:</strong> most <code>java.util</code> iterators throw <code>ConcurrentModificationException</code> if the collection is structurally modified during iteration (except through <code>iterator.remove()</code>). Concurrent collections use weakly consistent iterators.</li>
<li><code>Iterator</code> is single-pass and one-direction; <code>ListIterator</code> adds backward movement and <code>set/add</code>.</li>
<li><strong>Internal vs external iteration:</strong> the Streams API is internal iteration, where the library drives the loop.</li>
<li>Each call to <code>iterator()</code> should return an independent iterator.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 16. What is the Mediator Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Mediator</strong> centralizes communication between objects so they no longer reference each other directly. Colleagues talk only to the mediator, which turns many-to-many dependencies into many-to-one.</p>

<h3>Example</h3>

```java
interface ChatRoom {
    void send(String from, String message);
}

class SimpleChatRoom implements ChatRoom {
    private final List<User> users = new ArrayList<>();
    void join(User u) { users.add(u); }

    public void send(String from, String message) {
        for (User u : users) {
            if (!u.name().equals(from)) u.receive(from, message);
        }
    }
}

record User(String name, ChatRoom room) {
    void say(String m) { room.send(name, m); }
    void receive(String from, String m) { System.out.println(name + " got " + m + " from " + from); }
}
```

<h3>Mediator vs Observer</h3>
<table>
<tr><th>Mediator</th><th>Observer</th></tr>
<tr><td>Central object coordinates peers</td><td>Subject broadcasts to subscribers</td></tr>
<tr><td>Knows the colleagues and their logic</td><td>Subject does not know what observers do</td></tr>
<tr><td>Two-way interaction</td><td>Typically one-way notification</td></tr>
</table>

<h3>Gotchas</h3>
<ul>
<li>The mediator can become a god object that holds all the logic.</li>
<li>Examples: chat rooms, air-traffic-control analogies, UI dialog controllers, event-bus style components.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 17. Explain the Memento Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Memento</strong> captures an object's internal state in a snapshot so it can be restored later (undo, rollback), without breaking encapsulation.</p>

<h3>Roles</h3>
<ul>
<li><strong>Originator:</strong> creates a memento of its state and restores from one.</li>
<li><strong>Memento:</strong> immutable snapshot, opaque to everyone but the originator.</li>
<li><strong>Caretaker:</strong> stores mementos (for example in a stack) but never inspects them.</li>
</ul>

<h3>Example</h3>

```java
class TextEditor {
    private String content = "";

    record Memento(String content) {}          // immutable snapshot

    void type(String s)      { content += s; }
    Memento save()           { return new Memento(content); }
    void restore(Memento m)  { content = m.content(); }
    String content()         { return content; }
}

TextEditor ed = new TextEditor();
Deque<TextEditor.Memento> history = new ArrayDeque<>();   // caretaker
ed.type("Hello"); history.push(ed.save());
ed.type(" World");
ed.restore(history.pop());                                // back to "Hello"
```

<h3>Gotchas</h3>
<ul>
<li>Memory cost grows with snapshot size and count; bound the history or store diffs instead.</li>
<li>Snapshot mutable fields by deep copy, or the "snapshot" changes along with the object.</li>
<li>Memento vs Command undo: Memento restores saved state; Command undo reverses the operation. They are often combined.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 18. What is the State Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>State</strong> lets an object change its behavior when its internal state changes, by delegating to a separate state object per state. It replaces large <code>if/switch</code> blocks on a status field.</p>

<h3>Example</h3>

```java
interface OrderState {
    OrderState pay();
    OrderState ship();
}

class New implements OrderState {
    public OrderState pay()  { return new Paid(); }
    public OrderState ship() { throw new IllegalStateException("Pay first"); }
}
class Paid implements OrderState {
    public OrderState pay()  { throw new IllegalStateException("Already paid"); }
    public OrderState ship() { return new Shipped(); }
}
class Shipped implements OrderState {
    public OrderState pay()  { throw new IllegalStateException("Already paid"); }
    public OrderState ship() { throw new IllegalStateException("Already shipped"); }
}

class Order {
    private OrderState state = new New();
    void pay()  { state = state.pay(); }
    void ship() { state = state.ship(); }
}
```

<h3>State vs Strategy</h3>
<ul>
<li>Same structure, different intent: in State the object <em>transitions itself</em> between states; in Strategy the client picks the algorithm and it does not change on its own.</li>
</ul>

<h3>When to Use / Alternatives</h3>
<ul>
<li>Use when behavior depends on state and there are many states and transitions. Many states mean many classes.</li>
<li>For simple cases an <code>enum</code> with per-constant behavior is enough; for complex workflows consider Spring Statemachine.</li>
<li>Stateless state objects can be shared as singletons.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 19. Explain the Template Method Pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Template Method</strong> defines the skeleton of an algorithm in a base class and lets subclasses override specific steps without changing the overall structure. It uses inheritance.</p>

<h3>Example</h3>

```java
abstract class DataExporter {
    // The template method: final so subclasses cannot change the order
    public final void export() {
        List<String> rows = fetch();
        String body = format(rows);
        write(body);
        afterExport();                 // optional hook
    }

    protected abstract List<String> fetch();
    protected abstract String format(List<String> rows);
    protected void write(String body) { System.out.println(body); }
    protected void afterExport() { }   // hook with default no-op
}

class CsvExporter extends DataExporter {
    protected List<String> fetch() { return List.of("a", "b"); }
    protected String format(List<String> rows) { return String.join(",", rows); }
}
```

<h3>Gotchas and Follow-ups</h3>
<ul>
<li>Mark the template method <code>final</code>; distinguish <em>abstract steps</em> (must override) from <em>hooks</em> (may override).</li>
<li>Inheritance-based, so it is rigid. If you need runtime flexibility, use Strategy (composition) instead.</li>
<li>Hollywood principle: "don't call us, we'll call you".</li>
<li>Seen in <code>JdbcTemplate</code>/<code>RestTemplate</code> callbacks (callback variant), <code>AbstractList</code>, <code>HttpServlet.service()</code> calling <code>doGet/doPost</code>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 20. What is the Proxy Pattern?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Proxy</strong> is a stand-in object with the same interface as the real object that controls access to it, adding behavior such as lazy loading, caching, security checks, or logging before or after delegating.</p>

<h3>Types</h3>
<table>
<tr><th>Type</th><th>Purpose</th></tr>
<tr><td>Virtual</td><td>Lazy-create an expensive object (Hibernate lazy loading)</td></tr>
<tr><td>Protection</td><td>Check permissions before delegating</td></tr>
<tr><td>Remote</td><td>Local stand-in for a remote service (RMI stub, Feign client)</td></tr>
<tr><td>Caching / logging</td><td>Add cross-cutting behavior</td></tr>
</table>

<h3>Example: JDK Dynamic Proxy</h3>

```java
interface Service { String load(String id); }

Service real = id -> "data-" + id;

Service proxy = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class<?>[] { Service.class },
    (p, method, args) -> {
        long start = System.nanoTime();
        try {
            return method.invoke(real, args);
        } finally {
            System.out.println(method.getName() + " took " + (System.nanoTime() - start) + " ns");
        }
    });
```

<h3>Spring Gotchas</h3>
<ul>
<li>Spring AOP (<code>@Transactional</code>, <code>@Cacheable</code>, <code>@Async</code>) works through proxies: JDK proxies for interfaces, CGLIB subclasses otherwise.</li>
<li><strong>Self-invocation</strong> (calling a proxied method via <code>this</code>) bypasses the proxy, so the annotation is silently ignored.</li>
<li>CGLIB cannot proxy <code>final</code> classes or methods.</li>
<li>Proxy vs Decorator: structure is the same; Proxy controls access to its object, Decorator adds responsibilities.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## Additional Design Patterns

See [Design Patterns Reference](../../java-patterns-advanced/design-patterns.md) for:
- **Creational**: Prototype, Abstract Factory, Builder, Factory Method, Singleton
- **Structural**: Adapter, Bridge, Composite, Facade, Flyweight, Proxy
- **Behavioral**: Chain of Responsibility, Command, Iterator, Mediator, Memento, State, Strategy, Template Method, Visitor, Interpreter

---

**See also:** [Gang of Four Design Patterns](../../java-patterns-advanced/design-patterns.md), [Interview Preparation](../index.md)
