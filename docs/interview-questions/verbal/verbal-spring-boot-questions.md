# Spring Boot Interview Questions

Spring Framework: dependency injection, Bean management, REST APIs, web development, and Spring ecosystem.

---

## 1. What are the types of dependency injection and why use constructor injection?

<details>
<summary>View Answer</summary>

<h3>Three Types of Dependency Injection</h3>

<h4>1. Constructor Injection ⭐ (Recommended)</h4>

<p><strong>Best for required dependencies.</strong></p>

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

<p><strong>Advantages:</strong></p>
<ul>
<li>✅ Immutable (final fields)</li>
<li>✅ All dependencies clear at construction</li>
<li>✅ Testable without reflection</li>
<li>✅ Fails fast if dependency missing</li>
</ul>

<h4>2. Setter Injection</h4>

<p><strong>For optional dependencies.</strong></p>

```java
@Service
public class UserService {
    private UserRepository repo;
    
    @Autowired
    public void setRepository(UserRepository repo) {
        this.repo = repo;
    }
}
```

<p><strong>Advantages:</strong> Allows partial initialization, dependencies can change at runtime</p>
<p><strong>Disadvantages:</strong> Less clear intent, mutable</p>

<h4>3. Field Injection ❌ (Avoid)</h4>

<p><strong>Discouraged in modern Spring.</strong></p>

```java
@Service
public class UserService {
    @Autowired
    private UserRepository repo;  // ❌ Hidden dependency
}
```

<p><strong>Disadvantages:</strong></p>
<ul>
<li>❌ Hidden dependencies (not obvious from constructor)</li>
<li>❌ Hard to test (need reflection or setter)</li>
<li>❌ Can cause NPE if not initialized</li>
<li>❌ Mutable</li>
</ul>

<h3>Why Constructor Injection is Best</h3>

<p>Constructor injection makes dependencies explicit and forces initialization of all required fields. It's also the only way to truly immutable beans.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot Fundamentals</a></p>

</details>

---

## 2. How does @Transactional work and what are its rollback rules?

<details>
<summary>View Answer</summary>

<h3>How @Transactional Works</h3>

<p>@Transactional wraps method execution in a database transaction.</p>

```java
@Transactional
public void transferMoney(Account from, Account to, BigDecimal amount) {
    from.debit(amount);      // Within transaction
    to.credit(amount);        // Within transaction
}
// Commits automatically on successful return
```

<h3>Default Behavior</h3>

<ul>
<li><strong>✅ Commits on successful return</strong></li>
<li><strong>❌ Rolls back on unchecked exceptions</strong> (RuntimeException, Error)</li>
<li><strong>Does NOT roll back on checked exceptions</strong> (must specify rollbackFor)</li>
</ul>

<h3>Controlling Rollback</h3>

```java
// Roll back on ANY exception (including checked)
@Transactional(rollbackFor = Exception.class)
public void process() { ... }

// Don't roll back on specific exception
@Transactional(noRollbackFor = ValidationException.class)
public void process() { ... }

// Specify multiple exceptions
@Transactional(rollbackFor = {IOException.class, SQLException.class})
public void process() { ... }
```

<h3>Propagation Strategies</h3>

<table>
<tr>
<th>Propagation</th>
<th>Behavior</th>
<th>Use Case</th>
</tr>
<tr>
<td><strong>REQUIRED</strong> (default)</td>
<td>Use existing or create new transaction</td>
<td>Most operations</td>
</tr>
<tr>
<td><strong>REQUIRES_NEW</strong></td>
<td>Always create new, suspend current</td>
<td>Independent sub-transactions</td>
</tr>
<tr>
<td><strong>NESTED</strong></td>
<td>Create savepoint, can rollback child</td>
<td>Partial rollback scenarios</td>
</tr>
<tr>
<td><strong>NOT_SUPPORTED</strong></td>
<td>Run without transaction</td>
<td>Read-only operations</td>
</tr>
<tr>
<td><strong>MANDATORY</strong></td>
<td>Must have existing transaction (error if none)</td>
<td>Enforcing transaction context</td>
</tr>
</table>

<h3>Key Point</h3>

<p><strong>Unchecked exceptions roll back by default; checked exceptions do not.</strong> This is often surprising and a common interview question!</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 3. Explain Spring bean scopes and their lifecycle

<details>
<summary>View Answer</summary>

<h3>Bean Scopes</h3>

<table>
<tr>
<th>Scope</th>
<th>Description</th>
<th>Use Case</th>
<th>Thread-Safe</th>
</tr>
<tr>
<td><strong>singleton</strong></td>
<td>One bean per Spring container (default)</td>
<td>Stateless services</td>
<td>❌ Must be thread-safe</td>
</tr>
<tr>
<td><strong>prototype</strong></td>
<td>New bean for each request</td>
<td>Stateful beans</td>
<td>✅ Yes (one per request)</td>
</tr>
<tr>
<td><strong>request</strong></td>
<td>New bean per HTTP request (web only)</td>
<td>Request-scoped data</td>
<td>✅ Yes</td>
</tr>
<tr>
<td><strong>session</strong></td>
<td>Bean per HTTP session (web only)</td>
<td>User session data</td>
<td>✅ Yes</td>
</tr>
</table>

<h3>Bean Lifecycle Hooks</h3>

```java
@Component
public class MyBean implements InitializingBean, DisposableBean {
    
    @PostConstruct
    public void init() { 
        /* Called after construction & dependency injection */ 
    }
    
    @PreDestroy
    public void cleanup() { 
        /* Called before bean destruction */ 
    }
    
    @Override
    public void afterPropertiesSet() { 
        /* InitializingBean interface */ 
    }
    
    @Override
    public void destroy() { 
        /* DisposableBean interface */ 
    }
}
```

<h3>Lifecycle Order</h3>

<ol>
<li><strong>Constructor</strong> - Object created</li>
<li><strong>Setter methods</strong> - Dependencies injected</li>
<li><strong>@PostConstruct</strong> - Custom initialization</li>
<li><strong>afterPropertiesSet()</strong> - InitializingBean (if implemented)</li>
<li><strong>Bean is ready for use</strong></li>
<li><strong>@PreDestroy</strong> - Custom cleanup</li>
<li><strong>destroy()</strong> - DisposableBean (if implemented)</li>
</ol>

