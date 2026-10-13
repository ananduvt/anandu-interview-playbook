# Spring Threading & Async Patterns

Asynchronous processing in Spring: @Async methods, CompletableFuture, TaskExecutor configuration, and virtual thread integration for scalable web applications.

---

## @Async Annotation

Spring's `@Async` annotation executes methods asynchronously on a thread pool without blocking the caller.

### Basic Setup

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean(name = "taskExecutor")
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);           // Min threads
        executor.setMaxPoolSize(20);           // Max threads
        executor.setQueueCapacity(100);        // Queue size
        executor.setThreadNamePrefix("async-"); // Thread name
        executor.initialize();
        return executor;
    }
}

// Alternatively, use virtual threads (Java 21+)
@Configuration
@EnableAsync
public class VirtualThreadAsyncConfig {
    
    @Bean(name = "taskExecutor")
    public TaskExecutor taskExecutor() {
        return new SimpleAsyncTaskExecutor("vt-");  // Virtual threads
    }
}
```

### @Async Method Examples

```java
@Service
public class EmailService {
    
    // 1. Void return (fire-and-forget)
    @Async
    public void sendEmailAsync(String to, String subject, String body) {
        // Runs on thread pool, caller doesn't wait
        Thread.sleep(2000);  // Simulate I/O
        System.out.println("Email sent to " + to);
    }
    
    // 2. Return Future<T> (caller can wait)
    @Async
    public Future<Boolean> sendEmailWithResult(String to, String subject, String body) {
        try {
            Thread.sleep(2000);
            return new AsyncResult<>(true);  // Success
        } catch (Exception e) {
            return new AsyncResult<>(false);  // Failure
        }
    }
    
    // 3. Return CompletableFuture<T> (recommended)
    @Async
    public CompletableFuture<String> sendEmailWithFuture(String to, String subject, String body) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                return "Email sent to " + to;
            } catch (InterruptedException e) {
                return "Failed: " + e.getMessage();
            }
        });
    }
}

@RestController
@RequestMapping("/api")
public class OrderController {
    
    @Autowired
    private EmailService emailService;
    
    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        // Save order
        Order saved = orderRepository.save(order);
        
        // Send email asynchronously (non-blocking)
        emailService.sendEmailAsync(
            order.getCustomerEmail(),
            "Order Confirmed",
            "Your order #" + saved.getId() + " is confirmed"
        );
        
        // Return immediately (email still sending in background)
        return ResponseEntity.status(201).body(saved);
    }
}
```

**Key Points:**
- ✅ `@Async void` methods: fire-and-forget
- ✅ `@Async Future<T>`: caller can wait/check result
- ✅ `@Async CompletableFuture<T>`: recommended, composable
- ❌ Returns from @Async must be void or Future (not plain T)
- ❌ Calling @Async method from same class bypasses async (self-invocation issue)

---

## CompletableFuture in Spring

Powerful API for composing asynchronous operations without nested callbacks.

### Basic Operations

```java
@Service
public class PaymentService {
    
    private final ExternalPaymentGateway gateway;
    private final NotificationService notificationService;
    
    // 1. Simple async operation
    public CompletableFuture<PaymentResponse> processPayment(Order order) {
        return CompletableFuture.supplyAsync(() -> {
            return gateway.charge(order);  // Blocking I/O on executor
        });
    }
    
    // 2. Chain operations (non-blocking)
    public CompletableFuture<String> processPaymentWithNotification(Order order) {
        return processPayment(order)
            .thenApply(response -> {
                // Transform result
                return "Payment " + response.getStatus();
            })
            .thenApply(status -> {
                // More transformations
                return status.toUpperCase();
            })
            .exceptionally(ex -> {
                // Handle errors
                return "Payment failed: " + ex.getMessage();
            });
    }
    
    // 3. Combine multiple async operations
    public CompletableFuture<OrderSummary> getOrderWithPaymentAndNotification(Order order) {
        CompletableFuture<PaymentResponse> paymentFuture = processPayment(order);
        CompletableFuture<Notification> notificationFuture = 
            notificationService.createNotificationAsync(order);
        
        return paymentFuture.thenCombine(notificationFuture, (payment, notification) -> {
            // Both completed, combine results
            return new OrderSummary(order, payment, notification);
        });
    }
    
    // 4. Wait for first to complete
    public CompletableFuture<PaymentResponse> processWithFallback(Order order) {
        CompletableFuture<PaymentResponse> primary = processPayment(order);
        CompletableFuture<PaymentResponse> fallback = processFallbackPayment(order);
        
        return primary.acceptEither(fallback, response -> {
            System.out.println("Payment succeeded: " + response.getStatus());
        }).thenApply(v -> primary.getNow(null));  // Return the winner
    }
    
