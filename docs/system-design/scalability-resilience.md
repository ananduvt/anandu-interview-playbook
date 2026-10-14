# Scalability & Resilience

## Failover Mechanisms

[https://www.geeksforgeeks.org/failover-mechanisms-in-system-design/](https://www.geeksforgeeks.org/failover-mechanisms-in-system-design/)

A failover mechanism is a system that automatically switches to a backup system when the primary system fails. It's a key part of disaster recovery (DR) and is used to ensure business continuity.

**How does it work?**

* When the primary system fails, the failover mechanism automatically switches to the backup system.
* The backup system mimics the operating system environment of the primary system.
* Requests are redirected from the failed system to the backup system.

**Where is it used?**

* In data centers
* In network services
* In cloud computing environments
* In backend database support
* In servers.

**Benefits**

* Minimizes downtime and data loss
* Ensures business continuity
* Safeguards customer experiences and brand value.

**Key Terminologies**

1. **Circuit Breaker**
   A mechanism that monitors service health. When a predefined number of consecutive failures occurs, it interrupts further calls to the failing service, preventing cascading failures.

2. **Command**
   Represents a potentially risky action or service call wrapped with circuit breaker functionality. It corresponds to the method call making the remote call to another service.

3. **Fallback**
   A backup mechanism providing default or alternative behavior when the command fails or when the circuit breaker is open. It prevents application crashes due to failures.

4. **Timeout**
   The maximum time a command takes to execute before considering it a failure. If the command exceeds this time, it is aborted, and the fallback mechanism is invoked.

5. **Rolling Window**
   The time window within which Hystrix monitors service health. It calculates metrics such as error percentage and request volume within this window to determine the circuit state.

6. **Isolation**
   Hystrix provides isolation of commands using separate threads, thread pools, or semaphores. It ensures that the failure in one command does not affect others, providing fault tolerance.

7. **Metrics**
   Data collected by Hystrix, such as success rate, failure rate, latency, and concurrency. These metrics are used to make decisions about the circuit state and for monitoring and debugging purposes.

8. **Thread Pool**
    Used by Hystrix to execute commands asynchronously. Each command can have its thread pool, providing isolation and preventing resource contention between commands.

**Types of failover**

1. **Cold Standby**
   The backup system is inactive (offline or powered off) until needed, requiring time to boot up and become operational, resulting in longer downtime.

2. **Cozy Standby Failure Mode**
   A warm standby system is partially operational, ready to take over with minimal downtime, but not handling live traffic until the primary system fails.

3. **Warm Standby Failure-Over**
   A fully functional, synchronized backup system is always running in the background, ready to take over immediately, minimizing downtime.

4. **Active-Passive Switching**
   Only one system is active at a time, with the other(s) in standby mode, ready to take over if the active system fails.

5. **Dynamic-Active Switchover**
   Both primary and standby systems are active concurrently, processing traffic and fulfilling requests, ensuring high availability and no downtime.

**Types of Failover Mechanisms**

1. **Active-Passive**
   A primary system is active, with a secondary system in standby, ready to take over if the primary fails.

2. **Active-Active**
   Multiple systems are active simultaneously, distributing the workload, so if one fails, the others can continue serving requests.

3. **Manual Failover**
   Requires human intervention to switch to the backup system.

4. **Automatic Failover**
   The system automatically detects a failure and switches to the backup without human intervention.

5. **Failover Cluster**
   A group of servers that work together to provide high availability, where if one server fails, the others can take over its workload.

6. **Replication**
   Creating copies of data or applications on a secondary system, allowing for a swift switchover in case of failure.

7. **Redundancy**
   Having multiple systems or components that can perform the same function, ensuring that if one fails, others can take over.

8. **Failback**
   The process of switching back to the original system once it has been repaired or restored after a failure.

**Best practices**

* Determine critical systems
* Define recovery objectives
* Add redundancy to hardware, networking, storage, and power supply systems
* Employ load balancers
* Automate failover procedures
* Maintain system health

## Resilience patterns (quick reference)
- **Timeout · Retry** (idempotent + backoff+jitter) **· Circuit breaker · Bulkhead · Fallback/graceful degradation.**
- **Resilience4j** (successor to Hystrix) provides these as composable decorators/annotations in Spring.

---

## Java Scalability Patterns

Building scalable Java applications requires attention to concurrency, resource management, and architecture.

### Vertical Scaling (Single Server)

**Increase resources** on existing server: CPU, memory, disk.

#### Thread Management

```java
// ❌ Bad: Unbounded thread creation
@RestController
public class UserController {
    @PostMapping("/process")
    public void processData() {
        new Thread(() -> {
            expensiveOperation();  // Unbounded threads → OutOfMemoryError
        }).start();
    }
}

// ✅ Good: Thread pool (bounded, reusable)
@Configuration
public class ThreadPoolConfig {
    @Bean
    public ExecutorService executorService() {
        return Executors.newFixedThreadPool(10);  // Max 10 threads
    }
}

@RestController
public class UserController {
    @Autowired
    private ExecutorService executorService;
    
    @PostMapping("/process")
    public ResponseEntity<String> processData() {
        executorService.submit(() -> expensiveOperation());
        return ResponseEntity.accepted().build();
    }
}

// Spring way: @Async with thread pool
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class UserService {
    @Async  // Runs in thread pool
    public void processUserAsync(Long userId) {
        expensiveOperation();
    }
}
```

#### Connection Pooling

```yaml
# Limit DB connections (don't exhaust pool)
spring:
  datasource:
    hikari:
      maximum-pool-size: 20  # Max 20 concurrent connections
      minimum-idle: 5        # Keep 5 idle connections
      maximum-lifetime: 1800000  # 30 min
      idle-timeout: 600000       # 10 min before closing
      connection-timeout: 30000  # 30 sec wait for connection
```

```java
// Don't hold connections long
@Service
public class UserService {
    
    // ❌ Bad: Holds connection during sleep
    @Transactional
    public void slowOperation() {
        User user = userRepo.findById(1L).orElse(null);
        Thread.sleep(10000);  // Blocks connection for 10 sec
        userRepo.save(user);
    }
    
    // ✅ Good: Release connection early
    @Transactional
    public void quickOperation() {
        User user = userRepo.findById(1L).orElse(null);
        // Connection released here
    }
    
    // Separate async work
    @Async
    public void doBackgroundWork(User user) {
        Thread.sleep(10000);  // Not blocking connection
    }
}
```

#### Memory Management & GC

```java
// ❌ Bad: Large objects, memory leaks
public class DataCache {
    private Map<Integer, byte[]> largeDataCache = new HashMap<>();  // Unbounded growth
    
    public void addData(int id, byte[] data) {
        largeDataCache.put(id, data);  // Never removed → heap fills up
    }
}

// ✅ Good: Bounded cache with eviction
@Configuration
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager("data");
        manager.setCaffeine(
            Caffeine.newBuilder()
                .maximumSize(10000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
        );
        return manager;
    }
}

// Monitor GC
@Component
public class GcMonitor {
    @Scheduled(fixedRate = 60000)
    public void logGcInfo() {
        long maxMemory = Runtime.getRuntime().maxMemory() / (1024 * 1024);
        long totalMemory = Runtime.getRuntime().totalMemory() / (1024 * 1024);
        long freeMemory = Runtime.getRuntime().freeMemory() / (1024 * 1024);
        
        System.out.println("Max: " + maxMemory + "MB, Used: " + (totalMemory - freeMemory) + "MB");
    }
}
```

### Horizontal Scaling (Multiple Servers)

**Add more servers** behind a load balancer.

#### Stateless Design

```java
// ❌ Bad: Server-side state not scalable
@RestController
public class CartController {
    private Map<String, Cart> sessionCarts = new HashMap<>();  // Server-specific state
    
    @PostMapping("/cart/add")
    public ResponseEntity<Cart> addToCart(String sessionId, Product product) {
        sessionCarts.get(sessionId).add(product);  // Lost when switching servers
        return ResponseEntity.ok(sessionCarts.get(sessionId));
    }
}

// ✅ Good: Stateless, session in external store (Redis)
@Configuration
@EnableSpringHttpSession
public class HttpSessionConfig {
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory();
    }
}

@RestController
public class CartController {
    @PostMapping("/cart/add")
    public ResponseEntity<Cart> addToCart(HttpSession session, Product product) {
        Cart cart = (Cart) session.getAttribute("cart");  // From Redis, not local
        cart.add(product);
        session.setAttribute("cart", cart);  // Persisted to Redis
        return ResponseEntity.ok(cart);
    }
}
```

#### Database Read Replicas

```java
// Separate read & write databases
@Configuration
public class RoutingDataSourceConfig {
    
    @Bean
    public DataSource readDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://read-replica:3306/db")
            .build();
    }
    
    @Bean
    public DataSource writeDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://primary-db:3306/db")
            .build();
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Reads go to replica (non-modifying)
    @Transactional(readOnly = true)
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    Optional<User> findByEmail(String email);
    
    // Writes go to primary (modifying)
    @Transactional(readOnly = false)
    User save(User user);
}
```

#### Caching Layer

```java
// Reduce database load with distributed cache
@Service
public class UserService {
    
    // Cache miss → DB query; cache hit → immediate return
    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}

// All servers share same Redis cache
```

#### Async Processing & Queues

```java
// Don't block on slow operations
@RestController
public class OrderController {
    
    // ❌ Slow: Wait for email, payment, shipment (10+ seconds)
    @PostMapping("/order/slow")
    public ResponseEntity<Order> createOrderSync(Order order) {
        Order saved = orderRepo.save(order);
        emailService.sendConfirmation(saved);  // Blocking
        paymentService.charge(saved);          // Blocking
        shipmentService.schedule(saved);       // Blocking
        return ResponseEntity.ok(saved);
    }
    
    // ✅ Fast: Queue async work, return immediately
    @PostMapping("/order/async")
    public ResponseEntity<Order> createOrderAsync(Order order) {
        Order saved = orderRepo.save(order);
        
        // Publish events (async)
        eventPublisher.publishEvent(new OrderCreatedEvent(saved));
        
        return ResponseEntity.accepted().body(saved);  // Return 202, job queued
    }
    
    // Event listeners process in background
    @Component
    public class OrderEventListeners {
        @EventListener
        public void onOrderCreated(OrderCreatedEvent event) {
            Order order = event.getOrder();
            emailService.sendConfirmation(order);
            paymentService.charge(order);
            shipmentService.schedule(order);
        }
    }
}

// Or use message queue (RabbitMQ, Kafka)
@Service
public class OrderService {
    
    public void createOrder(Order order) {
        orderRepo.save(order);
        // Publish to message broker
        messageTemplate.convertAndSend("orders.created", order);
    }
}

@Service
public class OrderEventListener {
    
    @RabbitListener(queues = "orders.created")
    public void processOrderCreated(Order order) {
        emailService.sendConfirmation(order);
        paymentService.charge(order);
        shipmentService.schedule(order);
    }
}
```

#### Load Balancing

```yaml
# Kubernetes: Distribute traffic across pod replicas
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080

# Replicas handled by deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3  # 3 instances behind load balancer
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: user-service:1.0
          ports:
            - containerPort: 8080
```

### Performance Metrics to Monitor

```java
// CPU: Keep <70% for headroom
// Memory: Keep <80% heap usage
// Threads: Monitor thread count, queue size
// DB Connections: Track active/idle connections
// Request Latency: P50, P95, P99 percentiles
// Throughput: Requests per second

@Component
public class PerformanceMonitor {
    private final MeterRegistry meterRegistry;
    
    public PerformanceMonitor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        setupMetrics();
    }
    
    private void setupMetrics() {
        // Thread count gauge
        Gauge.builder("jvm.threads.active", () -> Thread.activeCount())
            .register(meterRegistry);
        
        // Memory gauge
        Gauge.builder("jvm.memory.heap", () -> 
            Runtime.getRuntime().totalMemory() / (1024 * 1024))
            .description("Heap memory in MB")
            .register(meterRegistry);
    }
}
```

### Scaling Decision Tree

```
Is your app slow?
├─ YES: CPU-bound?
│  ├─ YES: Vertical scale (faster CPU, more cores)
│  └─ NO: I/O-bound?
│     ├─ YES: Use async/non-blocking I/O
│     └─ Optimize: Cache, index DB, lazy load
│
└─ NO: Running out of resources?
   ├─ Memory: Reduce cache size, fix memory leaks
   ├─ Connections: Use connection pooling, read replicas
   ├─ Disk: Purge old data, archive logs
   └─ Need high availability?
      ├─ YES: Horizontal scale + load balancer
      └─ NO: Vertical scale sufficient
```