<h3>Example: Database Connection Pool</h3>

```java
@Component
public class DataSourceBean {
    private DataSource ds;
    
    @PostConstruct
    public void init() {
        // Initialize connection pool
        ds = createConnectionPool();
    }
    
    @PreDestroy
    public void cleanup() {
        // Close all connections
        ds.close();
    }
}
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 4. What's the difference between @Component and @Bean?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr>
<th>Aspect</th>
<th>@Component</th>
<th>@Bean</th>
</tr>
<tr>
<td><strong>Annotation Type</strong></td>
<td>Class-level</td>
<td>Method-level</td>
</tr>
<tr>
<td><strong>Where</strong></td>
<td>On class definition</td>
<td>In @Configuration class</td>
</tr>
<tr>
<td><strong>Discovery</strong></td>
<td>Auto-discovered by component scanning</td>
<td>Explicitly defined in configuration</td>
</tr>
<tr>
<td><strong>Use Case</strong></td>
<td>Your own classes</td>
<td>Third-party libraries or complex init</td>
</tr>
<tr>
<td><strong>Flexibility</strong></td>
<td>Limited - class must exist</td>
<td>High - any object can be bean</td>
</tr>
</table>

<h3>@Component Example</h3>

```java
@Component  // Auto-discovered
public class UserService {
    // Bean automatically created by Spring
}
```

<h3>@Bean Example</h3>

```java
@Configuration
public class AppConfig {
    
    @Bean  // Explicitly define bean
    public UserService userService(UserRepository repo) {
        // Complex initialization logic
        UserService service = new UserService(repo);
        service.configure();
        return service;
    }
    
    // For third-party libraries
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

<h3>When to Use</h3>

<ul>
<li><strong>@Component:</strong> Your own classes, simple initialization</li>
<li><strong>@Bean:</strong> Third-party libraries, complex initialization, conditional beans</li>
</ul>

<h3>Stereotypes (Specializations of @Component)</h3>

```java
@Service        // Business logic
@Repository     // Data access
@Controller     // Web controllers
@RestController // REST endpoints
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations</a></p>

</details>

---

## 5. How do you secure a Spring Boot REST API?

<details>
<summary>View Answer</summary>

<h3>Approach: Spring Security with JWT</h3>

<h4>Step 1: Authentication (Login)</h4>

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    // Validate username/password
    authenticate(req.getUsername(), req.getPassword());
    
    // Generate JWT token
    String token = jwtProvider.generateToken(req.getUsername());
    
    return ResponseEntity.ok(new JwtResponse(token));
}
```

<h4>Step 2: Authorization (Protected Resources)</h4>

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll()  // Public
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .antMatchers("/api/**").authenticated()    // Protected
                .and()
            .addFilter(new JwtAuthenticationFilter());
        
        return http.build();
    }
}
```

<h4>Step 3: Security Layers</h4>

<ul>
<li><strong>Password encryption:</strong> Use BCrypt (not plain text)</li>
<li><strong>HTTPS only:</strong> Encrypt in transit</li>
<li><strong>JWT validation:</strong> Verify signature on each request</li>
<li><strong>Short expiry:</strong> Tokens expire in 15-30 minutes</li>
<li><strong>Refresh tokens:</strong> Allow generating new access tokens</li>
<li><strong>CORS:</strong> Restrict cross-origin requests</li>
</ul>

<h3>JWT Token Structure</h3>

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.  // Header (algorithm)
eyJzdWIiOiIxMjM0NTY3ODkwIn0.              // Payload (claims)
dozjgNryP4J3jVmNHl0w5N_XgL0n3I9PlFUP0THsR8A  // Signature
```

<p><strong>Header + Payload + Signature</strong> (signed with secret)</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/security/">Learn more: Security & Authentication</a></p>

</details>

---

## 6. What are strategies for API versioning?

<details>
<summary>View Answer</summary>

<h3>Strategy 1: URL Path Versioning ⭐ (Recommended)</h3>

<p><strong>Version in the URL path</strong></p>

```
/api/v1/users  →  Old version
/api/v2/users  →  New version
```

<p><strong>Advantages:</strong> Explicit, search-engine friendly, easy to understand</p>
<p><strong>Implementation:</strong></p>

```java
@GetMapping("/api/v1/users")
public List<UserV1> getUsersV1() { ... }

@GetMapping("/api/v2/users")
public List<UserV2> getUsersV2() { ... }
```

<h3>Strategy 2: Request Header</h3>

<p><strong>Version specified in HTTP header</strong></p>

```
GET /api/users
Header: API-Version: 1
```

<p><strong>Advantages:</strong> Clean URL, multiple versions via same endpoint</p>
<p><strong>Disadvantages:</strong> Not visible in URL, harder to test</p>

<h3>Strategy 3: Content Negotiation</h3>

<p><strong>Version in Accept header with media type</strong></p>

```
GET /api/users
Accept: application/vnd.company.v1+json
```

<p><strong>Advantages:</strong> RESTful, semantic</p>
<p><strong>Disadvantages:</strong> Complex, less common</p>

<h3>Strategy 4: Query Parameter</h3>

<p><strong>Version as query parameter</strong></p>

```
GET /api/users?version=1
GET /api/users?v=2
```

<p><strong>Disadvantages:</strong> Easy to forget, caching issues</p>

<h3>Recommendation</h3>

<p><strong>Use URL path versioning</strong> for clarity and simplicity. It's the most straightforward and widely understood approach.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles</a></p>

</details>

---

## 7. When and how to use HTTP 202 Accepted for async operations?

<details>
<summary>View Answer</summary>

<h3>What is HTTP 202?</h3>

<p><strong>HTTP 202 Accepted:</strong> Request accepted for processing, but not yet complete.</p>

<p>Used for <strong>long-running operations</strong> that you don't want to block on.</p>

<h3>Use Case</h3>

```
Client: POST /api/reports/generate (long-running job)
Server: 202 Accepted + Location header + job ID
Client: Poll GET /api/reports/{jobId} periodically
Server: 202 (still processing) or 200 (complete)
```