    // 5. Chain async -> sync -> async
    public CompletableFuture<OrderConfirmation> completeOrder(Order order) {
        return processPayment(order)
            .thenApply(payment -> {
                // Sync: save payment result
                Order updated = order.withPaymentId(payment.getId());
                return orderRepository.save(updated);
            })
            .thenCompose(saved -> {
                // Async: send async notification
                return notificationService.notifyCustomerAsync(saved);
            });
    }
}
```

**Common CompletableFuture methods:**
- `supplyAsync()`: Create future with value
- `thenApply()`: Transform result
- `thenAccept()`: Consume result (void)
- `thenCombine()`: Combine two futures
- `thenCompose()`: Chain async → async
- `acceptEither()`: React to first completion
- `allOf()`: Wait for all futures
- `anyOf()`: Wait for any future
- `exceptionally()`: Handle errors
- `handle()`: Handle result or exception

---

## REST Controller with Async

### Non-blocking REST endpoints

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    // 1. Simple async endpoint
    @PostMapping
    public CompletableFuture<ResponseEntity<Order>> createOrder(@RequestBody Order order) {
        return orderService.createOrderAsync(order)
            .thenApply(saved -> ResponseEntity.status(201).body(saved))
            .exceptionally(ex -> ResponseEntity.status(500).build());
    }
    
    // 2. Async with query/processing
    @GetMapping("/{orderId}/status")
    public CompletableFuture<ResponseEntity<OrderStatus>> getOrderStatus(@PathVariable Long orderId) {
        return orderService.getOrderAsync(orderId)
            .thenApply(order -> new OrderStatus(order))
            .thenApply(ResponseEntity::ok)
            .exceptionally(ex -> ResponseEntity.notFound().build());
    }
    
    // 3. Parallel async requests
    @GetMapping("/{orderId}/details")
    public CompletableFuture<ResponseEntity<OrderDetails>> getOrderDetails(@PathVariable Long orderId) {
        CompletableFuture<Order> orderFuture = orderService.getOrderAsync(orderId);
        CompletableFuture<List<Item>> itemsFuture = orderService.getOrderItemsAsync(orderId);
        CompletableFuture<Payment> paymentFuture = orderService.getPaymentAsync(orderId);
        
        return CompletableFuture.allOf(orderFuture, itemsFuture, paymentFuture)
            .thenApply(v -> new OrderDetails(
                orderFuture.join(),
                itemsFuture.join(),
                paymentFuture.join()
            ))
            .thenApply(ResponseEntity::ok);
    }
}
```

**Benefits:**
- ✅ Non-blocking responses
- ✅ Efficient thread usage
- ✅ Composable async chains
- ✅ Timeout support (via `orTimeout()`)

---

## TaskExecutor Configuration

### Standard Spring TaskExecutor Types

```java
@Configuration
@EnableAsync
public class AsyncConfiguration {
    
    // 1. ThreadPoolTaskExecutor (most common)
    @Bean(name = "defaultExecutor")
    public TaskExecutor defaultExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("spring-async-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        executor.initialize();
        return executor;
    }
    
    // 2. SimpleAsyncTaskExecutor (unbounded threads, use with caution)
    @Bean(name = "simpleExecutor")
    public TaskExecutor simpleExecutor() {
        SimpleAsyncTaskExecutor executor = new SimpleAsyncTaskExecutor();
        executor.setThreadNamePrefix("simple-async-");
        return executor;
    }
    
    // 3. Virtual Thread Executor (Java 21+)
    @Bean(name = "virtualExecutor")
    public TaskExecutor virtualExecutor() {
        return new SimpleAsyncTaskExecutor("vt-");  // Auto-detects virtual threads
    }
    
    // 4. SyncTaskExecutor (for testing, blocks caller)
    @Bean(name = "syncExecutor")
    public TaskExecutor syncExecutor() {
        return new SyncTaskExecutor();  // Synchronous, no parallelism
    }
    
    // 5. Multiple executors for different purposes
    @Bean(name = "emailExecutor")
    public TaskExecutor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(100);
        executor.initialize();
        return executor;
    }
    
    @Bean(name = "reportExecutor")
    public TaskExecutor reportExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(1000);
        executor.initialize();
        return executor;
    }
}

// Use specific executor
@Service
public class MultiExecutorService {
    
    @Async("emailExecutor")
    public CompletableFuture<String> sendEmail(String to) {
        // Runs on email-specific executor
        return CompletableFuture.completedFuture("Email sent");
    }
    
    @Async("reportExecutor")
    public CompletableFuture<String> generateReport() {
        // Runs on report-specific executor
        return CompletableFuture.completedFuture("Report generated");
    }
}
```

---

## Virtual Threads in Spring (Java 21+)

Virtual threads dramatically simplify async handling:

```java
// Before (Java 8-20): Complex async chains needed
@Service
public class LegacyOrderService {
    
    @Async
    public CompletableFuture<OrderResponse> processOrder(Order order) {
        return getPaymentAsync(order)
            .thenCompose(payment -> saveOrderAsync(order, payment))
            .thenCompose(saved -> notifyAsync(saved));
    }
}

// After (Java 21+): Can write blocking code, Spring handles with virtual threads
@Service
public class ModernOrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Async  // Runs on virtual thread, looks synchronous!
    public void processOrder(Order order) {
        // Blocking code that looks synchronous but runs on virtual thread
        Payment payment = paymentGateway.charge(order);  // Blocks virtual thread
        Order saved = orderRepository.save(order.withPayment(payment));
        notificationService.notify(saved);  // Another blocking call
        
        // All happen on virtual threads, no context switching overhead!
    }
}

// Spring Boot config
@Configuration
public class VirtualThreadConfig {
    
    @Bean
    public TaskExecutor taskExecutor() {
        return new SimpleAsyncTaskExecutor("vt-");  // Virtual threads
    }
    
    @Bean
    public TomcatProtocolHandlerCustomizer<?> tomcatProtocolHandlerCustomizer() {
        // Use virtual threads for HTTP requests (Spring 6.1+)
        return protocolHandler -> protocolHandler.setExecutor(new SimpleAsyncTaskExecutor("vt-"));
    }
}
```

**Virtual threads advantages in Spring:**
- ✅ Write imperative code (no callback chains)
- ✅ Millions of concurrent requests
- ✅ Simple debugging (clear stack traces)
- ✅ Eliminates reactive frameworks complexity

---

## Common Patterns & Best Practices

### Pattern 1: Timeout Handling

```java
public CompletableFuture<PaymentResponse> processWithTimeout(Order order) {
    return processPayment(order)
        .orTimeout(5, TimeUnit.SECONDS)  // Fail if takes > 5s
        .exceptionally(ex -> {
            if (ex instanceof TimeoutException) {
                return new PaymentResponse(order.getId(), "TIMEOUT");
            }
            return new PaymentResponse(order.getId(), "ERROR");
        });
}
```

### Pattern 2: Retry with Exponential Backoff

```java
public CompletableFuture<PaymentResponse> processWithRetry(Order order) {
    return CompletableFuture.supplyAsync(() -> {
        int attempts = 0;
        while (attempts < 3) {
            try {
                return paymentGateway.charge(order);
            } catch (TemporaryFailureException e) {
                attempts++;
                if (attempts < 3) {
                    Thread.sleep((long) Math.pow(2, attempts) * 1000);  // Exponential backoff
                }
            }
        }
        throw new PaymentException("Failed after 3 attempts");
    });
}
```

### Pattern 3: Cascading Failures

```java
public CompletableFuture<OrderConfirmation> processOrderResilient(Order order) {
    return processPayment(order)
        .thenCompose(payment -> {
            if (!payment.isSuccessful()) {
                return CompletableFuture.failedFuture(
                    new PaymentException("Payment failed")
                );
            }
            return saveOrder(order);
        })
        .exceptionallyCompose(ex -> {
            // Fallback: mark as pending
            return markOrderPending(order)
                .thenApply(v -> new OrderConfirmation("PENDING"));
        });
}
```

---

## @Async vs. CompletableFuture vs. Virtual Threads

| Feature | @Async | CompletableFuture | Virtual Threads |
|---------|--------|------------------|-----------------|
| **Syntax** | Simple decorator | Composable chains | Looks synchronous |
| **Return** | void or Future<T> | Always explicit | Imperative code |
| **Composition** | Limited | Excellent | N/A (no futures) |
| **Error Handling** | Try-catch per method | `exceptionally()` | Normal try-catch |
| **Testing** | Need mock executor | Mock with TestExecutor | Native support |
| **Java 21+** | Use with virtual threads | Still valid | **Recommended** |

---

## Interview Talking Points

- **@Async vs. CompletableFuture?** @Async is simpler for basic async; CompletableFuture for complex chains
- **Virtual threads impact on Spring?** Eliminates need for reactive frameworks for I/O-bound apps
- **TaskExecutor sizing?** I/O-bound = large pool or virtual threads; CPU-bound = CPU count
- **Self-invocation issue?** Can't call @Async from same class (proxy limitation); use constructor injection workaround
- **Exception handling in async?** Use `exceptionally()` for CompletableFuture; @Async void needs custom handling
- **When to use @Async void?** Fire-and-forget logging, notifications
- **Spring WebFlux vs. Virtual Threads?** WebFlux for complex reactive chains; virtual threads for simpler I/O

---

**See also:** [Threading Deep-Dive & Virtual Threads](../java-collections-concurrency/threading.md), [Spring Boot Advanced](spring-boot-advanced.md)
