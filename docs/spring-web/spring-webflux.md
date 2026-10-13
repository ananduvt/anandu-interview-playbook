# Spring WebFlux & Reactive Programming

Non-blocking, event-driven web framework for building high-performance, scalable applications.

---

## Reactive Programming Fundamentals

**Reactive Programming** — Responding to changes asynchronously using streams of data.

### Core Principles
1. **Responsive**: React to user requests quickly
2. **Resilient**: Handle failures gracefully
3. **Elastic**: Scale under load
4. **Message-driven**: Event-based communication

### Async/Await vs Reactive
| Aspect | Async/Await | Reactive |
|--------|-----------|----------|
| **Model** | Imperative (what to do) | Declarative (how to handle) |
| **Threads** | Blocking (thread per request) | Non-blocking (event loop) |
| **Resource Efficient** | ❌ Many threads needed | ✅ Few threads, many requests |
| **Readability** | ✅ Easier for beginners | ⚠️ Steeper learning curve |
| **Error Handling** | try/catch | compose operators |
| **Use Case** | Simple async tasks | High-throughput APIs |

---

## Spring WebFlux Architecture

### Mono & Flux (Core Types)

**Mono**: Stream with 0 or 1 element
```java
Mono<String> mono = Mono.just("Hello");         // Single value
Mono<String> empty = Mono.empty();              // No value
Mono<String> error = Mono.error(new Exception()); // Error
```

**Flux**: Stream with 0 to N elements
```java
Flux<Integer> flux = Flux.just(1, 2, 3);       // Multiple values
Flux<Long> interval = Flux.interval(Duration.ofSeconds(1)); // Every second
Flux<String> empty = Flux.empty();              // No values
```

### Operators (Transformations)

```java
// map: Transform each element
Flux.just(1, 2, 3)
    .map(x -> x * 2)
    .subscribe(System.out::println);  // 2, 4, 6

// flatMap: Transform to another Flux (async composition)
Flux.just("user1", "user2")
    .flatMap(user -> fetchUserData(user))  // Returns Flux<UserData>
    .subscribe(userData -> System.out.println(userData));

// filter: Keep matching elements
Flux.just(1, 2, 3, 4, 5)
    .filter(x -> x > 2)
    .subscribe(System.out::println);  // 3, 4, 5

// timeout: Cancel if no element within timeframe
Flux.interval(Duration.ofSeconds(2))
    .timeout(Duration.ofSeconds(1))
    .doOnError(TimeoutException.class, e -> System.out.println("Timeout!"))
    .subscribe();

// retry: Retry on error
fetchData()
    .retry(3)  // Retry up to 3 times
    .subscribe();

// combine: Merge multiple streams
Flux.merge(
    Flux.just(1, 2),
    Flux.just(3, 4)
).subscribe(System.out::println);  // 1, 2, 3, 4 (order may vary)

// zip: Combine elements from multiple streams
Flux.zip(
    Flux.just("a", "b"),
    Flux.just(1, 2)
).subscribe(tuple -> System.out.println(tuple.getT1() + tuple.getT2()));  // a1, b2
```

---

## Spring WebFlux REST Controller

### Mono Response (Single Object)
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private UserRepository repo;
    
    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getUser(@PathVariable Long id) {
        return repo.findById(id)  // Returns Mono<User>
            .map(user -> ResponseEntity.ok(user))  // Transform to ResponseEntity
            .onErrorReturn(ResponseEntity.notFound().build());  // Handle error
    }
    
    @PostMapping
    public Mono<ResponseEntity<User>> createUser(@RequestBody Mono<User> userMono) {
        return userMono
            .flatMap(user -> repo.save(user))  // Save and get result
            .map(saved -> ResponseEntity.created(URI.create("/users/" + saved.getId()))
                .body(saved))
            .onErrorReturn(ResponseEntity.badRequest().build());
    }
}
```

### Flux Response (Multiple Objects)
```java
@GetMapping
public Flux<User> listUsers() {
    return repo.findAll()  // Returns Flux<User>
        .timeout(Duration.ofSeconds(5))
        .onErrorResume(e -> {
            System.out.println("Error fetching users: " + e.getMessage());
            return Flux.empty();
        });
}

