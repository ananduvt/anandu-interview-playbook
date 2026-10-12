# Spring Boot Circuit Breaker

Implementing resilient microservices: circuit breaker patterns, Resilience4j examples, and comparison of frameworks.

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

---

## Circuit Breaker Frameworks Comparison

| Framework | Type | Status | Approach | Use Case |
|-----------|------|--------|----------|----------|
| **Resilience4j** | Library | ✅ Active | Functional, fluent API, lightweight | **Recommended for Spring Boot** |
| **Hystrix** | Library | ⚠️ Maintenance | Annotation-based, thread pool isolation | Legacy projects, Netflix stack |
| **Vavr** | Library | ✅ Active | Functional (monadic), immutable | Functional programming style |
| **Failsafe** | Library | ✅ Active | Simple, chainable, no Spring dependency | Standalone microservices |
| **Sentinel** | Framework | ✅ Active (Alibaba) | Real-time, dashboard, flow control | High-throughput scenarios |
| **Polly** | Library | ✅ Active (.NET) | Policy chains, transient fault handling | .NET/C# applications |

### Detailed Comparison

#### **Resilience4j (Recommended)**
```java
// Lightweight, modular, no thread pool overhead
@CircuitBreaker(name = "service")
@Retry(name = "service")
@Timeout(name = "service")
public Response callService() { ... }

// Pros: Spring Boot integration, low overhead, modern
// Cons: Relatively new, smaller ecosystem than Hystrix
```

#### **Hystrix (Legacy - Maintenance Mode)**
```java
// Thread pool isolation, older annotation style
@HystrixCommand(
    fallbackMethod = "fallback",
    commandProperties = {
        @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD")
    }
)
public Response callService() { ... }

private Response fallback() { ... }

// Pros: Mature, Netflix battle-tested, well-documented
// Cons: Maintenance mode, thread pool overhead, complex config
// Status: Spring Cloud Netflix team recommends Resilience4j instead
```

#### **Vavr (Functional Style)**
```java
// Functional composition using monads
import io.vavr.control.Try;

public Response callServiceWithVavr() {
    return Try.of(() -> externalService.call())
        .recover(e -> defaultResponse())
        .get();
}

// Pros: Elegant functional syntax, immutable data
// Cons: Steeper learning curve, less Spring integration
```

#### **Failsafe (Lightweight, No Spring Required)**
```java
// Simple, chainable policy approach
CircuitBreaker<Response> breaker = new CircuitBreaker<Response>()
    .withFailureThreshold(5)
    .withDelay(Duration.ofSeconds(30));

Response response = Failsafe.with(breaker, retryPolicy)
    .get(() -> externalService.call());

// Pros: No framework dependencies, simple API, easy to test
// Cons: Limited Spring integration, manual orchestration
```

#### **Sentinel (High Throughput, Real-Time Control)**
```java
// Real-time flow control, dashboard monitoring
@SentinelResource(value = "payment", fallback = "paymentFallback")
public Response processPayment(Order order) {
    // Automatic flow control, circuit breaking, thread isolation
    return paymentGateway.charge(order);
}

public Response paymentFallback(Order order) { ... }

// Pros: Built-in dashboard, real-time tuning, high throughput
// Cons: More heavyweight, requires separate server components
```

### Choosing a Framework

| Scenario | Recommendation | Reason |
|----------|-----------------|--------|
| **New Spring Boot project** | Resilience4j | Spring official support, lightweight, modern |
| **Legacy Netflix microservices** | Hystrix | Already integrated, team familiar |
| **High-throughput SaaS** | Sentinel | Real-time control, dashboard, tuning |
| **Functional programming** | Vavr | Native monadic composition |
| **Standalone (no Spring)** | Failsafe | No dependencies, simple to integrate |
| **Hybrid microservices** | Resilience4j + Sentinel | Resilience4j for services, Sentinel for gateway |

### Combining Multiple Frameworks (Advanced)

```java
@Service
public class HybridResilientService {
    
    @Autowired
    private CircuitBreakerFactory circuitBreakerFactory;
    
    // Use Resilience4j for most calls
    @CircuitBreaker(name = "paymentService")
    public Response callPaymentService() { ... }
    
    // Use Failsafe for legacy service (no Spring dependency)
    public Response callLegacyService() {
        CircuitBreaker<Response> breaker = new CircuitBreaker<Response>()
            .withFailureThreshold(3)
            .withDelay(Duration.ofSeconds(15));
        
        return Failsafe.with(breaker)
            .get(() -> legacyService.call());
    }
    
    // Use Try (Vavr) for functional fallbacks
    public Response callWithVavrFallback() {
        return Try.of(() -> primaryService.call())
            .recover(e -> secondaryService.call())
            .get();
    }
}
```

### Migration Path (Hystrix → Resilience4j)

```java
// Before: Hystrix annotation
@HystrixCommand(fallbackMethod = "fallback")
public Response callService() { ... }

// After: Resilience4j annotation (drop-in replacement)
@CircuitBreaker(name = "service", fallbackMethod = "fallback")
public Response callService() { ... }

// Configuration: YAML instead of annotations/properties
resilience4j:
  circuitbreaker:
    instances:
      service:
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
```

### Interview Talking Points

- **Why circuit breaker?** Prevent cascading failures, fail fast, allow recovery
- **Resilience4j vs Hystrix?** Resilience4j is modern, lighter, Spring-recommended; Hystrix is legacy but battle-tested
- **Thread pool vs. semaphore isolation?** Thread pools (Hystrix): complete isolation but overhead; semaphores (Resilience4j): lower overhead, shared context
- **Combining patterns?** Use circuit breaker + retry + timeout + bulkhead for comprehensive resilience
- **Testing circuit breakers?** Mock failures, verify fallback behavior, test state transitions (CLOSED → OPEN → HALF_OPEN)
