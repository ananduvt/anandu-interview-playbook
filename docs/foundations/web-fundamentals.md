# Web Fundamentals

Core web concepts, HTTP, REST principles, API design, and communication protocols.

---

## HTTP (Hypertext Transfer Protocol)

**HTTP** — Application layer protocol for transferring data across the web.

### HTTP Methods (Verbs)

| Method | Purpose | Idempotent | Safe | Cacheable | Use |
|--------|---------|-----------|------|-----------|-----|
| **GET** | Retrieve resource | ✅ | ✅ | ✅ | Fetch data |
| **POST** | Create new resource | ❌ | ❌ | ⚠️ | Submit form, create |
| **PUT** | Replace entire resource | ✅ | ❌ | ❌ | Full update |
| **PATCH** | Partial update | ✅ | ❌ | ❌ | Partial update |
| **DELETE** | Remove resource | ✅ | ❌ | ❌ | Delete |
| **HEAD** | Like GET, no body | ✅ | ✅ | ✅ | Check if exists |
| **OPTIONS** | Describe communication | ✅ | ✅ | ❌ | CORS preflight |

### HTTP Status Codes

**2xx Success**
- **200 OK** — Request succeeded, response body included
- **201 Created** — Resource created, Location header + body
- **202 Accepted** — Request accepted, processing async
- **204 No Content** — Success, no body (DELETE, etc.)
- **206 Partial Content** — Partial response (pagination, ranges)

**3xx Redirection**
- **301 Moved Permanently** — Resource moved, cache new URL
- **302 Found** — Temporary redirect
- **304 Not Modified** — Cached version is current
- **307 Temporary Redirect** — Like 302 but preserve method

**4xx Client Error**
- **400 Bad Request** — Invalid input (malformed JSON, missing field)
- **401 Unauthorized** — Authentication required (missing token)
- **403 Forbidden** — Authenticated but no permission (insufficient role)
- **404 Not Found** — Resource doesn't exist
- **409 Conflict** — Conflict (duplicate key, pre-condition failed)
- **422 Unprocessable Entity** — Valid format, semantic error
- **429 Too Many Requests** — Rate limit exceeded
- **431 Request Header Fields Too Large** — Headers too large

**5xx Server Error**
- **500 Internal Server Error** — Unexpected error (exception)
- **502 Bad Gateway** — Upstream service unavailable
- **503 Service Unavailable** — Server temporarily down (maintenance)
- **504 Gateway Timeout** — Upstream service timeout

### HTTP Headers

**Request Headers**
```
Authorization: Bearer <token>          # Authentication
Content-Type: application/json         # Request body format
Accept: application/json               # Expected response format
Accept-Encoding: gzip, deflate        # Compression
User-Agent: curl/7.68.0               # Client info
If-None-Match: "33a64df551425fcc"     # ETag (caching)
```

**Response Headers**
```
Content-Type: application/json         # Response format
Content-Length: 1234                   # Body size
Cache-Control: max-age=3600            # Cache duration
ETag: "33a64df551425fcc"              # Version hash
Location: /users/123                   # For 201 Created, 3xx redirects
Set-Cookie: session=abc123             # Session cookie
Access-Control-Allow-Origin: *         # CORS
```

### HTTP Caching

**Cache-Control** (request/response)
```
Cache-Control: public, max-age=3600    # Cacheable, 1 hour
Cache-Control: private, max-age=1800   # Client cache, 30 min
Cache-Control: no-cache                # Can cache, validate before reuse
Cache-Control: no-store                # Don't cache
Cache-Control: must-revalidate         # Revalidate when expired
```

**ETag** (entity tag for cache validation)
- Server: `ETag: "abc123"`
- Client: `If-None-Match: "abc123"` → Server responds 304 Not Modified

---

## REST (Representational State Transfer)

**REST** — Architectural style for APIs using HTTP methods and resources.

### Core Principles

1. **Client-Server Architecture** — Separation of concerns
2. **Statelessness** — Each request contains all info needed, no server-side session
3. **Uniform Interface** — Resources identified by URLs, manipulated via representations
4. **Cacheability** — Response can be cached to improve performance
5. **Layered System** — Multiple layers (load balancer, proxy, server)
6. **Code on Demand** (optional) — Server can extend client functionality

### RESTful Design

**Resource Naming**
```
GET /api/users              # List all users
GET /api/users/123          # Get user 123
POST /api/users             # Create new user
PUT /api/users/123          # Replace user 123
PATCH /api/users/123        # Update user 123
DELETE /api/users/123       # Delete user 123

GET /api/users/123/posts    # User 123's posts (nested resource)
GET /api/posts?author=123   # Filter by query param
```

**Idempotency**
- GET, PUT, DELETE, HEAD, OPTIONS are idempotent (same result each call)
- POST is NOT idempotent (each call creates new resource)

**Solution for POST idempotency:**
```
Header: Idempotent-Key: uuid-12345
Server: If key seen before, return cached response
        If new, process and cache result
```

### REST Best Practices

