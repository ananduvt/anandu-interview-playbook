# API Evolution and Optimistic Concurrency (ETag / If-Match)

Two related contract topics: how to change an API without breaking consumers, and how to stop two clients overwriting each other's updates. For API styles and versioning URLs see [API Styles](api-styles.md); for HTTP caching with `ETag` / `If-None-Match` see [Web Fundamentals](../foundations/web-fundamentals.md).

## Compatibility Directions

| Term | Meaning | Who is protected |
| :-- | :-- | :-- |
| **Backward compatible** | New server works with old clients | Existing consumers |
| **Forward compatible** | Old server/client tolerates data from a newer one | Consumers or servers lagging behind a rollout |

During a rolling deploy or with independently released services, both directions occur at once.

### Safe vs Breaking Changes (JSON over HTTP)

| Change | Safe? |
| :-- | :-- |
| Add an optional response field | Yes (if clients ignore unknown fields) |
| Add an optional request field or query parameter | Yes |
| Add a new endpoint or enum value in a request | Yes |
| Add a new enum value in a **response** | Risky: old clients may fail on unknown values |
| Remove or rename a field, endpoint, or parameter | Breaking |
| Change a type, format, units, or meaning of a field | Breaking |
| Make an optional request field required | Breaking |
| Tighten validation or change error codes | Usually breaking |
| Change default behavior or ordering | Often breaking |

## Tolerant Reader (Postel's Law)

Be liberal in what you accept: a consumer reads only the fields it needs and ignores the rest, and never fails on unknown properties or enum values.

```yaml
spring:
  jackson:
    deserialization:
      fail-on-unknown-properties: false   # Boot default is already false
```

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public record PaymentEvent(String id, long amount,
        @JsonSetter(nulls = Nulls.SKIP) String currency) {}

// Unknown enum value falls back instead of throwing
@JsonEnumDefaultValue UNKNOWN   // with READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE enabled
```

Do not bind your own domain to a partner schema field-for-field; map the few fields you need (anti-corruption layer).

## Expand and Contract (Parallel Change)

Evolve in three steps so no consumer breaks at any moment:

1. **Expand**: add the new field/endpoint next to the old one; the server fills both.
2. **Migrate**: move consumers over; watch usage metrics and logs for the old one.
3. **Contract**: remove the old one after consumers are gone.

```java
public record CustomerResponse(
        Long id,
        @Deprecated String name,        // old, kept until consumers move
        String fullName) {}             // new

// Header-signalled deprecation (RFC 9745 Deprecation and RFC 8594 Sunset)
// Deprecation: true      Sunset: Wed, 01 Jul 2027 00:00:00 GMT
```

The same pattern applies to messages and schemas: for events (Kafka) use a schema registry with Avro or Protobuf compatibility modes (backward, forward, full), and see [Schema Migrations](../databases/schema-migrations.md) for the database version.

## Versioning When You Must Break

| Strategy | Example | Notes |
| :-- | :-- | :-- |
| URI | `/v2/orders` | Simple, cache and route friendly, clients see it |
| Header / media type | `Accept: application/vnd.shop.v2+json` | Clean URLs, harder to test |
| Query parameter | `?version=2` | Easy but easily dropped |
| No versioning, additive only | Evolve with expand/contract | Best default; version only for real breaks |

Spring Framework 7 / Boot 4 adds first-class API versioning support in MVC; on Boot 3 use path variables or `produces` on mappings.

```java
@GetMapping(value = "/orders/{id}", produces = "application/vnd.shop.v2+json")
OrderV2 getV2(@PathVariable Long id) { ... }
```

## Contract Tests

Integration tests against a shared environment catch breakage late. **Consumer-driven contract testing** records what each consumer really uses and verifies the provider against it in CI.

| Tool | Approach |
| :-- | :-- |
| **Pact** | Consumer writes expectations, publishes a pact; provider verifies it (pact broker, `can-i-deploy`) |
| **Spring Cloud Contract** | Contracts (Groovy/YAML) generate provider tests and consumer stubs (WireMock) |
| **OpenAPI diff** (`oasdiff`, `openapi-diff`) | Compare spec versions in CI and fail on breaking changes |

Pipeline idea: provider build verifies all consumer contracts, and a failing contract blocks the release, long before production. Testing details: [Testing](../foundations/testing.md).

## Optimistic Concurrency in REST: ETag, If-Match, 412

Problem: client A and B both GET a resource, both PUT changes; the second silently overwrites the first (**lost update**). Fix with a version validator the server checks on write.

| Header | Direction | Purpose |
| :-- | :-- | :-- |
| `ETag: "7"` | Response | Opaque version of the representation |
| `If-Match: "7"` | Request (PUT/PATCH/DELETE) | Apply only if current version still matches |
| `412 Precondition Failed` | Response | Version differs, update rejected |
| `428 Precondition Required` | Response | Server demands `If-Match`; client sent none |

```text
GET /orders/42           -> 200  ETag: "7"
PUT /orders/42  If-Match: "7"  -> 200  ETag: "8"      (matches)
PUT /orders/42  If-Match: "7"  -> 412                  (someone else saved "8")
```

```java
@GetMapping("/orders/{id}")
ResponseEntity<OrderResponse> get(@PathVariable Long id) {
    Order o = service.find(id);
    return ResponseEntity.ok()
            .eTag("\"" + o.getVersion() + "\"")      // JPA @Version value
            .body(mapper.toResponse(o));
}

