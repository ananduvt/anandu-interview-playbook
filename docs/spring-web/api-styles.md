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
