# Spring Boot

## Spring Boot

| Spring | Spring Boot |
| :---- | :---- |
| Spring is an open-source lightweight framework widely used to develop enterprise applications. | Spring Boot is built on top of the conventional spring framework, widely used to develop REST APIs. |
| The most important feature of the Spring Framework is dependency injection. | The most important feature of the Spring Boot is Autoconfiguration. |
| It helps to create a loosely coupled application. | It helps to create a stand-alone application. |
| To run the Spring application, we need to set the server explicitly. | Spring Boot provides embedded servers such as Tomcat and Jetty etc. |
| To run the Spring application, a deployment descriptor is required. | There is no requirement for a deployment descriptor. |
| To create a Spring application, the developers write lots of code. | It reduces the lines of code. |
| It doesn’t provide support for the in-memory database. | It provides support for the in-memory database such as H2. |
| Developers need to write boilerplate code for smaller tasks. | In Spring Boot, there is reduction in boilerplate code. |
| Developers have to define dependencies manually in the pom.xml file. | pom.xml file internally handles the required dependencies. |

## Core Concepts

### IoC & Dependency Injection
- **Inversion of Control (IoC)**: Spring manages object creation and lifecycles; beans are wired at runtime
- **Dependency Injection**: Dependencies passed via constructor, setter, or field injection
- **Bean**: Managed Spring object; scope: **singleton** (default, one per context), **prototype** (new each time), **request/session** (web)

### Spring Boot Advantages
- **Auto-configuration**: Detects classpath & creates beans automatically (e.g., `@EnableAutoConfiguration` or `@SpringBootApplication`)
- **Embedded servers**: Tomcat/Jetty/Undertow bundled; no WAR deployment needed
- **Starter POM**: Curated dependency sets (`spring-boot-starter-web`, etc.) eliminate version conflicts
- **Externalized config**: Properties from `application.yml`, env vars, command-line args (in precedence order)
- **Actuator**: `/actuator/health`, `/metrics`, `/loggers` for monitoring

---

## Dependency Injection (DI) Deep-Dive

Dependency Injection is the core of Spring: instead of objects creating their dependencies, the framework provides them.

### Without DI (Tight Coupling)
```java
// ❌ Bad: UserService creates its own dependency
@Service
public class UserService {
    private UserRepository repo = new UserRepository();  // Hard to test, tightly coupled
    
    public User getUser(Long id) {
        return repo.findById(id);
    }
}
```

### With DI (Loose Coupling)
```java
// ✅ Good: Spring injects dependency
@Service
public class UserService {
    private final UserRepository repo;  // Immutable
    
    // Constructor injection - preferred (testable, clear dependencies)
    public UserService(UserRepository repo) {
        this.repo = repo;
    }
    
    public User getUser(Long id) {
        return repo.findById(id);
    }
}

// In tests: inject mock
@Test
public void testGetUser() {
    UserRepository mockRepo = mock(UserRepository.class);
    UserService service = new UserService(mockRepo);  // Easy to inject test double
    // ...
}
```

### DI Patterns

| Pattern | Example | Pros | Cons |
|---------|---------|------|------|
| **Constructor** | `public Service(Repo r) {}` | Immutable, clear deps, required fields | Can't use circular deps |
| **Setter** | `@Autowired void setRepo(Repo r)` | Optional deps, flexible | Mutable, unclear contract |
| **Field** | `@Autowired Repo repo` | Concise | Can't test without framework, hides deps |

### Bean Lifecycle

```java
// Lifecycle hooks (methods called by Spring at specific stages)
@Service
public class MyService implements InitializingBean, DisposableBean {
    
    private DatabaseConnection dbConn;
    
    // Stage 1: Spring instantiates bean
    public MyService() {
        System.out.println("1. Constructor called");
    }
    
    // Stage 2: Spring injects dependencies
    @Autowired
    public void setDbConnection(DatabaseConnection conn) {
        System.out.println("2. Setter called - dependency injected");
        this.dbConn = conn;
    }
    
    // Stage 3: Post-construction initialization (after DI, before bean ready)
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct - initializing resources");
        dbConn.connect();
    }
    
    // Stage 4: Bean in use
    public void doWork() {
        System.out.println("4. Bean in use");
    }
    
    // Stage 5: Pre-destruction cleanup (before removal from context)
    @PreDestroy
    public void cleanup() {
        System.out.println("5. @PreDestroy - cleaning up resources");
        dbConn.close();
    }
    
    // Alternative (implements interface)
    @Override
    public void afterPropertiesSet() throws Exception { }
    
    @Override
    public void destroy() throws Exception { }
}
```