**Versioning**
```
URL path:   GET /api/v1/users (explicit)
Header:     GET /api/users + Accept: application/vnd.company.v1+json
Query:      GET /api/users?version=1
```

**Pagination**
```
GET /api/users?page=0&size=20&sort=id,desc

Response:
{
  "content": [...],
  "totalElements": 150,
  "totalPages": 8,
  "currentPage": 0,
  "pageSize": 20,
  "isLast": false
}
```

**Filtering & Searching**
```
GET /api/users?role=ADMIN&status=active
GET /api/users?search=john&createdAfter=2024-01-01
GET /api/invoices?minAmount=1000&maxAmount=5000
```

**HATEOAS (Hypermedia As The Engine Of Application State)**
Include links to related resources:
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "_links": {
    "self": { "href": "/api/users/1", "method": "GET" },
    "all": { "href": "/api/users", "method": "GET" },
    "update": { "href": "/api/users/1", "method": "PUT" },
    "delete": { "href": "/api/users/1", "method": "DELETE" }
  }
}
```

---

## Web Services Architecture

### SOAP (Simple Object Access Protocol)
- **Protocol**: XML-based messaging over HTTP/SMTP
- **Contract**: WSDL (Web Services Description Language)
- **Pros**: Standardized (WS-* specs), platform-independent, strict contracts
- **Cons**: Verbose (XML), complex, slower than REST
- **Use**: Enterprise integrations, legacy systems, security-critical transactions

### REST
- **Simplicity**: HTTP methods, JSON/XML
- **Stateless**: Each request independent
- **Cache-friendly**: Built-in HTTP caching
- **Use**: Web APIs, mobile apps, microservices

### GraphQL
- **Query Language**: Clients request exactly the fields they need
- **Single Endpoint**: POST /graphql
- **Strongly Typed**: Schema defines all possible queries
- **Avoids**: Over-fetching (unnecessary data), under-fetching (multiple requests)
- **Use**: Complex/nested data needs, frontend-driven queries

### gRPC
- **Protocol**: HTTP/2 + Protocol Buffers (binary)
- **Performance**: High-speed, low-latency, bidirectional streaming
- **Type Safety**: Strongly typed with .proto IDL
- **Use**: Internal service-to-service communication, high-performance needs

### WebSocket
- **Connection**: Full-duplex, persistent TCP connection
- **Real-time**: Push data without polling
- **Use**: Chat, live dashboards, real-time notifications, online gaming

---

## API Design Best Practices

### Error Handling
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "User not found",
    "details": {
      "userId": "123 does not exist"
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/users/123"
  }
}
```

### Rate Limiting Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705323600
```

### Async Operations (202 Accepted)
```
POST /api/reports/generate
→ 202 Accepted
→ Location: /api/jobs/abc123

GET /api/jobs/abc123
→ 202 Accepted (still processing)
→ GET /api/jobs/abc123
→ 200 OK + result (when done)
```

### Bulk Operations
```
POST /api/users/bulk
{
  "users": [
    {"name": "john"},
    {"name": "jane"}
  ]
}
→ 207 Multi-Status
{
  "responses": [
    {"status": 201, "body": {...}},
    {"status": 409, "body": {"error": "duplicate"}}
  ]
}
```

### Deprecation
```
GET /api/v1/users
Response Header: Deprecation: true
                Sunset: Sun, 01 Jan 2025 00:00:00 GMT
                Link: </api/v2/users>; rel="successor-version"
```

---

## Security Considerations

### HTTPS/TLS
- Always use HTTPS in production
- Encrypt data in transit
- Validate certificates

### Authentication
- **API Keys**: Simple, suitable for public APIs
- **Bearer Tokens (JWT)**: Stateless, scalable
- **OAuth2**: Delegated authorization, third-party apps
- **Mutual TLS (mTLS)**: Certificate-based, service-to-service

### Authorization
- Role-Based (RBAC): Users have roles with permissions
- Attribute-Based (ABAC): Fine-grained policies
- **Never** trust client-provided authorization data

### Input Validation
- Validate all input (size, type, format)
- Reject invalid data early (400 Bad Request)
- Sanitize to prevent injection attacks

### CORS (Cross-Origin Resource Sharing)
- Browser blocks requests across domains by default
- Server sends `Access-Control-Allow-Origin` header
- Preflight OPTIONS request for non-simple requests

```
Request from: https://frontend.example.com
To: https://api.example.com

OPTIONS /api/users
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type

Response:
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

---

## Observability

### Logging
```
POST /api/users {"name": "John"}
→ 201 Created, Location: /api/users/123
LOG: [2024-01-15 10:30:00] POST /api/users → 201 (15ms)
```

### Tracing
- Trace ID: Unique per request (carry across microservices)
- Span ID: Individual operation

```
X-Trace-ID: abc-123-def
X-Span-ID: span-001

Service A → Service B → Database
All share same Trace ID
```

### Metrics
- Request count, latency, error rate
- Response time percentiles (p50, p95, p99)
- Payload sizes

---

**See also:** [REST API Best Practices](../spring-web/api-styles.md), [HTTP Security](../spring-web/security.md), [Web Services](../spring-web/web-services.md)

