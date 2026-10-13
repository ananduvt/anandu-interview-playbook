# Spring Boot Interview Questions

Spring Framework: dependency injection, Bean management, REST APIs, web development, and Spring ecosystem.

---

## Question Categories

### Dependency Injection & Beans
- IoC container, bean lifecycle
- Autowiring, qualifier disambiguation
- Scopes (singleton, prototype, request, session)
- Bean initialization/destruction hooks

### Spring Boot Auto-Configuration
- @SpringBootApplication, @Configuration
- Conditional beans, property binding
- Starter dependencies

### REST APIs & Web
- @RestController, @RequestMapping
- HTTP methods, status codes, content negotiation
- Exception handling, validation
- CORS, security

### Data Access & ORM
- Spring Data JPA, repositories
- Transaction management, propagation
- N+1 problem, lazy vs eager loading
- Query derivation

### Spring Security
- Authentication, authorization
- OAuth2, JWT, session management
- Spring Security configuration
- CORS integration

### Aspect-Oriented Programming (AOP)
- Aspects, pointcuts, advice
- @Around, @Before, @After
- Transaction management via AOP

### Testing
- Unit testing with JUnit, Mockito
- Integration testing with @SpringBootTest
- Mocking beans with @MockBean

---

## Common Questions & Answers

### Dependency Injection Types
**Q: What are the types of dependency injection and why use constructor injection?**

**Types:**
1. **Constructor Injection**: Pass dependencies via constructor
   - ✅ Immutable, all dependencies clear at construction
   - ✅ Testable, no hidden dependencies
   - ✅ Best for required dependencies
2. **Setter Injection**: Pass dependencies via setters
   - Allows partial initialization
   - Dependencies can change at runtime
3. **Field Injection**: `@Autowired` on fields
   - ❌ Hidden dependencies (not obvious in constructor)
   - ❌ Hard to test (need reflection or setter)
   - ❌ Can cause NPE if not initialized

**Best Practice: Constructor Injection**
```java
@Service
public class UserService {
    private final UserRepository repo;
    
    // Constructor injection - all dependencies visible
    public UserService(UserRepository repo) {
        this.repo = repo;
    }
}
```

---

### @Transactional Semantics
**Q: How does @Transactional work and what are its rollback rules?**

```java
@Transactional
public void transferMoney(Account from, Account to, BigDecimal amount) {
    from.debit(amount);
    to.credit(amount);
}
```

**Behavior:**
- Method runs in a transaction
- Commits on successful return
- **Rolls back on unchecked exceptions (RuntimeException, Error) by default**
- Does NOT roll back on checked exceptions (must specify `rollbackFor`)

**Rollback control:**
```java
@Transactional(rollbackFor = Exception.class)  // Roll back on any exception
@Transactional(noRollbackFor = ValidationException.class)  // Don't roll back on specific
```

**Propagation rules:**
- `REQUIRED` (default): Use existing or create new transaction
- `REQUIRES_NEW`: Always create new transaction (suspend current)
- `NESTED`: Create savepoint (can rollback child)
- `NOT_SUPPORTED`: Run without transaction

---

### Bean Scope & Lifecycle
**Q: Explain Spring bean scopes and their lifecycle.**

**Scopes:**
- **singleton** (default): One bean per Spring container
  - ✅ Efficient, stateless
  - ❌ Must be thread-safe
- **prototype**: New bean for each request
  - For stateful beans
  - `@Scope("prototype")`
- **request**: New bean per HTTP request (web only)
- **session**: Bean per HTTP session (web only)

**Lifecycle Hooks:**
```java
@Component
public class MyBean implements InitializingBean, DisposableBean {
    @PostConstruct
    public void init() { /* Called after construction */ }
    
    @PreDestroy
    public void cleanup() { /* Called before destruction */ }
    
    public void afterPropertiesSet() { /* InitializingBean */ }
    public void destroy() { /* DisposableBean */ }
}
```

**Order:** Constructor → @PostConstruct → Spring methods used → @PreDestroy

---

### Securing an API
**Q: How do you secure a Spring Boot REST API?**

**Approach: Spring Security with JWT**

1. **Authentication**: Validate username/password
2. **Token Generation**: Issue JWT token
3. **Authorization**: Check token on each request

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .antMatchers("/api/**").authenticated()
                .and()
            .addFilter(jwtAuthenticationFilter());
    }
}
```

**Layers:**
- Encrypt passwords (BCrypt)
- Use HTTPS only
- Validate JWT signature
- Set short token expiry
- Implement refresh tokens

---

### API Versioning
**Q: What are strategies for API versioning?**

**Approach 1: URL Path**
```
/api/v1/users  →  old version
/api/v2/users  →  new version
```

**Approach 2: Request Header**
```
GET /api/users
Header: API-Version: 1
```

**Approach 3: Content Negotiation**
```
GET /api/users
Accept: application/vnd.company.v1+json
```

**Best Practice:** URL path versioning is explicit and search-engine friendly.

---

### HTTP 202 Accepted for Async Operations
**Q: When and how to use HTTP 202 Accepted?**

Use 202 for **long-running operations** that are accepted but not yet completed.

```java
@PostMapping("/reports/generate")
public ResponseEntity<AsyncResponse> generateReport(@RequestBody ReportRequest req) {
    String jobId = taskService.submitAsync(req);
    return ResponseEntity.accepted()
        .location(URI.create("/api/reports/" + jobId))
        .body(new AsyncResponse(jobId));
}

@GetMapping("/reports/{jobId}")
public ResponseEntity<?> checkStatus(@PathVariable String jobId) {
    Job job = taskService.getJob(jobId);
    if (job.isComplete()) {
        return ResponseEntity.ok(job.getResult());  // 200 OK
    }
    return ResponseEntity.accepted().body(job.getProgress());  // 202 Accepted
}
```

**Flow:**
1. POST request → 202 Accepted + Location header
2. Client polls GET /reports/{jobId}
3. While processing: 202 Accepted
4. When done: 200 OK with result

---

## Sample Topics (To expand)

- Explain dependency injection and why it matters
- What's the difference between @Component, @Service, @Repository?
- How does Spring manage bean lifecycle?
- Explain @Autowired vs constructor injection
- What's the difference between singleton and prototype scope?
- How does Spring Data JPA repository derivation work?
- Explain the N+1 query problem and how to solve it
- What's the difference between @Transactional propagation types?
- Explain Spring Security authentication flow
- What's AOP and how does Spring implement it?
- How do you test Spring applications?
- What are Spring profiles and when to use them?
- Explain Spring Cloud circuit breaker patterns

---

[More content to be added progressively]

**See also:** [Spring Boot Fundamentals](../../spring-web/spring-boot.md), [Spring Annotations](../../spring-web/spring-annotations-components.md), [Interview Questions Overview](../index.md)