<h3>Implementation</h3>

```java
@PostMapping("/reports/generate")
public ResponseEntity<AsyncResponse> generateReport(@RequestBody ReportRequest req) {
    // Submit async task
    String jobId = taskService.submitAsync(req);
    
    // Return 202 Accepted + Location header
    return ResponseEntity.accepted()
        .location(URI.create("/api/reports/" + jobId))
        .body(new AsyncResponse(jobId, "Processing"));
}

@GetMapping("/reports/{jobId}")
public ResponseEntity<?> checkStatus(@PathVariable String jobId) {
    Job job = taskService.getJob(jobId);
    
    if (job.isComplete()) {
        return ResponseEntity.ok(job.getResult());  // 200 OK
    }
    
    if (job.isProcessing()) {
        return ResponseEntity.accepted()
            .body(new StatusResponse("Processing", job.getProgress()));  // 202
    }
    
    if (job.isFailed()) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(job.getError());  // 409 Conflict
    }
    
    return ResponseEntity.notFound().build();  // 404
}
```

<h3>Complete Flow</h3>

<ol>
<li><strong>Client POST:</strong> Request long-running operation</li>
<li><strong>Server returns 202:</strong> Job accepted, returns Location header</li>
<li><strong>Client polls GET:</strong> Check status at Location URL</li>
<li><strong>While processing:</strong> 202 Accepted with progress</li>
<li><strong>When complete:</strong> 200 OK with result</li>
</ol>

<h3>Alternatives to Polling</h3>

<ul>
<li><strong>Webhooks:</strong> Server calls client when done</li>
<li><strong>WebSocket:</strong> Real-time status updates</li>
<li><strong>Server-Sent Events (SSE):</strong> One-way server push</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles & REST Design</a></p>

</details>

---

## 8. What are the key differences between SOAP and REST?

<details>
<summary>View Answer</summary>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Aspect</th>
<th>SOAP</th>
<th>REST</th>
</tr>
<tr>
<td><strong>Protocol</strong></td>
<td>XML-based messaging</td>
<td>HTTP methods on resources</td>
</tr>
<tr>
<td><strong>Message Format</strong></td>
<td>XML (verbose)</td>
<td>JSON/XML (lightweight)</td>
</tr>
<tr>
<td><strong>HTTP Methods</strong></td>
<td>Usually POST for everything</td>
<td>GET, POST, PUT, DELETE (semantics)</td>
</tr>
<tr>
<td><strong>Stateless</strong></td>
<td>Can be stateful</td>
<td>Always stateless</td>
</tr>
<tr>
<td><strong>Caching</strong></td>
<td>Difficult</td>
<td>Built-in (HTTP caching)</td>
</tr>
<tr>
<td><strong>Learning Curve</strong></td>
<td>Steep (WS-* specs)</td>
<td>Easy (HTTP fundamentals)</td>
</tr>
<tr>
<td><strong>Best For</strong></td>
<td>Enterprise, financial systems</td>
<td>Web APIs, mobile, modern apps</td>
</tr>
</table>

<h3>SOAP Example</h3>

```xml
POST /webservice HTTP/1.1
Content-Type: text/xml

<?xml version="1.0"?>
<soap:Envelope xmlns:soap="...">
  <soap:Body>
    <GetUser>
      <userId>123</userId>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

<h3>REST Example</h3>

```
GET /api/users/123 HTTP/1.1
Accept: application/json

Response:
{
  "id": 123,
  "name": "John Doe"
}
```

<h3>When to Choose</h3>

<ul>
<li><strong>Use SOAP:</strong> Legacy enterprise systems, strict contracts, WS-* security requirements</li>
<li><strong>Use REST:</strong> New APIs, web/mobile apps, microservices, public APIs</li>
</ul>

<h3>Key Difference: Method Semantics</h3>

<p><strong>SOAP:</strong> Everything is a method call (GetUser, CreateUser, DeleteUser)</p>
<p><strong>REST:</strong> Resources with HTTP methods (GET /users/123, POST /users, DELETE /users/123)</p>

<p>📚 <a href="/anandu-interview-playbook/foundations/web-fundamentals/">Learn more: Web Fundamentals</a></p>

</details>

---

## 9. What's the difference between HTTP PUT and PATCH?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr><th>Aspect</th><th>PUT</th><th>PATCH</th></tr>
<tr><td><strong>Semantics</strong></td><td>Replace entire resource</td><td>Partial update</td></tr>
<tr><td><strong>Idempotent</strong></td><td>✅ Yes (same = same state)</td><td>✅ Yes (usually)</td></tr>
<tr><td><strong>Body</strong></td><td>Complete resource</td><td>Only changed fields</td></tr>
<tr><td><strong>Missing fields</strong></td><td>❌ Set to null (dangerous)</td><td>✅ Unchanged</td></tr>
<tr><td><strong>Use Case</strong></td><td>Full replacements</td><td>Updates (most common)</td></tr>
</table>

<h3>Example Scenarios</h3>

```
Current user: {"id": 1, "name": "John", "email": "john@old.com", "role": "USER"}

PUT /users/1
{
  "name": "John",
  "email": "john@new.com",
  "role": "USER"
}
→ Replaces all fields. If "role" is missing, set to null (likely error!)

PATCH /users/1
{
  "email": "john@new.com"
}
→ Updates only email. Name and role unchanged.
```

<h3>Implementation</h3>

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // PUT: Replace entire resource
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUserFull(@PathVariable Long id, @RequestBody User user) {
        // Validate: all required fields must be present
        if (user.getName() == null || user.getEmail() == null) {
            return ResponseEntity.badRequest().build();
        }
        
        // Replace entire user
        User updated = userService.replaceUser(id, user);
        return ResponseEntity.ok(updated);
    }
    
    // PATCH: Partial update
    @PatchMapping("/{id}")
    public ResponseEntity<User> updateUserPartial(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        // Only update provided fields
        User updated = userService.partialUpdate(id, updates);
        return ResponseEntity.ok(updated);
    }
}
```

<h3>Best Practice</h3>

<p><strong>Use PATCH for most updates.</strong> PUT is rarely used because replacing entire resources is risky—clients often forget to include all fields, causing accidental data loss.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles & REST Design</a></p>