@PutMapping("/orders/{id}")
ResponseEntity<OrderResponse> update(@PathVariable Long id,
        @RequestHeader(value = HttpHeaders.IF_MATCH, required = false) String ifMatch,
        @Valid @RequestBody UpdateOrderRequest req) {
    if (ifMatch == null) {
        return ResponseEntity.status(HttpStatus.PRECONDITION_REQUIRED).build();   // 428
    }
    long expected = Long.parseLong(ifMatch.replace("\"", ""));
    Order saved = service.update(id, expected, req);      // throws on mismatch
    return ResponseEntity.ok().eTag("\"" + saved.getVersion() + "\"")
            .body(mapper.toResponse(saved));
}

@ExceptionHandler({StaleVersionException.class, ObjectOptimisticLockingFailureException.class})
ResponseEntity<Void> stale() {
    return ResponseEntity.status(HttpStatus.PRECONDITION_FAILED).build();   // 412
}
```

```java
@Transactional
public Order update(Long id, long expectedVersion, UpdateOrderRequest req) {
    Order o = repo.findById(id).orElseThrow(() -> new NotFoundException(id));
    if (o.getVersion() != expectedVersion) throw new StaleVersionException();
    mapper.update(req, o);
    return o;           // JPA @Version also guards the write itself at flush
}
```

Notes:

- The explicit version comparison covers the gap between the client's read and this transaction; JPA `@Version` (see [JPA and ORM](jpa-orm.md)) catches races inside the transaction (`OptimisticLockException`).
- ETag can be the entity version or a hash of the body. Strong ETags (`"7"`) are required for `If-Match`; weak (`W/"7"`) never match for it.
- Spring's `ShallowEtagHeaderFilter` hashes the response body: good for conditional GET caching (saves bandwidth, not server work), not a substitute for a version for writes.
- The same `If-Match` can guard `DELETE`. Use `If-None-Match: *` on PUT to mean create only if absent.
- Without ETags, expose a `version` field in the body and check it, a simpler JSON-level equivalent.
- Choose optimistic over pessimistic locking when conflicts are rare and requests span user think time.

## Common Interview Questions

- **What is a breaking change?** Anything an existing consumer can fail on: removed or renamed fields, type changes, new required inputs, changed semantics.
- **What is the tolerant reader pattern?** Read what you need, ignore unknown fields and enum values, validate only what you use.
- **How do you remove a field safely?** Expand and contract: deprecate, measure usage, remove after consumers migrate.
- **Pact vs Spring Cloud Contract?** Pact is consumer driven and polyglot with a broker; Spring Cloud Contract is provider-written contracts generating stubs and tests, JVM centric.
- **How do you prevent lost updates in REST?** ETag from version, client sends `If-Match`, server returns 412 on mismatch (428 if missing), backed by JPA `@Version`.
- **412 vs 409?** 412 means a request precondition header failed; 409 is a general state conflict (e.g. duplicate).

Reference: [RFC 9110 conditional requests](https://www.rfc-editor.org/rfc/rfc9110#name-conditional-requests), [Pact docs](https://docs.pact.io/), [Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract).