---

## Spring Application Lifecycle

Complete flow from startup to shutdown:

### 1. **Application Startup**
```
SpringApplication.run(Application.class, args)
  ↓
1a. Create ApplicationContext (environment setup, config loading)
  ↓
1b. Load beans from @Configuration classes and classpath scanning
  ↓
1c. Auto-configuration (detect classpath, create default beans)
  ↓
1d. Instantiate and wire beans (DI phase)
  ↓
1e. Call @PostConstruct on all beans (init hooks)
  ↓
1f. Publish ApplicationReadyEvent (app ready for requests)
  ↓
Application running (listen on port, handle requests)
```

### 2. **Bean Initialization Order**

```
Bean Definition → Bean Instantiation → Dependency Injection → @PostConstruct → Ready for use
```

**Example flow:**
```
Order of bean creation:
1. Singleton beans created eagerly (default scope)
2. Dependencies satisfied via constructor/setter
3. @PostConstruct methods called
4. Prototype beans created on-demand (each time requested)
```

### 3. **Runtime - Request Handling**

```
HTTP Request
  ↓
DispatcherServlet (front controller)
  ↓
Handler Mapping (find @Controller/@RestController)
  ↓
Controller method execution (with DI resolving method args)
  ↓
@Transactional open (if applied)
  ↓
Business logic (service layer)
  ↓
Database operations (repository layer)
  ↓
@Transactional commit/rollback
  ↓
Response serialization (JSON/XML)
  ↓
HTTP Response
```

### 4. **Application Shutdown**

```
Shutdown signal (SIGTERM, ctrl+C)
  ↓
@PreDestroy called on all beans (cleanup hooks)
  ↓
Connections closed (DB, cache, etc.)
  ↓
ApplicationContext destroyed
  ↓
Process exits
```

---

## Profiles & Environment-Specific Configuration

Profiles allow different configurations for different environments (dev, test, prod).

### Profile Activation Methods

```yaml
# Method 1: application.yml (base config)
spring:
  profiles:
    active: dev

# Method 2: Command line
# java -jar app.jar --spring.profiles.active=prod

# Method 3: Environment variable
# export SPRING_PROFILES_ACTIVE=prod

# Method 4: application.properties
# spring.profiles.active=dev

# Method 5: Programmatic
# SpringApplication app = new SpringApplication(MyApp.class);
# app.setAdditionalProfiles("prod");
```

### Profile-Specific Configuration

```yaml
# application.yml (shared across all profiles)
spring:
  application:
    name: my-service
  profiles:
    active: dev
  jpa:
    hibernate:
      ddl-auto: validate  # default: validate schema

# application-dev.yml (dev overrides)
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myapp_dev
    username: dev_user
    password: dev_pass
  jpa:
    hibernate:
      ddl-auto: create-drop  # recreate schema each time
    show-sql: true  # log SQL
  
logging:
  level:
    root: INFO
    com.myapp: DEBUG  # verbose logging for app code

# application-prod.yml (prod overrides)
spring:
  datasource:
    url: jdbc:mysql://prod-db.example.com:3306/myapp_prod
    username: ${DB_USER}  # inject from env var
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate  # don't touch schema
    show-sql: false
  cache:
    type: redis
    redis:
      host: ${REDIS_HOST}

logging:
  level:
    root: WARN  # minimal logging in prod

server:
  port: 8443  # HTTPS
  ssl:
    key-store: /etc/app/keystore.p12
```

### Profile-Specific Beans

```java
// Only created if "dev" profile is active
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource devDataSource() {
        return new H2DataSource();  // In-memory DB for dev
    }
}

// Only created if "prod" profile is active
@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource prodDataSource() {
        return new PooledMysqlDataSource();  // Production DB pool
    }
}

// Conditional logic
@Service
public class AnalyticsService {
    @Autowired(required = false)
    private Optional<DatadogMetrics> metrics;  // null if not in "prod"
    
    public void logMetric(String event) {
        if (metrics.isPresent()) {
            metrics.get().send(event);  // Only in prod
        }
    }
}
```

