# Spring Boot Advanced

Advanced Spring Boot concepts: dependency injection patterns, bean lifecycle management, application startup/shutdown, environment-specific profiles, and fault tolerance.

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

---

---

## Circuit Breaker Pattern

A circuit breaker is a fault-tolerance pattern that prevents cascading failures in distributed systems. It monitors service calls and temporarily stops calling a failing service to allow recovery.

### Circuit Breaker States

```
CLOSED (normal operation)
  ↓ (failures exceed threshold)
OPEN (block calls, fail fast)
  ↓ (after timeout, attempt recovery)
HALF_OPEN (test if service recovered)
  ↓ (success → CLOSED; failure → OPEN)
```

### Spring Cloud Circuit Breaker (Modern Approach)

**Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

**Configuration:**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10          # Track last 10 calls
        failureRateThreshold: 50       # Open if 50% fail
        slowCallRateThreshold: 100     # Open if 100% are slow
        slowCallDurationThreshold: 2s  # Call > 2s is "slow"
        waitDurationInOpenState: 30s   # Wait 30s before trying again
        permittedNumberOfCallsInHalfOpenState: 3  # Allow 3 test calls
      
      # Define fallback separately (optional)
      default:
        slidingWindowSize: 100
        failureRateThreshold: 50
```

**Implementation with `@CircuitBreaker` annotation:**
```java
@Service
public class PaymentService {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentService.class);
    
    @Autowired
    private RestTemplate restTemplate;
    
    // Protected method with circuit breaker
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(Order order) {
        logger.info("Processing payment for order: {}", order.getId());
        
        // This call will be monitored by circuit breaker
        ResponseEntity<PaymentResponse> response = restTemplate.postForEntity(
            "http://payment-service/api/pay",
            order,
            PaymentResponse.class
        );
        
        return response.getBody();
    }
    
    // Fallback method (called when circuit is OPEN)
    // Signature must match the protected method (same params, return type)
    private PaymentResponse paymentFallback(Order order, CircuitBreakerOpenException ex) {
        logger.warn("Circuit breaker OPEN - using fallback for order: {}", order.getId());
        
        // Option 1: Return cached/default response
        return new PaymentResponse(
            order.getId(),
            "PENDING",  // Mark as pending for manual retry
            "Payment service temporarily unavailable. Will retry automatically."
        );
        
        // Option 2: Throw exception and let caller handle
        // throw new PaymentUnavailableException("Payment service down", ex);
    }
}
```

**Circuit Breaker with Resilience4j and Retry:**
```java
@Service
public class OrderService {
    
    @Autowired
    private PaymentService paymentService;
    
    // Combine circuit breaker + retry
    @CircuitBreaker(name = "orderService", fallbackMethod = "orderFallback")
    @Retry(name = "orderService", fallbackMethod = "orderFallback")
    public Order placeOrder(Order order) {
        // Try up to 3 times (default)
        // If still fails, circuit breaker takes over
        PaymentResponse payment = paymentService.processPayment(order);
        
        if (!"SUCCESS".equals(payment.getStatus())) {
            throw new PaymentFailedException("Payment failed: " + payment.getReason());
        }
        
        order.setStatus("CONFIRMED");
        return order;
    }
    
    private Order orderFallback(Order order, Exception ex) {
        order.setStatus("RETRY_LATER");
        return order;
    }
}
```

### Monitoring Circuit Breaker Status

```java
@RestController
@RequestMapping("/api/circuit-breaker")
public class CircuitBreakerMonitorController {
    
    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;
    
    @GetMapping("/status")
    public Map<String, Object> getStatus() {
        CircuitBreaker cb = circuitBreakerRegistry.circuitBreaker("paymentService");
        CircuitBreakerMetrics metrics = cb.getMetrics();
        
        return Map.of(
            "state", cb.getState(),  // CLOSED, OPEN, HALF_OPEN
            "failureRate", metrics.getFailureRate() + "%",
            "slowCallRate", metrics.getSlowCallRate() + "%",
            "successfulCalls", metrics.getNumberOfSuccessfulCalls(),
            "failedCalls", metrics.getNumberOfFailedCalls(),
            "slowCalls", metrics.getNumberOfSlowCalls()
        );
    }
}
```

### Programmatic Circuit Breaker Usage

```java
@Service
public class InventoryService {
    