@GetMapping("/search")
public Flux<User> searchUsers(@RequestParam String query) {
    return repo.findByNameContaining(query)
        .delayElement(Duration.ofMillis(100))  // Artificial delay (simulation)
        .take(10);  // Limit to 10 results
}
```

### Server-Sent Events (SSE)
Stream data to client continuously:
```java
@GetMapping("/events")
public Flux<ServerSentEvent<User>> streamUsers() {
    return repo.findAll()
        .delayElements(Duration.ofSeconds(1))  // Emit 1 user per second
        .map(user -> ServerSentEvent.<User>builder()
            .data(user)
            .id(String.valueOf(user.getId()))
            .build());
}
```

### WebSocket (Bi-directional)
```java
@Component
public class WebSocketHandler implements org.springframework.web.reactive.socket.WebSocketHandler {
    @Override
    public Mono<Void> handle(WebSocketSession session) {
        return session.send(
            Mono.just(session.textMessage("Hello WebSocket!"))
        ).and(session.receive()
            .map(msg -> new String(msg.getPayloadAsText()))
            .doOnNext(text -> System.out.println("Received: " + text))
            .then()
        );
    }
}
```

---

## Reactive Data Access

### Spring Data Reactive Repositories

```java
public interface UserRepositoryReactive extends ReactiveCrudRepository<User, Long> {
    Mono<User> findByEmail(String email);
    Flux<User> findByAgeGreaterThan(int age);
}
```

### R2DBC (Relational Database Connectivity)
Non-blocking driver for SQL databases:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

### MongoDB Reactive
```java
public interface UserRepositoryMongo extends ReactiveMongoRepository<User, String> {
    Flux<User> findByCity(String city);
}

@RestController
public class UserController {
    private UserRepositoryMongo repo;
    
    @GetMapping("/users/city/{city}")
    public Flux<User> getUsersByCity(@PathVariable String city) {
        return repo.findByCity(city);
    }
}
```

---

## WebClient (Reactive HTTP Client)

Replace `RestTemplate` with `WebClient`:

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
}

@Service
public class ExternalApiService {
    private WebClient webClient;
    
    // GET request returning Mono
    public Mono<User> getUser(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatus::is4xxClientError, response -> 
                Mono.error(new UserNotFoundException("User not found")))
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .retry(3);
    }
    
    // GET request returning Flux
    public Flux<User> listUsers() {
        return webClient.get()
            .uri("/users")
            .retrieve()
            .bodyToFlux(User.class);
    }
    
    // POST request
    public Mono<User> createUser(User user) {
        return webClient.post()
            .uri("/users")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }
}
```

---

## Error Handling

### Operators
```java
// onErrorReturn: Fallback value
Flux.just(1, 2, 0)
    .map(x -> 10 / x)  // Division by zero
    .onErrorReturn(-1)  // Return -1 on any error
    .subscribe(System.out::println);

// onErrorResume: Fallback Mono/Flux
fetchFromPrimary()
    .onErrorResume(e -> fetchFromBackup());

// onErrorMap: Transform exception
fetchData()
    .onErrorMap(SocketTimeoutException.class, e -> 
        new ServiceUnavailableException("Service timeout", e));

// doOnError: Side effect (logging)
fetchData()
    .doOnError(Exception.class, e -> 
        logger.error("Error fetching data", e))
    .onErrorReturn(fallbackValue);

// timeout: Cancel if exceeds duration
fetchData()
    .timeout(Duration.ofSeconds(5))
    .onErrorResume(TimeoutException.class, e -> 
        Mono.just(cachedValue));
```

---

## Configuration

