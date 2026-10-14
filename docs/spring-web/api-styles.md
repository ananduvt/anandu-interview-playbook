# API Styles & Protocols

A quick comparison of the common ways services talk to each other.

## REST (Representational State Transfer)
- Architectural style over HTTP: **stateless**, client-server, cacheable, uniform interface, layered.
- Resources identified by URLs; manipulated via representations (JSON/XML); optional HATEOAS links.
- **Use for**: web/mobile APIs, cloud services — the default for CRUD-style resource APIs.

## Webhooks
- **Reverse API**: the server pushes an HTTP POST to a client-registered URL when an event occurs (no polling).
- Event-driven, real-time notifications.
- **Use for**: payment notifications, VCS/CI events, social updates.

## GraphQL
- Query language + runtime; clients request **exactly** the fields they need from a **single endpoint**.
- Strongly-typed schema; avoids over-/under-fetching.
- **Use for**: complex/nested data needs, mobile/front-end-driven data fetching.

## SOAP (Simple Object Access Protocol)
- XML-based messaging protocol over HTTP/SMTP; standardized, with WS-* specs (security, transactions).
- Platform/language independent, contract-first (WSDL).
- **Use for**: enterprise/legacy integrations, strict contracts, security-sensitive transactions.

## WebSocket
- **Full-duplex**, persistent connection over a single TCP connection; low latency.
- Event-driven push without polling.
- **Use for**: chat, live dashboards, online gaming, real-time collaboration.

## gRPC
- High-performance RPC over **HTTP/2** using **Protocol Buffers** (IDL + binary message format).
- Strongly typed, language-independent, supports bidirectional **streaming**.
- **Use for**: internal service-to-service calls where performance and strict contracts matter.

## Quick Chooser
| Need | Style |
|------|-------|
| Standard resource CRUD API | REST |
| Client picks exact fields, nested data | GraphQL |
| Push notifications to third parties | Webhooks |
| High-perf internal RPC + streaming | gRPC |
| Real-time bidirectional | WebSocket |
| Enterprise contract + WS-* security | SOAP |

---

## HTTP Methods Deep-Dive

### GET — Retrieve Resource
- **Idempotent**: Multiple calls return same result ✅
- **Safe**: No side effects, no data change ✅
- **Cacheable**: Response can be cached ✅
- **Body**: No request body (ignored if present)
- **Status**: 200 OK, 404 Not Found, 410 Gone, 304 Not Modified (cached)

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return user != null ? ResponseEntity.ok(user) : ResponseEntity.notFound().build();
}

@GetMapping("/users")
public ResponseEntity<List<User>> listUsers(
    @RequestParam(required = false) String role,
    @RequestParam(defaultValue = "0") int page) {
    List<User> users = userService.findByRole(role, page);
    return ResponseEntity.ok(users);
}
```

### POST — Create New Resource
- **Idempotent**: ❌ Each call creates new resource
- **Safe**: ❌ Changes server state
- **Cacheable**: Only if response has Cache-Control header
- **Body**: Required, contains new resource data
- **Status**: 201 Created + Location header, 400 Bad Request, 409 Conflict (duplicate)

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody CreateUserRequest req) {
    User created = userService.create(req);
    return ResponseEntity
        .created(URI.create("/users/" + created.getId()))  // Include Location header
        .body(created);
}
```

**Idempotency Issue:**
```
POST /users {"name": "John"}
→ Creates user #1

POST /users {"name": "John"}  (Network retry)
→ Creates user #2 again! (Duplicate)

Solution: Idempotent key (client-generated UUID)
Header: Idempotent-Key: abc-123-def
Server checks: if seen before, return cached response
```

