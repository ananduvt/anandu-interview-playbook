# Spring Annotations & Components

Understanding Spring stereotypes, bean registration, component scanning, and how annotations work under the hood.

---

## Spring Stereotypes (Annotations)

Stereotypes are annotations that mark a class for automatic detection and bean registration during classpath scanning.

### @Component (Generic)

Base annotation for any Spring-managed component. Auto-registers as a bean with singleton scope.

```java
@Component
public class EmailService {
    public void sendEmail(String to, String subject) {
        System.out.println("Sending email to " + to);
    }
}

// Bean name: emailService (class name with first letter lowercase)
// Can be injected:
@Autowired
private EmailService emailService;
```

### @Service (Business Logic)

Semantically identical to @Component, but signals business/domain logic layer.

```java
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    public Order createOrder(Order order) {
        // Business logic
        return orderRepository.save(order);
    }
}

// Bean name: orderService
// Best for: domain-driven services, business operations
```

### @Repository (Data Access)

Semantically identical to @Component, but signals data access layer. Also translates database exceptions to Spring DataAccessException.

```java
@Repository
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long id) {
        try {
            return jdbcTemplate.queryForObject(
                "SELECT * FROM users WHERE id = ?",
                new Object[]{id},
                new UserRowMapper()
            );
        } catch (EmptyResultDataAccessException e) {
            // Spring translates SQL exceptions
            throw new UserNotFoundException("User not found");
        }
    }
}

// Bean name: userRepository
// Best for: DAOs, database operations, ORM repositories
```

### @Controller (Web Layer - MVC)

Signals a controller that handles HTTP requests. Returns view names (HTML, JSP).

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "user-detail";  // View name (returns HTML)
    }
    
    @PostMapping
    public String createUser(@ModelAttribute User user) {
        userService.save(user);
        return "redirect:/users";  // Redirect to another view
    }
}

// Bean name: userController
// Best for: MVC controllers that return views
```

### @RestController (Web Layer - REST)

Combination of @Controller + @ResponseBody. Returns JSON/XML, not views.

```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);  // Serialized to JSON
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User saved = userService.save(user);
        return ResponseEntity.status(201).body(saved);  // JSON response
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}

// Bean name: userRestController
// Best for: REST APIs that return JSON/XML
```

---

## Stereotype Comparison Table

| Annotation | Layer | Returns | Use Case |
|-----------|-------|---------|----------|
| **@Component** | Generic | Bean | Utility classes, shared logic |
| **@Service** | Business | Bean | Business logic, domain operations |
| **@Repository** | Data Access | Bean | Database operations, ORM, DAOs |
| **@Controller** | Web/MVC | View | HTML views, form submissions |
| **@RestController** | Web/REST | JSON/XML | REST APIs, microservices |

---

## Beans vs Components

### What is a Bean?

A **bean** is an object instantiated, assembled, and managed by the Spring container.

### What is a Component?

A **component** is a class annotated with a stereotype (@Component, @Service, etc.) that Spring auto-detects and registers as a bean.

### Relationship

```
Component (annotated class) → Spring scans → Registers as Bean → Container manages
```

**Example:**

```java
// This is a COMPONENT
@Service
public class PaymentService {
    public void charge(Order order) {
        // ...
    }
}

// When Spring starts:
// 1. Scans classpath for @Service annotation
// 2. Creates an instance of PaymentService
// 3. Registers it as a BEAN in the ApplicationContext
// 4. Bean name: paymentService
// 5. Container manages its lifecycle (init, destroy)
```

### Bean vs Non-Bean Class

```java
// ✅ BEAN - Spring manages it
@Service
public class OrderService {
    public void process(Order order) { }
}

// ❌ NOT A BEAN - You create it manually
public class OrderValidator {
    public boolean isValid(Order order) { }
}

// Usage:
@Service
public class OrderController {
    
    @Autowired
    private OrderService orderService;  // Injected bean
    
    public void handleOrder(Order order) {
        // Create manually - not a bean
        OrderValidator validator = new OrderValidator();
        
        if (validator.isValid(order)) {
            orderService.process(order);
        }
    }
}
```

---

## How Annotations Work

### Component Scanning Process

```
1. ApplicationContext starts
   ↓
2. Scans classpath for packages
   (default: same package as @SpringBootApplication)
   ↓
3. Finds classes with stereotype annotations
   (@Component, @Service, @Controller, @Repository, etc.)
   ↓
4. Instantiates each class
   ↓
5. Registers as bean in ApplicationContext
   (bean name: class name with lowercase first letter)
   ↓
6. Performs dependency injection (@Autowired)
   ↓
7. Container ready for use
```

### Under the Hood: Component Scanning

```yaml
# application.yml
spring:
  application:
    name: my-app

# Equivalent to:
@SpringBootApplication
// Scans: com.example (and all subpackages)
// Includes: @ComponentScan with basePackages
```

```java
@SpringBootApplication
// Equivalent to:
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(basePackages = "com.example")  // Scan this package
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

// Custom component scanning
@SpringBootApplication
@ComponentScan(basePackages = {
    "com.myapp.services",
    "com.myapp.controllers",
    "com.external.library"
})
public class MyApp { }
```

### Automatic Bean Registration

When Spring finds a component:

```java
@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // Dependency
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}

// Behind the scenes:
// 1. Spring instantiates UserService
// 2. Detects @Autowired on 'repository' field
// 3. Looks up UserRepository bean from ApplicationContext
// 4. Injects it into UserService
// 5. Stores UserService bean with key "userService"
```

---

## Bean Scope Annotations

Control how long a bean lives and how many instances exist.

### @Scope("singleton") - Default

One instance per ApplicationContext. Shared across entire application.

```java
@Service
@Scope("singleton")  // Default, can be omitted
public class ApplicationConfig {
    private int counter = 0;
    