### Using Profiles in Code

```java
@Component
public class PaymentService {
    
    @Autowired
    private Environment env;
    
    public void processPayment(Order order) {
        if (env.getProperty("spring.profiles.active").contains("prod")) {
            // Real payment gateway
            realGateway.charge(order);
        } else {
            // Mock payment in dev
            mockGateway.simulateCharge(order);
        }
    }
}

// Better: use @Value with property
@Component
public class MailService {
    
    @Value("${mail.enabled:false}")  // defaults to false
    private boolean mailEnabled;
    
    public void sendEmail(String to, String subject) {
        if (mailEnabled) {
            // Send real email
        }
    }
}
```

## Common Annotations

### Application Setup
| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@SpringBootApplication` | Combines `@Configuration`, `@ComponentScan`, `@EnableAutoConfiguration` | Main class |
| `@Configuration` | Marks class as bean factory (like XML `<beans>`) | Bean definitions |
| `@Bean` | Method-level; returns managed object | `@Bean public MyService service() {}` |
| `@ComponentScan` | Scans packages for `@Component`, `@Service`, etc. | Base path for auto-discovery |
| `@EnableAutoConfiguration` | Activates auto-config; rarely used (in `@SpringBootApplication`) | Config class |

### Stereotypes (Component Registration)
| Annotation | Purpose |
|-----------|---------|
| `@Component` | Generic managed bean |
| `@Service` | Business logic layer |
| `@Repository` | Data access layer (enables exception translation, transaction support) |
| `@Controller` | Web layer; handles HTTP requests |
| `@RestController` | Like `@Controller` + `@ResponseBody` (returns JSON/XML, not views) |

### Dependency Injection
```java
// Constructor injection (preferred - immutable, testable)
@Service
public class UserService {
    private final UserRepository repo;
    
    public UserService(UserRepository repo) {
        this.repo = repo;
    }
}

// Setter injection
@Service
public class OrderService {
    private PaymentGateway gateway;
    
    @Autowired
    public void setGateway(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}

// Field injection (discouraged - hard to test, hides dependencies)
@Service
public class ReportService {
    @Autowired
    private ReportRepository repo;
}

// Qualifier - pick specific bean when multiple implementations exist
@Autowired
@Qualifier("primaryDB")
private DataSource dataSource;

// Optional dependency
@Autowired(required = false)
private Logger logger;
```

### Request Handling (Spring MVC)
| Annotation | Purpose |
|-----------|---------|
| `@GetMapping` | GET request; shorthand for `@RequestMapping(method=GET)` |
| `@PostMapping` | POST request |
| `@PutMapping` | PUT request |
| `@DeleteMapping` | DELETE request |
| `@PatchMapping` | PATCH request (partial update) |
| `@RequestParam` | Query/form parameter: `@RequestParam String name` |
| `@PathVariable` | URL path segment: `@GetMapping("/user/{id}")` with `@PathVariable Long id` |
| `@RequestBody` | JSON/XML body deserialized to object |
| `@ResponseBody` | Return value serialized to JSON/XML (implicit in `@RestController`) |
| `@RequestHeader` | HTTP header: `@RequestHeader String authorization` |
| `@CookieValue` | Cookie parameter |

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) { ... }
    
    @PostMapping
    public User create(@RequestBody User user) { ... }
    
    @PutMapping("/{id}")
    public User update(@PathVariable Long id, @RequestBody User user) { ... }
    
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) { ... }
    
    @GetMapping
    public List<User> list(@RequestParam(defaultValue = "0") int page) { ... }
}
```

### Configuration & Properties
| Annotation | Purpose |
|-----------|---------|
| `@ConfigurationProperties` | Bind config to object (type-safe); e.g., `@ConfigurationProperties(prefix="app.db")` maps `app.db.host`, `app.db.port` to fields |
| `@Value` | Inject single property: `@Value("${app.name}")` |
| `@Profile` | Conditional bean/config per environment: `@Profile("prod")`, `@Profile("!prod")` |
| `@Conditional` | Complex condition: `@Conditional(MyCondition.class)` |
| `@EnableConfigurationProperties` | Register `@ConfigurationProperties` classes |

```java
@Configuration
@ConfigurationProperties(prefix = "app.database")
public class DatabaseConfig {
    private String host;      // app.database.host
    private int port;         // app.database.port
    private String username;  // app.database.username
}

// Use in beans
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource(DatabaseConfig config) {
        // Build DataSource from config
    }
}
```

### Exception Handling
| Annotation | Purpose |
|-----------|---------|
| `@ControllerAdvice` | Global exception handler; catches exceptions from all controllers |
| `@ExceptionHandler` | Method-level; handles specific exception type |
| `@ResponseStatus` | HTTP status code for exception response |

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(UserNotFoundException ex) {
        return new ErrorResponse(ex.getMessage());
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex) {
        return new ErrorResponse("Internal error");
    }
}
```

### Transactions
```java
@Service
public class OrderService {
    
