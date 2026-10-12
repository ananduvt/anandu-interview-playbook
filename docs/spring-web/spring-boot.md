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