### application.yml
```yaml
spring:
  webflux:
    base-path: /api
    max-in-memory-upload-size: 16MB
  
  r2dbc:
    url: r2dbc:postgresql://localhost/mydb
    username: user
    password: pass
    pool:
      max-size: 10
      initial-size: 5

server:
  netty:
    connection-timeout: 5000
```

---

## WebFlux vs MVC

| Aspect | Spring MVC | Spring WebFlux |
|--------|-----------|-----------------|
| **Threading** | Blocking (thread pool) | Event loop (Netty) |
| **Threads** | 200-1000 threads | 100s of requests per thread |
| **Memory** | High (thread stack) | Low |
| **Database** | Traditional JDBC | R2DBC required |
| **Throughput** | Moderate (blocking) | High (non-blocking) |
| **Complexity** | Simpler | Steeper learning curve |
| **When to Use** | Standard apps, simple services | High-throughput, many concurrent users |
| **Legacy Support** | Excellent (established) | Growing (newer ecosystem) |

### When to Choose WebFlux
✅ **Use WebFlux if:**
- Handling 10,000+ concurrent connections
- Integrating with reactive databases (MongoDB, R2DBC)
- Building real-time features (WebSocket, SSE)
- Upstream service is reactive
- Team is comfortable with reactive paradigm

❌ **Stick with MVC if:**
- Traditional blocking database (JDBC)
- Team unfamiliar with reactive programming
- App doesn't have high concurrency needs
- Simpler to maintain and debug is priority

---

## Testing Reactive Code

### StepVerifier (Unit Testing)
```java
@Test
public void testFetchUser() {
    Mono<User> user = userService.getUser(1L);
    
    StepVerifier.create(user)
        .expectNext(new User(1L, "John"))
        .verifyComplete();
}

@Test
public void testListUsers() {
    Flux<User> users = userService.listUsers();
    
    StepVerifier.create(users)
        .expectNext(
            new User(1L, "John"),
            new User(2L, "Jane")
        )
        .verifyComplete();
}

@Test
public void testError() {
    Mono<User> user = userService.getUser(-1L);
    
    StepVerifier.create(user)
        .expectError(UserNotFoundException.class)
        .verify();
}
```

### Integration Testing with WebTestClient
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserControllerTest {
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    public void testGetUser() {
        webTestClient.get()
            .uri("/api/users/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody(User.class)
            .value(user -> assertThat(user.getName()).isEqualTo("John"));
    }
    
    @Test
    public void testListUsers() {
        webTestClient.get()
            .uri("/api/users")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(User.class)
            .hasSize(2);
    }
}
```

---

## Performance Considerations

### Backpressure
Handle when producer emits faster than consumer can handle:
```java
Flux.range(1, 1_000_000)
    .onBackpressureBuffer()  // Buffer excess
    // .onBackpressureLatest()  // Keep only latest
    // .onBackpressureDrop()    // Drop excess
    .subscribe(x -> process(x));
```

### Thread Pool Configuration
```yaml
spring:
  webflux:
    thread-pool-size: 10  # Default: number of CPU cores
```

### Monitoring
- Use Micrometer for metrics (latency, throughput)
- Monitor backpressure and error rates
- Profile for memory leaks in reactive chains

---

## Troubleshooting Common Issues

**Problem:** `BlockingOperationNotAllowedException`
- **Cause:** Calling blocking code (JDBC, Thread.sleep()) on reactive thread
- **Solution:** Use `subscribeOn(Schedulers.boundedElastic())` for blocking operations

**Problem:** Leaking Subscriptions
- **Cause:** Not disposing of subscription
- **Solution:** Use `Disposable` or `StepVerifier` in tests

**Problem:** Mono/Flux Not Executing
- **Cause:** Lazy evaluation (only runs on subscribe)
- **Solution:** Call `.subscribe()` or return from controller

---

**See also:** [Spring Boot Advanced](spring-boot-advanced.md), [Spring Threading & Async](spring-threading.md), [API Styles & Protocols](api-styles.md)