</details>

---

## 10. Explain Hibernate's persist(), save(), and merge() lifecycle

<details>
<summary>View Answer</summary>

<h3>Entity State Transitions</h3>

```
Transient → [persist/save] → Persistent → [detach] → Detached → [merge] → Persistent
```

<h3>Core Differences</h3>

<table>
<tr><th>Method</th><th>Input State</th><th>Output State</th><th>DB Effect</th><th>Use Case</th></tr>
<tr><td><strong>persist()</strong></td><td>Transient</td><td>Persistent</td><td>INSERT scheduled</td><td>New entities only</td></tr>
<tr><td><strong>save()</strong></td><td>Transient/Detached</td><td>Persistent</td><td>INSERT or UPDATE</td><td>General save (Spring Data)</td></tr>
<tr><td><strong>merge()</strong></td><td>Detached</td><td>Persistent (new instance)</td><td>UPDATE (if exists)</td><td>Re-attach modified detached entity</td></tr>
</table>

<h3>persist() — New Entity, Guaranteed INSERT</h3>

```java
User user = new User("John", "john@example.com");  // Transient
em.persist(user);  // Now Persistent; INSERT queued

// ID is generated at flush/commit
System.out.println(user.getId());  // ✅ Available after commit
```

<h3>save() — Auto-Detect INSERT vs UPDATE</h3>

```java
User user = new User("Jane", "jane@example.com");
User saved = userRepository.save(user);

// Spring Data's save():
// - ID null → persist() → INSERT
// - ID exists → merge() → UPDATE
```

<h3>merge() — Re-Attach Detached Entity (Critical!)</h3>

```java
// Detached entity (loaded in previous session)
User user = em.find(User.class, 1L);  // Persistent
em.detach(user);  // Now DETACHED

user.setName("Updated Name");  // ❌ Changes NOT tracked!

// Re-attach
User managed = em.merge(user);  // Returns NEW managed instance

// user != managed (different objects!)
// managed.setEmail("new@email.com");  // ✅ Changes tracked
```

<h3>Key Insight: Dirty Checking</h3>

```java
@Transactional
public void updateUser(Long id, String newName) {
    User user = userRepository.findById(id).get();  // Persistent
    
    user.setName(newName);
    
    // ❌ DON'T need to call save()!
    // ✅ Dirty checking at transaction commit:
    // 1. Hibernate compares current state to snapshot
    // 2. Detects changes
    // 3. Auto-generates UPDATE SQL
    
    // No @Transactional = no dirty check = changes lost!
}
```

<h3>Common Mistake: Forgetting @Transactional</h3>

```java
// ❌ WRONG: No @Transactional
public void updateUser(Long id, String name) {
    User user = userRepository.findById(id).get();
    user.setName(name);  // Changes silently lost!
}

// ✅ RIGHT: With @Transactional
@Transactional
public void updateUser(Long id, String name) {
    User user = userRepository.findById(id).get();
    user.setName(name);  // Tracked automatically
}
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 11. When should you choose Hibernate/JPA over MyBatis and vice versa?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr><th>Aspect</th><th>JPA/Hibernate</th><th>MyBatis</th></tr>
<tr><td><strong>Paradigm</strong></td><td>ORM (Object-Relational Mapping)</td><td>SQL Mapper</td></tr>
<tr><td><strong>SQL Writing</strong></td><td>Automatic (from objects)</td><td>Manual (full SQL control)</td></tr>
<tr><td><strong>Relationships</strong></td><td>Built-in (@OneToMany, @ManyToMany)</td><td>Manual (complex joins)</td></tr>
<tr><td><strong>Boilerplate</strong></td><td>Less (auto CRUD)</td><td>More (write mappings)</td></tr>
<tr><td><strong>Performance</strong></td><td>Good (caching, lazy loading)</td><td>Very fast (pure SQL, no overhead)</td></tr>
<tr><td><strong>Flexibility</strong></td><td>Less (constrained by ORM)</td><td>High (full SQL control)</td></tr>
<tr><td><strong>Learning Curve</strong></td><td>Moderate (lazy, cascade, context)</td><td>Low (just SQL + mappings)</td></tr>
<tr><td><strong>Database Agnostic</strong></td><td>Yes (works across DBs)</td><td>Requires dialect-specific SQL</td></tr>
</table>

<h3>Use Hibernate/JPA When...</h3>

<ul>
<li><strong>✅ Domain-model-driven design:</strong> Building OOP-heavy applications with entities</li>
<li><strong>✅ Standard CRUD:</strong> Simple insert/update/delete operations</li>
<li><strong>✅ Relationships matter:</strong> Complex entity relationships (1-to-many, many-to-many)</li>
<li><strong>✅ Rapid development:</strong> Less boilerplate, faster to prototype</li>
<li><strong>✅ Database portability:</strong> Need to support multiple databases</li>
<li><strong>✅ Automatic caching:</strong> Leverage first and second-level caches</li>
<li><strong>✅ Inheritance hierarchies:</strong> Need polymorphic queries</li>
</ul>

```java
// JPA: Minimal boilerplate, automatic relationships
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;  // Automatic handling
}

User user = userRepository.findById(1L).get();
List<Order> orders = user.getOrders();  // Lazy-loaded automatically
```

<h3>Use MyBatis When...</h3>

<ul>
<li><strong>✅ Complex/optimized queries:</strong> Need fine-tuned SQL for performance</li>
<li><strong>✅ Legacy databases:</strong> Odd schemas that don't map well to entities</li>
<li><strong>✅ Bulk operations:</strong> Mass inserts/updates where ORM overhead matters</li>
<li><strong>✅ Stored procedures:</strong> Heavy reliance on DB-specific features</li>
<li><strong>✅ Team SQL expertise:</strong> Team comfortable writing/optimizing SQL</li>
<li><strong>✅ Predictable performance:</strong> Need exact control over DB access</li>
<li><strong>✅ Multiple data sources:</strong> Querying multiple databases with dialect-specific SQL</li>
</ul>

```java
// MyBatis: Full SQL control, explicit mappings
@Mapper
public interface OrderMapper {
    @Select("SELECT o.* FROM orders o " +
            "JOIN users u ON o.user_id = u.id " +
            "WHERE u.id = #{userId} AND o.amount > #{minAmount}")
    List<Order> findHighValueOrdersByUser(@Param("userId") Long userId, 
                                          @Param("minAmount") BigDecimal amount);
}