### PUT — Replace Entire Resource
- **Idempotent**: ✅ Same replacement = same state
- **Safe**: ❌ Modifies data
- **Cacheable**: ❌ No
- **Body**: Required, complete resource representation
- **Status**: 200 OK, 204 No Content, 201 Created (if resource didn't exist), 400 Bad Request, 404 Not Found

```java
@PutMapping("/users/{id}")
public ResponseEntity<User> updateUser(
    @PathVariable Long id,
    @RequestBody User updatedUser) {
    // Replace entire user (all fields)
    User result = userService.replace(id, updatedUser);
    return result != null ? ResponseEntity.ok(result) : ResponseEntity.notFound().build();
}
```

**PUT vs PATCH:**
```
Current: {"name": "John", "email": "john@old.com", "role": "USER"}

PUT /users/1 {"name": "John", "email": "john@new.com", "role": "USER"}
→ Replace all fields; if "role" missing, set to null?

PATCH /users/1 {"email": "john@new.com"}
→ Update only email field; keep other fields
```

### PATCH — Partial Update
- **Idempotent**: ✅ (if implemented correctly)
- **Safe**: ❌ Modifies data
- **Cacheable**: ❌ No
- **Body**: Only changed fields (partial representation)
- **Status**: 200 OK, 204 No Content, 400 Bad Request, 404 Not Found

```java
@PatchMapping("/users/{id}")
public ResponseEntity<User> partialUpdate(
    @PathVariable Long id,
    @RequestBody Map<String, Object> updates) {
    // Update only provided fields
    User result = userService.patch(id, updates);
    return result != null ? ResponseEntity.ok(result) : ResponseEntity.notFound().build();
}

// Example: Only update email
// PATCH /users/1 {"email": "newemail@example.com"}
```

**JSON Patch Standard (RFC 6902):**
```json
PATCH /users/1
Content-Type: application/json-patch+json

[
  {"op": "replace", "path": "/email", "value": "new@example.com"},
  {"op": "add", "path": "/tags", "value": ["vip"]},
  {"op": "remove", "path": "/notes"}
]
```

### DELETE — Remove Resource
- **Idempotent**: ✅ Deleting twice = already gone
- **Safe**: ❌ Modifies data
- **Cacheable**: ❌ No
- **Body**: Usually none (some APIs allow reason in body)
- **Status**: 204 No Content, 200 OK (with response body), 404 Not Found, 409 Conflict (can't delete due to constraints)

```java
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    boolean deleted = userService.delete(id);
    return deleted ? ResponseEntity.noContent().build() : ResponseEntity.notFound().build();
}

// With reason in body (soft delete):
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(
    @PathVariable Long id,
    @RequestBody Map<String, String> reason) {
    userService.delete(id, reason.get("reason"));
    return ResponseEntity.noContent().build();
}
```

### HEAD — Like GET but No Body
- Same as GET but response has no body
- Use for checking if resource exists without downloading large data
- **Status**: Same as GET (200, 404, etc.)

```bash
HEAD /users/1          # Check if user exists (no body returned)
→ 200 OK (exists)
→ 404 Not Found (doesn't exist)
```

### OPTIONS — Describe Communication
- Returns allowed methods for a resource
- Used by browsers before CORS requests

```bash
OPTIONS /users/1
→ Allow: GET, POST, PUT, PATCH, DELETE, OPTIONS
```

---

## REST Status Codes Cheat Sheet

| Code | Meaning | Use |
|------|---------|-----|
| **2xx Success** |
| 200 | OK | Request succeeded, body included |
| 201 | Created | Resource created, Location header + body |
| 204 | No Content | Request succeeded, no body (often DELETE) |
| 206 | Partial Content | Partial response (pagination, ranges) |
| **3xx Redirection** |
| 301 | Moved Permanently | Resource moved to new URL (cache it) |
| 302 | Found | Temporary redirect (don't cache) |
| 304 | Not Modified | Cached version is current (no body) |
| **4xx Client Error** |
| 400 | Bad Request | Invalid input (malformed JSON, missing field) |
| 401 | Unauthorized | Authentication required (missing JWT token) |
| 403 | Forbidden | Authenticated but no permission (insufficient role) |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Request conflicts (duplicate key, pre-condition failed) |
| 422 | Unprocessable Entity | Valid format but semantic error (e.g., invalid email) |
| 429 | Too Many Requests | Rate limit exceeded |
| **5xx Server Error** |
| 500 | Internal Server Error | Unexpected error (exception) |
| 502 | Bad Gateway | Upstream service unavailable |
| 503 | Service Unavailable | Server temporarily down (maintenance) |
| 504 | Gateway Timeout | Upstream service timeout |

---

## REST Best Practices

### Versioning
```
Option 1: URL Path (explicit)
  GET /api/v1/users
  GET /api/v2/users (breaking changes in v2)

Option 2: Header (cleaner URLs)
  GET /api/users
  Header: Accept: application/vnd.company.v1+json

Option 3: Query Parameter
  GET /api/users?version=1
```

### Pagination
```java
@GetMapping("/users")
public ResponseEntity<Page<User>> listUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id,desc") String sort) {
    return ResponseEntity.ok(
        userService.findAll(PageRequest.of(page, size, Sort.by(sort)))
    );
}

// Response: /users?page=0&size=20
{
  "content": [...],
  "pageNumber": 0,
  "pageSize": 20,
  "totalElements": 150,
  "totalPages": 8,
  "isLast": false
}
```

### Filtering & Searching
```
GET /users?role=ADMIN&status=active&search=john
GET /invoices?dateFrom=2024-01-01&dateTo=2024-12-31&minAmount=1000
```

### HATEOAS (Hypermedia As The Engine Of Application State)
Include links to related resources:
```json
{
  "id": 1,
  "name": "John Doe",
  "_links": {
    "self": { "href": "/users/1" },
    "all": { "href": "/users" },
    "update": { "href": "/users/1", "method": "PUT" },
    "delete": { "href": "/users/1", "method": "DELETE" }
  }
}
```

Spring HATEOAS:
```java
@GetMapping("/users/{id}")
public EntityModel<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return EntityModel.of(user,
        linkTo(methodOn(UserController.class).getUser(id)).withSelfRel(),
        linkTo(methodOn(UserController.class).listUsers(null, 0)).withRel("all"),
        linkTo(methodOn(UserController.class).updateUser(id, null)).withRel("update")
    );
}
```

---

## Swagger / OpenAPI

**OpenAPI** (formerly Swagger) is the standard for documenting REST APIs. Automatic documentation, testing, and code generation from annotations.

### Setup: Springdoc OpenAPI

Maven dependency:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.4</version>
</dependency>
```

Automatic endpoints:
- `/swagger-ui.html` — Interactive UI (test endpoints live)
- `/v3/api-docs` — OpenAPI JSON spec
- `/v3/api-docs.yaml` — OpenAPI YAML spec

### Basic Annotations

```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "User Management", description = "Operations for managing users")
public class UserController {
    
    @GetMapping("/{id}")
    @Operation(summary = "Get user by ID", description = "Fetch a specific user")
    @ApiResponse(responseCode = "200", description = "User found")
    @ApiResponse(responseCode = "404", description = "User not found")
    public ResponseEntity<User> getUser(
            @Parameter(description = "User ID") @PathVariable Long id) {
        return ResponseEntity.ok(new User(id, "John"));
    }
    
    @PostMapping
    @Operation(summary = "Create new user")
    @ApiResponse(responseCode = "201", description = "User created")
    @ApiResponse(responseCode = "400", description = "Invalid input")
    public ResponseEntity<User> createUser(
            @RequestBody 
            @io.swagger.v3.oas.annotations.media.Content(
                    mediaType = "application/json",
                    examples = @ExampleObject(value = "{\"name\":\"John\",\"email\":\"john@example.com\"}"))
            CreateUserRequest req) {
        User user = new User(1L, req.getName());
        return ResponseEntity.status(201).body(user);
    }
    
    @PutMapping("/{id}")
    @Operation(summary = "Update user")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id,
            @RequestBody UpdateUserRequest req) {
        return ResponseEntity.ok(new User(id, req.getName()));
    }
    
    @PatchMapping("/{id}")
    @Operation(summary = "Partially update user")
    public ResponseEntity<User> patchUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        // Update only provided fields
        return ResponseEntity.ok(new User(id, "Updated"));
    }
    
    @DeleteMapping("/{id}")
    @Operation(summary = "Delete user")
    @ApiResponse(responseCode = "204", description = "User deleted")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        return ResponseEntity.noContent().build();
    }
}
```

### Key Annotations

| Annotation | Purpose |
|-----------|---------|
| `@Tag` | Group endpoints (shown in UI tabs) |
| `@Operation` | Describe endpoint (summary, description) |
| `@Parameter` | Describe path/query/header parameters |
| `@RequestBody` | Document request body |
| `@ApiResponse` | Document HTTP responses (status codes, description) |
| `@Schema` | Document data model fields |
| `@Deprecated` | Mark endpoint as deprecated |

### Document Model/DTO

```java
@Data
@Schema(description = "User model")
public class User {
    