    // ACID: all-or-nothing. If exception, rollback automatically
    @Transactional
    public void createOrder(Order order) {
        orderRepo.save(order);
        paymentGateway.charge(order.getAmount());  // If fails, both rolled back
    }
    
    // Read-only (no writes expected; optimization)
    @Transactional(readOnly = true)
    public List<Order> getOrders() { ... }
    
    // Propagation: REQUIRED (default), REQUIRES_NEW, SUPPORTS, etc.
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void audit(String action) { ... }
}
```

### Other Useful Annotations
| Annotation | Purpose |
|-----------|---------|
| `@Async` | Run method in thread pool (requires `@EnableAsync`) |
| `@Scheduled` | Cron/fixed-delay task (requires `@EnableScheduling`) |
| `@Validated` | Enable bean validation (JSR-380) |
| `@Valid` | Trigger validation on parameter/field |
| `@NotNull`, `@NotEmpty`, `@Email`, etc. | Validation constraints |
| `@CrossOrigin` | Allow CORS from specified origins |
| `@PreAuthorize`, `@PostAuthorize` | Security checks (Spring Security) |

## Configuration Files

### Properties Precedence (highest to lowest)
1. Command-line args: `java -jar app.jar --server.port=9000`
2. System properties: `-Dkey=value`
3. Environment variables: `SPRING_APPLICATION_NAME=myapp`
4. `application.yml` (active profile overrides base): `application-prod.yml`
5. `application-default.properties`

```yaml
# application.yml
spring:
  application:
    name: my-service
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: root
  jpa:
    hibernate:
      ddl-auto: update
  jackson:
    serialization:
      write-dates-as-timestamps: false

server:
  port: 8080
  servlet:
    context-path: /api

app:
  name: My App
  version: 1.0.0
```

### Profiles
```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db
  jpa:
    show-sql: true

# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/db
  jpa:
    show-sql: false
```

Activate: `java -jar app.jar --spring.profiles.active=prod`

## Hystrix circuit breaker

[https://www.geeksforgeeks.org/implementing-a-basic-circuit-breaker-with-hystrix-in-spring-boot-microservices/](https://www.geeksforgeeks.org/implementing-a-basic-circuit-breaker-with-hystrix-in-spring-boot-microservices/)
[https://www.baeldung.com/spring-cloud-netflix-hystrix](https://www.baeldung.com/spring-cloud-netflix-hystrix)
[https://cloud.spring.io/spring-cloud-netflix/multi/multi__circuit_breaker_hystrix_clients.html](https://cloud.spring.io/spring-cloud-netflix/multi/multi__circuit_breaker_hystrix_clients.html)

The Hystrix circuit breaker is an open-source Java library that helps prevent cascading failures in distributed systems. It's a fault tolerance technique that monitors services and temporarily rejects calls when they're not behaving normally.

**How it works**

* Hystrix monitors services for failures
* When a service fails above a certain threshold, the circuit breaker opens
* The circuit breaker prevents further calls to the failing service
* The developer can provide a fallback, such as another Hystrix protected call, static data, or an empty value

**Hystrix benefits**

* Prevents cascading failures: Stops failures from spreading throughout a system
* Improves resilience: Helps systems recover quickly from failures
* Provides fault tolerance: Protects against latency and failure from dependencies
* Enables monitoring and alerting: Provides near real-time monitoring and alerting