List<Order> orders = orderMapper.findHighValueOrdersByUser(1L, new BigDecimal("100.00"));
```

<h3>Real-World Scenarios</h3>

| Scenario | Choose |
|----------|--------|
| Building microservice with standard CRUD endpoints | Hibernate |
| E-commerce app with complex product filtering & reporting | MyBatis |
| Data warehouse or analytics pipeline with bulk loads | MyBatis |
| SaaS application with multi-tenant design | Hibernate |
| High-frequency trading system with performance-critical queries | MyBatis |
| Traditional 3-tier app with OOP domain model | Hibernate |

<h3>Hybrid Approach</h3>

```java
// Use both: Hibernate for standard operations, MyBatis for complex queries
@Service
public class OrderService {
    private OrderRepository orderRepository;  // Hibernate
    private OrderQueryMapper orderQueryMapper;  // MyBatis
    
    public Order getSimpleOrder(Long id) {
        // Simple: Use Hibernate
        return orderRepository.findById(id).orElse(null);
    }
    
    public List<OrderAnalytics> getComplexAnalytics() {
        // Complex: Use MyBatis for optimized SQL
        return orderQueryMapper.analyzeOrderTrends();
    }
}
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM vs MyBatis</a></p>

</details>

---

## 12. What happens when SpringApplication.run() is called?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>SpringApplication.run()</code> builds the environment, creates and refreshes the <code>ApplicationContext</code> (the IoC container), starts the embedded web server, and finally runs <code>CommandLineRunner</code>/<code>ApplicationRunner</code> beans before the app is "ready".</p>

<h3>Startup Steps</h3>
<ol>
<li><strong>Bootstrap:</strong> create the <code>SpringApplication</code>, detect app type (servlet, reactive, none), load <code>ApplicationContextInitializer</code>s and listeners from <code>META-INF/spring</code>.</li>
<li><strong>Environment:</strong> load <code>application.yml/properties</code>, env vars, command-line args, activate profiles.</li>
<li><strong>Create context:</strong> e.g. <code>AnnotationConfigServletWebServerApplicationContext</code>; print banner.</li>
<li><strong>Prepare:</strong> register the main class as a configuration source.</li>
<li><strong>Refresh:</strong> component scan, process auto-configuration imports, register bean definitions, apply <code>BeanFactoryPostProcessor</code>s, instantiate singletons (running <code>BeanPostProcessor</code>s, <code>@PostConstruct</code>), start embedded Tomcat/Jetty/Netty.</li>
<li><strong>After refresh:</strong> publish <code>ApplicationStartedEvent</code>, call runners, then <code>ApplicationReadyEvent</code> (AvailabilityState becomes ready).</li>
</ol>

<h3>Example: Code Running After Startup</h3>

```java
@Component
public class CacheWarmer implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) {
        // runs once after the context is refreshed, before ApplicationReadyEvent
    }
}
```

<h3>Gotchas / Follow-ups</h3>
<ul>
<li>An exception in a runner fails the startup; keep them fast or move work to an <code>ApplicationReadyEvent</code> listener.</li>
<li>Singletons are eager by default, so misconfigured beans fail at startup (fail fast), not on first request.</li>
<li><code>spring.main.lazy-initialization=true</code> speeds startup but moves failures to runtime.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 13. What is the purpose of @Scheduled in Spring Boot?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@Scheduled</code> runs a method periodically (fixed rate, fixed delay, or cron). It needs <code>@EnableScheduling</code> on a configuration class, and the method must be <code>void</code> with no arguments.</p>

<h3>Options</h3>
<table>
<tr><th>Attribute</th><th>Meaning</th></tr>
<tr><td><code>fixedRate</code></td><td>Start every N ms regardless of previous run's duration</td></tr>
<tr><td><code>fixedDelay</code></td><td>Wait N ms after the previous run finishes</td></tr>
<tr><td><code>initialDelay</code></td><td>Delay before first execution</td></tr>
<tr><td><code>cron</code></td><td>Six fields: second minute hour day month weekday; optional <code>zone</code></td></tr>
</table>

```java
@Configuration
@EnableScheduling
class SchedulingConfig {}

@Component
class ReportJob {
    @Scheduled(cron = "0 0 2 * * *", zone = "UTC")   // every day at 02:00 UTC
    void nightlyReport() { /* ... */ }

    @Scheduled(fixedDelayString = "${jobs.cleanup.delay-ms:60000}")
    void cleanup() { /* ... */ }
}
```

<h3>Gotchas</h3>
<ul>
<li>The default scheduler has <strong>one thread</strong>, so a slow job delays others; set <code>spring.task.scheduling.pool.size</code>.</li>
<li>In a cluster every instance runs the job. Use a distributed lock (e.g. ShedLock) or Quartz with a JDBC store to run it once.</li>
<li>Exceptions are logged and the schedule continues; handle errors explicitly.</li>
<li>Do not confuse with <code>@Async</code>: scheduling decides <em>when</em>, async decides <em>which thread</em>.</li>
</ul>

<h3>When to Use</h3>
<p>Cleanup, report generation, polling, cache refresh. For durable, dynamic, or clustered jobs prefer Quartz or an external scheduler.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 14. What is the purpose of @Value and @Configuration?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@Configuration</code> marks a class that declares beans via <code>@Bean</code> methods (Java-based config). <code>@Value</code> injects a single property or SpEL expression into a field, parameter, or constructor argument.</p>

```java
@Configuration
public class AppConfig {
    @Bean
    public RestClient paymentClient(@Value("${payment.base-url}") String baseUrl) {
        return RestClient.builder().baseUrl(baseUrl).build();
    }
}

@Service
class Mailer {
    private final int retries;
    Mailer(@Value("${mail.retries:3}") int retries) { this.retries = retries; } // default 3
}
```