    @Autowired
    private CircuitBreakerFactory circuitBreakerFactory;
    
    public InventoryResponse checkInventory(String itemId) {
        // Create circuit breaker dynamically
        CircuitBreaker circuitBreaker = circuitBreakerFactory
            .create("inventoryService");
        
        return circuitBreaker.run(
            () -> {
                // Protected call
                return callInventoryAPI(itemId);
            },
            throwable -> {
                // Fallback
                return new InventoryResponse(itemId, 0, "Service unavailable");
            }
        );
    }
    
    private InventoryResponse callInventoryAPI(String itemId) {
        // Actual API call
        return new InventoryResponse(itemId, 100, "Available");
    }
}
```

### Real-World Scenario

```java
@Service
public class PaymentGatewayService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Autowired
    private NotificationService notificationService;
    
    @CircuitBreaker(name = "externalPaymentGateway", fallbackMethod = "handlePaymentFailure")
    @Retry(name = "externalPaymentGateway")  // Retry before circuit opens
    @Timeout(name = "externalPaymentGateway")  // Fail if takes > configured time
    public ChargeResult chargeCard(PaymentRequest request) {
        try {
            ResponseEntity<ChargeResult> response = restTemplate.postForEntity(
                "https://api.payment-gateway.com/charge",
                request,
                ChargeResult.class
            );
            
            return response.getBody();
            
        } catch (RestClientException e) {
            // Retry will catch this and retry automatically
            // If retries exhaust, circuit breaker opens
            throw e;
        }
    }
    
    // Fallback: handle gracefully when circuit is open
    private ChargeResult handlePaymentFailure(
        PaymentRequest request,
        Exception ex
    ) {
        String transactionId = UUID.randomUUID().toString();
        
        // Log for manual processing
        logger.error("Payment gateway circuit open. Storing for retry. TxnID: {}", transactionId, ex);
        
        // Notify customer
        notificationService.sendEmail(
            request.getCustomerEmail(),
            "Payment Status",
            "Your payment is being processed. Reference: " + transactionId
        );
        
        // Return pending status (not failed)
        return new ChargeResult(
            transactionId,
            "PENDING",
            "Gateway temporarily unavailable. Retry scheduled.",
            request.getAmount()
        );
    }
}
```

**Configuration for the above scenario:**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      externalPaymentGateway:
        slidingWindowSize: 20
        failureRateThreshold: 50
        waitDurationInOpenState: 60s
        permittedNumberOfCallsInHalfOpenState: 5
  
  retry:
    instances:
      externalPaymentGateway:
        maxAttempts: 3
        waitDuration: 1000
        retryExceptions:
          - java.net.SocketTimeoutException
          - org.springframework.web.client.ResourceAccessException
  
  timelimit:
    instances:
      externalPaymentGateway:
        timeoutDuration: 5s  # Fail if call takes > 5 seconds
```

### Circuit Breaker Benefits & Trade-offs

| Benefit | Example |
|---------|---------|
| **Fail fast** | Return error immediately instead of waiting for timeout |
| **Prevent cascades** | Stop calling failed service; doesn't overload it |
| **Graceful degradation** | Use fallback (cache, default value) instead of complete failure |
| **Auto-recovery** | Automatically retry after timeout (HALF_OPEN state) |
| **Observability** | Monitor health via metrics (failure rate, latency) |

| Trade-off | Mitigation |
|-----------|-----------|
| **Complexity** | Use annotations; framework handles state transitions |
| **Stale fallback data** | Combine with cache; document fallback behavior |
| **False positives** | Tune thresholds per service (slow vs. fast endpoints) |
| **Cascading opens** | Use bulkhead pattern (separate thread pools) for isolation |

### Related Patterns

- **Retry**: Re-attempt failed calls (exponential backoff)
- **Timeout**: Fail fast if call takes too long
- **Bulkhead**: Isolate resources (thread pools) per service
- **Fallback**: Provide alternate behavior when primary fails
