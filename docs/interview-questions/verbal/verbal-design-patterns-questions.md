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

## Additional Design Patterns

See [Design Patterns Reference](../../java-patterns-advanced/design-patterns.md) for:
- **Creational**: Prototype, Abstract Factory, Builder, Factory Method, Singleton
- **Structural**: Adapter, Bridge, Composite, Facade, Flyweight, Proxy
- **Behavioral**: Chain of Responsibility, Command, Iterator, Mediator, Memento, State, Strategy, Template Method, Visitor, Interpreter

---

**See also:** [Gang of Four Design Patterns](../../java-patterns-advanced/design-patterns.md), [Interview Questions Overview](../index.md)
