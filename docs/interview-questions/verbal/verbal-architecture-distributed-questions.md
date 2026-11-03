# Distributed Architecture Patterns Interview Questions

Cross-cutting concerns, MVC vs WebFlux, API evolution, resilience, idempotency, and CQRS.

---

## 1. How do you manage cross-cutting concerns (logging, tracing, auth) across services?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Handle each concern once, at a shared layer, instead of in every service's business code: an <strong>API gateway</strong> for edge concerns, <strong>shared libraries/starters</strong> for in-process concerns, and a <strong>platform layer</strong> (sidecar/service mesh, collectors) for infrastructure concerns.</p>

<table>
<tr><th>Concern</th><th>Approach</th></tr>
<tr><td>Authentication</td><td>Gateway validates the token (OAuth2/JWT); services validate again as resource servers and apply authorization (zero trust)</td></tr>
<tr><td>Logging</td><td>Structured (JSON) logs with a correlation/trace ID in MDC, shipped to a central store</td></tr>
<tr><td>Tracing</td><td>Propagate trace context headers (W3C <code>traceparent</code>); Micrometer Tracing / OpenTelemetry</td></tr>
<tr><td>Metrics</td><td>Micrometer + Prometheus; consistent naming and tags</td></tr>
<tr><td>Resilience</td><td>Timeouts, retries, circuit breakers via Resilience4j or mesh policy</td></tr>
<tr><td>Config and secrets</td><td>Central config server or platform secrets manager</td></tr>
</table>

<h3>Example: In-Process Correlation ID Filter</h3>

```java
@Component
class CorrelationIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String id = Optional.ofNullable(req.getHeader("X-Correlation-Id"))
                            .orElse(UUID.randomUUID().toString());
        MDC.put("correlationId", id);
        res.setHeader("X-Correlation-Id", id);
        try {
            chain.doFilter(req, res);
        } finally {
            MDC.clear();             // avoid leaking into pooled threads
        }
    }
}
```

<h3>Gotchas</h3>
<ul>
<li>MDC is thread-local: it does not follow async hops or reactive pipelines unless context is propagated.</li>
<li>Package shared concerns as a versioned Spring Boot starter, but keep it thin; a fat common library couples all services' releases.</li>
<li>Never log secrets or personal data. Use AOP for method-level concerns inside one service.</li>
<li>Gateway vs mesh: the gateway handles north-south (client) traffic; a mesh handles east-west (service-to-service) traffic such as mTLS.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/spring-cloud/">Learn more: Spring Cloud</a></p>

</details>

---

## 2. When would you choose Spring MVC over Spring WebFlux?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Choose <strong>Spring MVC</strong> (servlet, thread-per-request, blocking) by default. Choose <strong>WebFlux</strong> (reactive, non-blocking, event loop) when the workload is dominated by waiting on I/O with very high concurrency, streaming, or back-pressure needs, and the whole stack can be non-blocking.</p>

<table>
<tr><th>Aspect</th><th>Spring MVC</th><th>Spring WebFlux</th></tr>
<tr><td>Model</td><td>One thread per request</td><td>Few event-loop threads, async pipelines (<code>Mono</code>/<code>Flux</code>)</td></tr>
<tr><td>Style</td><td>Imperative, simple to read and debug</td><td>Declarative reactive; steeper learning curve</td></tr>
<tr><td>Data access</td><td>JDBC/JPA fine</td><td>Needs reactive drivers (R2DBC, reactive Mongo/Redis)</td></tr>
<tr><td>Best for</td><td>CRUD, CPU-bound work, typical business apps</td><td>Streaming, SSE/WebSocket, gateways, many slow downstream calls</td></tr>
<tr><td>Failure mode</td><td>Thread pool exhaustion under slow I/O</td><td>Blocking an event-loop thread stalls many requests</td></tr>
</table>

<h3>Gotchas and Follow-ups</h3>
<ul>
<li>WebFlux does not make a single request faster; it improves scalability under concurrent I/O waits. CPU-bound work gains nothing.</li>
<li>One blocking call (JDBC, <code>Thread.sleep</code>, blocking HTTP client) inside a reactive chain defeats the model. Detect with BlockHound; offload with <code>publishOn(Schedulers.boundedElastic())</code>.</li>
<li><strong>Java 21+ virtual threads</strong> let blocking-style MVC code scale much better on I/O (<code>spring.threads.virtual.enabled=true</code> in recent Spring Boot), which removes much of the reason to go reactive for scalability alone.</li>
<li>Reactive stack traces and debugging are harder; the team skill set matters.</li>
<li>Choose WebFlux for end-to-end streaming/back-pressure or when you already have a reactive stack (for example Spring Cloud Gateway is reactive).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-webflux/">Learn more: Spring WebFlux</a></p>