<h3>Key Points</h3>
<ul>
<li><code>${...}</code> is a property placeholder (supports defaults after <code>:</code>); <code>#{...}</code> is SpEL.</li>
<li><code>@Configuration</code> classes are CGLIB-proxied, so calling one <code>@Bean</code> method from another returns the same singleton. With <code>proxyBeanMethods = false</code> (or <code>@Component</code>), you get plain method calls and a new instance each time.</li>
<li>A missing placeholder without a default fails startup.</li>
</ul>

<h3>@Value vs @ConfigurationProperties</h3>
<table>
<tr><th></th><th>@Value</th><th>@ConfigurationProperties</th></tr>
<tr><td>Best for</td><td>One or two values</td><td>Groups of related properties</td></tr>
<tr><td>Binding</td><td>Exact key, SpEL allowed</td><td>Relaxed binding, nested objects, lists</td></tr>
<tr><td>Validation</td><td>None built in</td><td>Works with <code>@Validated</code></td></tr>
</table>

<h3>When to Use</h3>
<p>Prefer <code>@ConfigurationProperties</code> for structured config; keep <code>@Value</code> for simple cases. Never put secrets in plain property files.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 15. What are the commonly used Spring Boot annotations and their purposes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@SpringBootApplication</code> bootstraps the app; stereotypes (<code>@Component</code>, <code>@Service</code>, <code>@Repository</code>, <code>@Controller</code>/<code>@RestController</code>) register beans; <code>@Autowired</code>, <code>@Value</code>, <code>@Bean</code> wire and configure them; web annotations map requests.</p>

<table>
<tr><th>Annotation</th><th>Purpose</th></tr>
<tr><td><code>@SpringBootApplication</code></td><td>= <code>@Configuration</code> + <code>@EnableAutoConfiguration</code> + <code>@ComponentScan</code></td></tr>
<tr><td><code>@Component</code></td><td>Generic managed bean</td></tr>
<tr><td><code>@Service</code></td><td>Business logic (semantic marker)</td></tr>
<tr><td><code>@Repository</code></td><td>Data access; enables persistence exception translation</td></tr>
<tr><td><code>@RestController</code></td><td><code>@Controller</code> + <code>@ResponseBody</code>; returns JSON</td></tr>
<tr><td><code>@RequestMapping</code>, <code>@GetMapping</code>, ...</td><td>Map HTTP requests to handlers</td></tr>
<tr><td><code>@PathVariable</code>, <code>@RequestParam</code>, <code>@RequestBody</code></td><td>Bind request data</td></tr>
<tr><td><code>@Bean</code> / <code>@Configuration</code></td><td>Declare beans in Java config</td></tr>
<tr><td><code>@Value</code>, <code>@ConfigurationProperties</code></td><td>Externalised configuration</td></tr>
<tr><td><code>@Transactional</code></td><td>Declarative transactions</td></tr>
<tr><td><code>@ConditionalOnProperty</code>, <code>@Profile</code></td><td>Conditional bean registration</td></tr>
<tr><td><code>@ControllerAdvice</code>, <code>@ExceptionHandler</code></td><td>Central exception handling</td></tr>
</table>

<h3>Follow-ups</h3>
<ul>
<li>Which annotation inside <code>@SpringBootApplication</code> triggers scanning? <code>@ComponentScan</code>, starting at the main class's package.</li>
<li>Prefer constructor injection; <code>@Autowired</code> is optional on a single constructor.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations & Components</a></p>

</details>

---

## 16. What does @EnableAutoConfiguration do?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>It tells Spring Boot to register beans automatically based on what is on the classpath, existing beans, and properties. It imports the auto-configuration classes listed in <code>META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports</code> (Spring Boot 2.7+/3; earlier versions used <code>spring.factories</code>).</p>