    public synchronized void increment() {
        counter++;  // Shared across all injections
    }
    
    public int getCounter() {
        return counter;
    }
}

// Usage:
@Component
public class ComponentA {
    @Autowired
    private ApplicationConfig config;
    
    public void doWork() {
        config.increment();  // Increments shared counter
    }
}

@Component
public class ComponentB {
    @Autowired
    private ApplicationConfig config;
    
    public void doWork() {
        config.increment();  // Same bean, shared counter
    }
}

// Result: counter value is shared between ComponentA and ComponentB
```

### @Scope("prototype")

New instance created every time bean is requested.

```java
@Component
@Scope("prototype")
public class RequestContext {
    private String requestId = UUID.randomUUID().toString();
    
    public String getRequestId() {
        return requestId;
    }
}

// Usage:
@Service
public class RequestHandler {
    @Autowired
    private RequestContext context;  // New instance each time
    
    public void handle(Request req) {
        System.out.println("Request ID: " + context.getRequestId());
        // Each injection gets new RequestContext instance
    }
}
```

### Other Scopes

```java
@Scope("request")      // New bean per HTTP request
@Scope("session")      // One bean per HTTP session
@Scope("application")  // One bean per ServletContext
```

---

## Qualifier & Primary Annotations

### Problem: Multiple Beans of Same Type

```java
// ❌ Ambiguous - which UserRepository?
public interface UserRepository { }

@Repository
public class MysqlUserRepository implements UserRepository { }

@Repository
public class PostgresUserRepository implements UserRepository { }

// UserService doesn't know which to inject
@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // ERROR: which one?
}
```

### Solution 1: @Primary

Mark default bean when multiple options exist.

```java
@Repository
@Primary  // Use this by default
public class MysqlUserRepository implements UserRepository {
    public User findById(Long id) {
        return query("SELECT * FROM users WHERE id = ?");
    }
}

@Repository
public class PostgresUserRepository implements UserRepository {
    public User findById(Long id) {
        return query("SELECT * FROM users WHERE id = ?");
    }
}

@Service
public class UserService {
    @Autowired
    private UserRepository repository;  // Injects MysqlUserRepository (@Primary)
}
```

### Solution 2: @Qualifier

Explicitly specify which bean to inject.

```java
@Repository("mysql")
public class MysqlUserRepository implements UserRepository { }

@Repository("postgres")
public class PostgresUserRepository implements UserRepository { }

@Service
public class UserService {
    
    @Autowired
    @Qualifier("mysql")  // Explicitly choose mysql
    private UserRepository repository;
    
    public User findUser(Long id) {
        return repository.findById(id);
    }
}
```

### Custom Qualifier

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface DatabaseType {
    String value();
}

@Repository
@DatabaseType("mysql")
public class MysqlUserRepository implements UserRepository { }

@Repository
@DatabaseType("postgres")
public class PostgresUserRepository implements UserRepository { }

@Service
public class UserService {
    
    @Autowired
    @DatabaseType("mysql")
    private UserRepository repository;
}
```

---

## Common Annotations Reference

| Annotation | Purpose |
|-----------|---------|
| `@Component` | Generic component (auto-registered bean) |
| `@Service` | Business logic layer |
| `@Repository` | Data access layer |
| `@Controller` | MVC controller (returns views) |
| `@RestController` | REST controller (returns JSON/XML) |
| `@Autowired` | Dependency injection |
| `@Qualifier` | Disambiguate multiple beans |
| `@Primary` | Default bean when multiple options |
| `@Scope` | Bean lifecycle scope |
| `@Lazy` | Defer bean instantiation until first use |
| `@Bean` | Manually register bean (in @Configuration class) |
| `@Configuration` | Declare bean definitions |
| `@Value` | Inject property values |
| `@PropertySource` | Load external properties file |
| `@Profile` | Conditional bean registration |

---

## Real-World Example: Layered Architecture

```java
// 1. DATA ACCESS LAYER (@Repository)
@Repository
public class UserRepository {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{id},
            new UserRowMapper()
        );
    }
}

// 2. BUSINESS LOGIC LAYER (@Service)
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(Long id) {
        // Business logic
        User user = userRepository.findById(id);
        if (user == null) {
            throw new UserNotFoundException("User not found");
        }
        return user;
    }
}

// 3. REST CONTROLLER LAYER (@RestController)
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        return ResponseEntity.ok(user);
    }
}

// Spring bean hierarchy:
// UserRepository (bean)
//   ↓ injected into
// UserService (bean)
//   ↓ injected into
// UserController (bean)
//   ↓ manages HTTP requests
```

---

## Interview Talking Points

- **@Component vs @Service vs @Repository?** All auto-register as beans; semantic difference signals layer (@Service = business logic, @Repository = data access)
- **Component vs Bean?** Component is annotated class; Bean is registered instance in Spring container
- **How does @Autowired work?** Spring scans for @Autowired fields, looks up matching beans in ApplicationContext, injects them
- **Singleton vs Prototype?** Singleton: one instance per ApplicationContext; Prototype: new instance per request
- **@Primary vs @Qualifier?** @Primary: default when multiple beans; @Qualifier: explicitly choose which bean
- **Component scanning?** Spring scans classpath at startup for stereotype annotations, auto-instantiates and registers as beans
- **Bean naming?** Default: class name with first letter lowercase (UserService → userService); can override with @Component("customName")
- **When to use @Bean vs @Component?** @Component for classes you control; @Bean in @Configuration for external libraries or complex instantiation

---

**See also:** [Spring Boot Fundamentals](spring-boot.md), [Spring Boot Advanced](spring-boot-advanced.md)