</details>

---

## 3. How do you design backward and forward compatible APIs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Backward compatible:</strong> new server versions keep working for old clients. <strong>Forward compatible:</strong> old clients/servers tolerate data from newer versions. Evolve additively, never remove or repurpose, and version only for unavoidable breaking changes.</p>

<h3>Safe vs Breaking Changes</h3>
<table>
<tr><th>Safe (additive)</th><th>Breaking</th></tr>
<tr><td>Add optional request field or new response field</td><td>Remove or rename a field or endpoint</td></tr>
<tr><td>Add a new endpoint</td><td>Change a field type or meaning</td></tr>
<tr><td>Add a new enum value (if clients tolerate it)</td><td>Make an optional field required; tighten validation</td></tr>
</table>

<h3>Rules</h3>
<ul>
<li><strong>Tolerant reader:</strong> clients ignore unknown fields; in Jackson set <code>FAIL_ON_UNKNOWN_PROPERTIES</code> to false (<code>spring.jackson.deserialization.fail-on-unknown-properties=false</code>) or use <code>@JsonIgnoreProperties(ignoreUnknown = true)</code>. Treat unknown enum values defensively.</li>
<li><strong>Deprecate, then remove:</strong> mark fields/endpoints deprecated (documentation, <code>Deprecation</code>/<code>Sunset</code> headers), monitor usage, remove after a notice period.</li>
<li><strong>Version when needed:</strong> URI (<code>/v2/orders</code>), header, or media type; support old and new in parallel and keep the number of versions small.</li>
<li><strong>Expand/contract for data changes:</strong> add the new field, dual-write, migrate readers, then drop the old one.</li>
<li><strong>Contract tests</strong> (consumer-driven, for example Pact or Spring Cloud Contract) and OpenAPI diff checks in CI catch accidental breaks.</li>
<li>For events or binary formats, use schema evolution with a registry (Avro/Protobuf rules: add optional fields with defaults, never reuse field numbers).</li>
</ul>

<h3>Example</h3>

```java
@JsonIgnoreProperties(ignoreUnknown = true)       // forward compatible reader
public record OrderResponse(
        String id,
        BigDecimal total,
        @JsonInclude(JsonInclude.Include.NON_NULL) String currency   // added later, optional
) { }
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles and Protocols</a></p>

</details>

---

## 4. How do you implement circuit breaking and bulkheading?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A <strong>circuit breaker</strong> stops calling a failing dependency (fail fast) and probes for recovery. A <strong>bulkhead</strong> limits the resources one dependency can consume so its slowness cannot exhaust the whole service. In Spring they are usually implemented with <strong>Resilience4j</strong>.</p>

<h3>Circuit Breaker States</h3>
<ul>
<li><strong>Closed:</strong> calls pass; failures are counted over a sliding window.</li>
<li><strong>Open:</strong> failure or slow-call rate crosses the threshold; calls are rejected immediately (fallback).</li>
<li><strong>Half-open:</strong> after a wait duration, a few trial calls decide whether to close or re-open.</li>
</ul>

<h3>Bulkhead Types</h3>
<table>
<tr><th>Type</th><th>How it isolates</th></tr>
<tr><td>Semaphore bulkhead</td><td>Caps concurrent calls; runs on the caller thread</td></tr>
<tr><td>Thread-pool bulkhead</td><td>Dedicated bounded pool and queue per dependency</td></tr>
</table>

<h3>Example</h3>

```java
@Service
class PricingClient {
    @CircuitBreaker(name = "pricing", fallbackMethod = "fallback")
    @Bulkhead(name = "pricing")
    public Price fetch(String sku) {
        return restClient.get().uri("/prices/{sku}", sku).retrieve().body(Price.class);
    }