    @Schema(description = "Unique user ID", example = "123")
    private Long id;
    
    @Schema(description = "User's full name", example = "John Doe", minLength = 1, maxLength = 100)
    private String name;
    
    @Schema(description = "User email", example = "john@example.com")
    @Email(message = "Invalid email format")
    private String email;
    
    @Schema(description = "User role", allowableValues = {"USER", "ADMIN", "MODERATOR"})
    private String role;
    
    @Schema(description = "Account creation date", example = "2024-01-15T10:30:00Z")
    private LocalDateTime createdAt;
}

@Schema(description = "Request to create a new user")
public class CreateUserRequest {
    
    @Schema(required = true, example = "John Doe")
    private String name;
    
    @Schema(required = true, example = "john@example.com")
    private String email;
}
```

### Configuration

```yaml
# application.yml
springdoc:
  api-docs:
    path: /v3/api-docs  # Custom path for OpenAPI JSON
  swagger-ui:
    path: /swagger-ui.html
    enabled: true
    show-common-extensions: true
    display-request-duration: true
    filter: true  # Enable search filter
  
  # Hide endpoints matching pattern
  ignored-url-patterns: /internal/**

# Optionally, expose only specific endpoints
springdoc:
  paths-to-match: /api/**
  paths-to-exclude: /admin/**
```

### Advanced: Custom OpenAPI Configuration

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User Management API")
                .version("1.0.0")
                .description("REST API for user management")
                .contact(new Contact()
                    .name("API Support")
                    .email("support@example.com")
                    .url("https://example.com"))
                .license(new License()
                    .name("Apache 2.0")
                    .url("https://www.apache.org/licenses/LICENSE-2.0")))
            .servers(Arrays.asList(
                new Server()
                    .url("https://api.example.com")
                    .description("Production"),
                new Server()
                    .url("http://localhost:8080")
                    .description("Development")))
            .components(new Components()
                .addSecuritySchemes("bearer-jwt",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")
                        .description("Enter JWT token")));
    }
}
```

### Security Documentation

```java
@Configuration
public class SecurityOpenApiConfig {
    
    @Bean
    public OpenAPI securedOpenAPI() {
        return new OpenAPI()
            .info(new Info().title("Secured API").version("1.0"))
            .components(new Components()
                .addSecuritySchemes("bearer-jwt",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")))
            .security(Arrays.asList(
                new SecurityRequirement().addList("bearer-jwt")));
    }
}

@RestController
public class ProtectedController {
    
    @GetMapping("/protected")
    @Operation(summary = "Protected endpoint")
    @SecurityRequirement(name = "bearer-jwt")
    public ResponseEntity<String> protectedEndpoint() {
        return ResponseEntity.ok("Access granted");
    }
}
```

### Example Responses

Full example with nested models:

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @GetMapping("/{id}")
    @Operation(summary = "Get order with items", description = "Fetch complete order including all line items")
    @ApiResponse(responseCode = "200", description = "Order found",
        content = @Content(
            mediaType = "application/json",
            schema = @Schema(implementation = OrderResponse.class),
            examples = @ExampleObject(
                value = "{\"id\":1,\"userId\":10,\"total\":299.99,\"items\":[{\"productId\":5,\"quantity\":2,\"price\":149.99}]}")))
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        return ResponseEntity.ok(new OrderResponse(1L, 10L, 299.99, 
            List.of(new OrderItem(5L, 2, 149.99))));
    }
}

@Data
@Schema(description = "Complete order with items")
public class OrderResponse {
    
    @Schema(example = "1")
    private Long id;
    
    @Schema(example = "10")
    private Long userId;
    
    @Schema(example = "299.99")
    private Double total;
    
    @Schema(description = "Line items in this order")
    private List<OrderItem> items;
}

@Data
@Schema(description = "Individual order item")
public class OrderItem {
    
    @Schema(example = "5")
    private Long productId;
    
    @Schema(example = "2", minimum = "1")
    private Integer quantity;
    
    @Schema(example = "149.99", minimum = "0")
    private Double price;
}
```

### Generate Code from OpenAPI

Use OpenAPI Generator to create client/server code:

```bash
# Generate Java client from OpenAPI spec
mvn org.openapitools:openapi-generator-maven-plugin:generate \
  -Dopenapi.generator.maven.plugin.inputSpec=http://localhost:8080/v3/api-docs \
  -Dopenapi.generator.maven.plugin.generatorName=java \
  -Dopenapi.generator.maven.plugin.output=generated

# Generate TypeScript client
openapi-generator-cli generate \
  -i http://localhost:8080/v3/api-docs \
  -g typescript-axios \
  -o generated/client
```

### Best Practices

✅ **Do's**
- Document all endpoints and response codes
- Include examples in schemas
- Use meaningful descriptions (helps frontend developers)
- Document authentication requirements
- Keep OpenAPI spec updated with code
- Use tags to group related endpoints
- Document error responses

❌ **Don'ts**
- Don't expose sensitive endpoints in public API docs
- Don't include secrets/passwords in examples
- Don't over-document (brief and clear > verbose)
- Don't ignore deprecated endpoints (mark with @Deprecated)
- Don't generate docs manually (use annotations)