<h3>How It Works</h3>
<ul>
<li>Each auto-configuration class is guarded by conditions such as <code>@ConditionalOnClass</code>, <code>@ConditionalOnMissingBean</code>, <code>@ConditionalOnProperty</code>.</li>
<li>Example: with <code>spring-boot-starter-web</code> on the classpath and no <code>DataSource</code> beans defined, Boot configures Tomcat and Spring MVC; with H2 plus JPA it configures an embedded <code>DataSource</code>.</li>
<li><code>@ConditionalOnMissingBean</code> makes auto-config <strong>back off</strong> when you define your own bean.</li>
</ul>

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class App { }
```

<h3>Debugging</h3>
<ul>
<li>Run with <code>--debug</code> or <code>debug=true</code> to print the conditions evaluation report (positive and negative matches).</li>
<li>Actuator <code>/actuator/conditions</code> shows the same data.</li>
<li>Exclude via <code>exclude</code> or <code>spring.autoconfigure.exclude</code>.</li>
</ul>

<h3>Follow-up</h3>
<p>Custom starter: write an <code>@AutoConfiguration</code> class with conditions and list it in the <code>.imports</code> file.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 17. What is a Spring Profile and how is it used?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A profile is a named group of configuration and beans that is active only in certain environments (dev, test, prod). It lets one artifact run with different settings.</p>

<h3>Usage</h3>
<ul>
<li><strong>Activate:</strong> <code>spring.profiles.active=prod</code>, env var <code>SPRING_PROFILES_ACTIVE</code>, or <code>--spring.profiles.active=prod</code>.</li>
<li><strong>Profile files:</strong> <code>application-dev.yml</code> overrides values in <code>application.yml</code>.</li>
<li><strong>Beans:</strong> <code>@Profile("dev")</code> on a <code>@Component</code> or <code>@Bean</code> (supports <code>!prod</code> and expressions).</li>
<li><strong>Groups (Boot 2.4+):</strong> <code>spring.profiles.group.prod=proddb,prodmq</code>.</li>
<li><strong>Tests:</strong> <code>@ActiveProfiles("test")</code>.</li>
</ul>

```java
@Configuration
class MailConfig {
    @Bean @Profile("prod")  MailSender smtpSender() { return new SmtpSender(); }
    @Bean @Profile("!prod") MailSender logSender()  { return new LoggingSender(); }
}
```

<h3>Gotchas</h3>
<ul>
<li>Command-line and env values override file values; profile-specific files override the base file.</li>
<li>Do not store production secrets in <code>application-prod.yml</code> in the repo; use env vars or a secrets manager.</li>
<li>Too many profiles make behaviour hard to reason about; prefer externalised config over profile-per-customer.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-advanced/">Learn more: Spring Boot Advanced</a></p>

</details>

---

## 18. How do you increase memory and CPU resources for a Spring Boot application?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Spring Boot does not manage resources itself. You size the JVM and the container/host, then tune thread pools and connection pools to match.</p>

<h3>Memory</h3>
<ul>
<li>Heap: <code>-Xms</code>/<code>-Xmx</code>, or in containers <code>-XX:MaxRAMPercentage=75</code> (container-aware since Java 10; use a modern LTS such as 17, 21, or 25).</li>
<li>Remember non-heap: metaspace, thread stacks, direct buffers, code cache. Heap should not equal container limit.</li>
<li>Set it via <code>JAVA_TOOL_OPTIONS</code> or <code>java -jar app.jar</code> flags; choose GC (G1 default, ZGC for low latency).</li>
</ul>

<h3>CPU</h3>
<ul>
<li>Raise cores on the host or container (<code>resources.limits.cpu</code> in Kubernetes). The JVM sizes GC and common pool threads from visible CPUs.</li>
<li>Tune <code>server.tomcat.threads.max</code> (default 200), the HikariCP <code>maximum-pool-size</code>, and async/scheduler executors.</li>
<li>Virtual threads (Java 21+, <code>spring.threads.virtual.enabled=true</code> in Boot 3.2+) help I/O-bound workloads.</li>
</ul>

<h3>Process</h3>
<ol>
<li>Measure first: Actuator metrics, GC logs, profiler (JFR).</li>
<li>Change one thing at a time and load test.</li>
<li>Consider horizontal scaling before giant heaps (long GC pauses).</li>
</ol>

<p>📚 <a href="/anandu-interview-playbook/java-core/jvm-internals/">Learn more: JVM Internals</a></p>

</details>

---

## 19. How would you use Spring Boot to create and deploy microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Each service is a small Spring Boot app that owns its data, exposes a REST/messaging API, runs on an embedded server as a single executable jar, and is packaged as a container image and deployed by CI/CD.</p>

<h3>Build</h3>
<ul>
<li>Starters + auto-configuration for web, data, security, actuator.</li>
<li>Externalised config (profiles, env vars, Config Server or Kubernetes ConfigMaps/Secrets).</li>
<li>Service-to-service calls with <code>RestClient</code>/<code>WebClient</code> or OpenFeign; resilience with Resilience4j (timeouts, retry, circuit breaker).</li>
<li>Service discovery and gateway via Spring Cloud, or leave both to Kubernetes and an ingress/gateway.</li>
<li>Observability: Actuator health probes, Micrometer metrics, distributed tracing, structured logs.</li>
</ul>

<h3>Deploy</h3>
<ol>
<li>Build image: <code>mvn spring-boot:build-image</code> (Buildpacks) or a multi-stage Dockerfile.</li>
<li>CI pipeline: test, scan, build, push image.</li>
<li>Kubernetes Deployment with liveness/readiness probes (<code>/actuator/health/liveness</code>, <code>/readiness</code>), resource limits, rolling updates.</li>
</ol>

<h3>Pitfalls</h3>
<ul>
<li>Shared database between services destroys independence.</li>
<li>Distributed transactions: use sagas/outbox rather than 2PC.</li>
<li>Do not start with microservices if a modular monolith meets the need.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/spring-cloud/">Learn more: Spring Cloud & Microservices</a></p>

</details>

---

## 20. How do you test Spring applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Follow the test pyramid: many fast unit tests without Spring, fewer slice tests that load part of the context, and a few full integration tests.</p>

<table>
<tr><th>Level</th><th>Tool</th><th>Notes</th></tr>
<tr><td>Unit</td><td>JUnit 5 + Mockito (<code>@ExtendWith(MockitoExtension.class)</code>)</td><td>No Spring context; constructor injection makes this easy</td></tr>
<tr><td>Web slice</td><td><code>@WebMvcTest</code> + <code>MockMvc</code></td><td>Controllers only; mock services with <code>@MockitoBean</code> (Boot 3.4+; earlier <code>@MockBean</code>)</td></tr>
<tr><td>Data slice</td><td><code>@DataJpaTest</code></td><td>Repositories with embedded DB, rolled back per test</td></tr>
<tr><td>Full</td><td><code>@SpringBootTest</code> (+ <code>TestRestTemplate</code>/<code>WebTestClient</code>)</td><td>Whole context; slowest</td></tr>
<tr><td>Real dependencies</td><td>Testcontainers (<code>@ServiceConnection</code> in Boot 3.1+)</td><td>Real Postgres/Kafka in Docker instead of H2</td></tr>
</table>

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mvc;
    @MockitoBean UserService service;

    @Test
    void returns404WhenMissing() throws Exception {
        when(service.find(1L)).thenThrow(new NotFoundException());
        mvc.perform(get("/users/1")).andExpect(status().isNotFound());
    }
}
```

<h3>Tips</h3>
<ul>
<li>Spring caches contexts between tests with identical config; avoid needless <code>@DirtiesContext</code>.</li>
<li>Use <code>@ActiveProfiles("test")</code> and test properties for config.</li>
<li>H2 can behave differently from production DBs; Testcontainers is more faithful.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 21. What is a Feign client?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>OpenFeign (Spring Cloud OpenFeign) is a declarative HTTP client: you write an interface with Spring MVC annotations and Spring generates the implementation that calls the remote service.</p>

```java
@EnableFeignClients            // on a configuration class
public class App { }

@FeignClient(name = "inventory", url = "${inventory.url}", fallbackFactory = InventoryFallback.class)
public interface InventoryClient {
    @GetMapping("/items/{id}")
    ItemDto getItem(@PathVariable("id") long id);

    @PostMapping("/items")
    ItemDto create(@RequestBody ItemDto item);
}
```