    private Price fallback(String sku, Throwable t) {
        return Price.cached(sku);          // degrade gracefully
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      pricing:
        slidingWindowSize: 20
        failureRateThreshold: 50
        slowCallDurationThreshold: 2s
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
  bulkhead:
    instances:
      pricing:
        maxConcurrentCalls: 10
        maxWaitDuration: 0
```

<h3>Gotchas</h3>
<ul>
<li>Always set <strong>timeouts</strong> first; a breaker without timeouts never sees slow calls as failures.</li>
<li>Retries multiply load on a struggling service; use backoff with jitter and retry only idempotent calls. Annotation order matters (retry wraps circuit breaker by default).</li>
<li>Fallbacks must be cheap and safe; do not call another unreliable service from a fallback. Do not count business errors (4xx) as failures.</li>
<li>Monitor breaker state and rejected calls through Actuator/Micrometer metrics.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker/">Learn more: Spring Boot Circuit Breaker</a></p>

</details>

---

## 5. What is idempotency and why is it important in write APIs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>An operation is <strong>idempotent</strong> if performing it multiple times has the same effect as performing it once. It matters because networks fail and clients retry: without idempotency a retried <code>POST /payments</code> can charge twice.</p>

<h3>Context</h3>
<ul>
<li>HTTP: GET, PUT, DELETE are idempotent by definition; <strong>POST and (in general) PATCH are not</strong>.</li>
<li>Delivery guarantees are usually at-least-once. "Exactly-once" in practice means <strong>at-least-once delivery plus idempotent processing</strong>.</li>
</ul>

<h3>Idempotency Key Pattern</h3>
<ol>
<li>Client generates a unique key per logical operation and sends it (for example the <code>Idempotency-Key</code> header), reusing it on retries.</li>
<li>Server stores the key with the request fingerprint and the result, under a unique constraint.</li>
<li>On a repeat with the same key, return the stored response instead of re-executing.</li>
<li>Same key with a different payload returns an error (conflict/unprocessable).</li>
</ol>

```java
@PostMapping("/payments")
ResponseEntity<Payment> pay(@RequestHeader("Idempotency-Key") String key,
                            @RequestBody PaymentRequest req) {
    return service.payOnce(key, req);        // returns stored result if key was seen
}

@Transactional
ResponseEntity<Payment> payOnce(String key, PaymentRequest req) {
    Optional<StoredResult> prior = store.find(key);
    if (prior.isPresent()) return prior.get().toResponse();
    Payment p = execute(req);
    store.save(key, p);                      // unique index on key: concurrent duplicate fails
    return ResponseEntity.status(201).body(p);
}
```

<h3>Gotchas</h3>
<ul>
<li>Race condition: two concurrent requests with the same key; the DB unique constraint (or an atomic insert) must decide the winner.</li>
<li>Store the key and the business effect in the <strong>same transaction</strong>; expire keys after a retention window.</li>
<li>Natural alternatives: PUT with a client-chosen ID, conditional updates (version/ETag), upserts, and a dedupe table for message consumers (processed-message IDs).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/web-fundamentals/">Learn more: Web Fundamentals</a></p>

</details>

---

## 6. What is CQRS and when should you apply it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>CQRS (Command Query Responsibility Segregation)</strong> uses separate models for writing (commands that change state) and reading (queries that return data). Each side can be shaped, scaled, and stored independently.</p>

<h3>How It Works</h3>
<ul>
<li><strong>Command side:</strong> validates and applies business rules, writes to the write store, publishes events.</li>
<li><strong>Query side:</strong> a denormalized read model (a table, search index, or cache) built from those events and shaped exactly for the screens or APIs that read it.</li>
<li>Often paired with <strong>event sourcing</strong>, but the two are independent.</li>
</ul>

<h3>Example Skeleton</h3>

```java
// Write side
record PlaceOrder(String customerId, List<Item> items) { }
class OrderCommandHandler {
    void handle(PlaceOrder cmd) {
        Order order = Order.create(cmd);          // domain rules
        repository.save(order);
        events.publish(new OrderPlaced(order.id(), order.total()));
    }
}

// Read side: projection kept up to date by events
class OrderSummaryProjection {
    @EventListener
    void on(OrderPlaced e) { summaryStore.upsert(e.orderId(), e.total()); }
}
```

<h3>Pros and Cons</h3>
<table>
<tr><th>Benefits</th><th>Costs</th></tr>
<tr><td>Read and write scaled and optimized independently</td><td>Eventual consistency between write and read models</td></tr>
<tr><td>Simple, fast queries; no complex joins</td><td>More moving parts and code; projection rebuild and failure handling</td></tr>
<tr><td>Clean separation of business rules from reporting</td><td>Overkill for simple CRUD</td></tr>
</table>

<h3>When to Use</h3>
<ul>
<li>Read and write workloads or shapes differ a lot (heavy reporting, high read volume), complex domains, or when an audit trail is required.</li>
<li>Avoid it for plain CRUD. A lighter form is the same database with separate read DTOs/queries.</li>
<li>Handle "read your own writes" explicitly (return the result from the command, or poll/version).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging</a></p>

</details>

---