<h3>Why / Context</h3>
<ul>
<li>Removes boilerplate compared with hand-written <code>RestTemplate</code> code.</li>
<li>Integrates with service discovery and load balancing (Spring Cloud LoadBalancer) and Resilience4j circuit breakers/fallbacks.</li>
<li>Supports request interceptors (auth headers), custom encoders/decoders, and error decoders.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Set connect/read timeouts explicitly; defaults can be too generous.</li>
<li>Non-2xx responses throw <code>FeignException</code>; map them with an <code>ErrorDecoder</code>.</li>
<li>Retries on non-idempotent calls can duplicate side effects.</li>
<li>Alternatives in current Spring: <code>RestClient</code> (6.1+) with HTTP interface clients (<code>@HttpExchange</code>), which are the built-in declarative option.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/spring-cloud/">Learn more: Spring Cloud & Microservices</a></p>

</details>

---

## 22. Explain Spring Cloud circuit breaker patterns

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Spring Cloud Circuit Breaker is an abstraction over circuit-breaker libraries (Resilience4j is the standard; Hystrix is retired). A breaker wraps a remote call and, after too many failures, opens to fail fast instead of piling up on a sick dependency.</p>

<h3>States</h3>
<ul>
<li><strong>Closed:</strong> calls pass; failures are counted in a sliding window.</li>
<li><strong>Open:</strong> failure or slow-call rate exceeded the threshold; calls are rejected immediately (fallback).</li>
<li><strong>Half-open:</strong> after a wait duration, a few trial calls decide whether to close or re-open.</li>
</ul>

<h3>Example</h3>

```java
@Service
class QuoteService {
    private final CircuitBreakerFactory<?, ?> factory;
    private final QuoteClient client;

    QuoteService(CircuitBreakerFactory<?, ?> factory, QuoteClient client) {
        this.factory = factory; this.client = client;
    }

    Quote quote(String id) {
        return factory.create("quotes")
                .run(() -> client.fetch(id), t -> Quote.cached(id)); // fallback
    }
}
```

<h3>Related Patterns</h3>
<ul>
<li><strong>Timeout, Retry (with backoff), Bulkhead, Rate limiter</strong> from Resilience4j; combine them (order matters: retry wraps the breaker or vice versa deliberately).</li>
<li>Fallback must be cheap and safe (cache, default, degraded response).</li>
</ul>

<h3>Gotchas</h3>
<p>Tune thresholds from real traffic; count slow calls as failures; expose breaker state through Actuator and metrics.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker/">Learn more: Spring Boot Circuit Breaker</a></p>

</details>

---

## 23. How do you handle multithreading in Spring Boot applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Each web request already runs on a container thread (Tomcat pool, 200 by default). For extra concurrency use managed executors: <code>@Async</code>, a <code>TaskExecutor</code>/<code>ExecutorService</code> bean, <code>CompletableFuture</code>, or virtual threads on Java 21+.</p>

<h3>Options</h3>
<ul>
<li><strong>@Async</strong> with a named <code>ThreadPoolTaskExecutor</code> (core/max/queue sizes, thread name prefix, rejection policy).</li>
<li><strong>CompletableFuture</strong> with your executor passed explicitly (avoid the common ForkJoin pool for blocking I/O).</li>
<li><strong>Virtual threads:</strong> <code>spring.threads.virtual.enabled=true</code> (Boot 3.2+, Java 21+) runs request handling and <code>@Async</code>/scheduling on virtual threads.</li>
<li><strong>Reactive</strong> (WebFlux) for event-loop style non-blocking I/O.</li>
</ul>

```java
@Bean("ioExecutor")
ThreadPoolTaskExecutor ioExecutor() {
    var ex = new ThreadPoolTaskExecutor();
    ex.setCorePoolSize(8);
    ex.setMaxPoolSize(16);
    ex.setQueueCapacity(200);
    ex.setThreadNamePrefix("io-");
    ex.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
    return ex;
}
```

<h3>Gotchas</h3>
<ul>
<li>Singleton beans are shared across threads: keep them stateless or thread-safe.</li>
<li>Thread-locals (security context, MDC, transaction) do not propagate to new threads automatically.</li>
<li>Unbounded queues hide overload; size pools from workload (CPU-bound about cores, I/O-bound more).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-threading/">Learn more: Spring Threading & Async Patterns</a></p>

</details>

---

## 24. What is the difference between async and sync execution in Spring?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Sync: the caller thread waits until the method returns. Async (<code>@Async</code>): Spring runs the method on a task-executor thread and the caller continues, optionally holding a <code>Future</code>/<code>CompletableFuture</code>.</p>

<table>
<tr><th></th><th>Sync</th><th>@Async</th></tr>
<tr><td>Thread</td><td>Caller's</td><td>Executor thread</td></tr>
<tr><td>Return type</td><td>Any</td><td><code>void</code>, <code>Future</code>, <code>CompletableFuture</code></td></tr>
<tr><td>Exceptions</td><td>Propagate to caller</td><td>Lost for <code>void</code> unless an <code>AsyncUncaughtExceptionHandler</code> is set; carried in the future otherwise</td></tr>
<tr><td>Transaction / security context</td><td>Shared with caller</td><td>Not propagated</td></tr>
</table>

```java
@EnableAsync @Configuration class AsyncConfig {}

@Service
class ReportService {
    @Async("ioExecutor")
    public CompletableFuture<Report> build(long id) {
        return CompletableFuture.completedFuture(generate(id));
    }
}

// caller
var a = reports.build(1);
var b = reports.build(2);
CompletableFuture.allOf(a, b).join();   // run in parallel, then wait
```

<h3>Gotchas</h3>
<ul>
<li>Works through a proxy: calling an <code>@Async</code> method from the same class (self-invocation) or on a private method runs synchronously.</li>
<li>Without <code>@EnableAsync</code> nothing happens; the default executor config matters (name your own).</li>
<li>Plain <code>Future.get()</code> blocks; prefer <code>CompletableFuture</code> composition.</li>
</ul>

<h3>When to Use</h3>
<p>Fire-and-forget side effects (emails, audit) and parallel independent I/O. Not for work that must be part of the caller's transaction.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-threading/">Learn more: Spring Threading & Async Patterns</a></p>

</details>

---

**See also:** [Spring Boot Fundamentals](../../spring-web/spring-boot.md), [Spring Annotations](../../spring-web/spring-annotations-components.md), [Interview Preparation](../index.md)
