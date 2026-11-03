# Microservices Resilience Interview Questions

Resilience patterns for distributed systems: timeouts, retries, circuit breakers, bulkheads and handling network failures.

---

## 1. What are resilience patterns and why are they needed?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Resilience patterns keep a system working, or degrade gracefully, when a dependency is slow or failing. In a distributed system partial failure is normal, and without protection one slow service can exhaust threads in its callers and cause a cascading failure.</p>

<h3>The Core Patterns</h3>
<table>
<tr><th>Pattern</th><th>What it does</th><th>Watch out for</th></tr>
<tr><td>Timeout</td><td>Stops waiting after a limit</td><td>Must be set on every remote call (connect and read)</td></tr>
<tr><td>Retry (with backoff and jitter)</td><td>Re-attempts transient failures</td><td>Only for idempotent calls, can cause retry storms</td></tr>
<tr><td>Circuit breaker</td><td>Fails fast after repeated failures, probes for recovery</td><td>Needs sensible thresholds and a fallback</td></tr>
<tr><td>Bulkhead</td><td>Isolates resources per dependency</td><td>Pool sizing</td></tr>
<tr><td>Rate limiter</td><td>Caps request rate</td><td>Choose per client or global</td></tr>
<tr><td>Fallback</td><td>Cached or default response</td><td>Stale data must be acceptable</td></tr>
</table>

<h3>How They Combine</h3>
<p>A typical order of decoration, outer to inner: Bulkhead, then Circuit Breaker, then Retry, then Timeout, around the call. Timeouts bound each attempt, retries handle blips, the breaker stops hammering a dead service, and the bulkhead limits the blast radius.</p>

<h3>Example</h3>

```java
@Bulkhead(name = "inventory")
@CircuitBreaker(name = "inventory", fallbackMethod = "cached")
@Retry(name = "inventory")
public Stock getStock(String sku) { return client.fetch(sku); }
```

<h3>Gotchas</h3>
<ul>
<li>Retries multiply load: 3 layers each retrying 3 times means 27 calls. Retry at one layer only.</li>
<li>Combine with load shedding, graceful degradation and health checks.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 2. How do you implement the circuit breaker pattern with Resilience4j?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Resilience4j is a lightweight fault-tolerance library, the replacement for Netflix Hystrix (in maintenance mode). In Spring Boot add the <code>resilience4j-spring-boot3</code> starter plus AOP, configure an instance in YAML, and annotate the calling method with <code>@CircuitBreaker</code>.</p>

<h3>Configuration</h3>

```yaml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 20
        minimumNumberOfCalls: 10
        failureRateThreshold: 50          # open when 50% of calls fail
        slowCallDurationThreshold: 2s
        slowCallRateThreshold: 80
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
```

<h3>Usage</h3>

```java
@CircuitBreaker(name = "payment", fallbackMethod = "fallback")
public Receipt charge(Order o) { return gateway.charge(o); }

// Same arguments plus the exception; same return type
private Receipt fallback(Order o, Throwable t) { return Receipt.pending(o.id()); }
```

<h3>States</h3>
<ul>
<li><strong>CLOSED:</strong> calls pass, failures and slow calls are recorded in the sliding window.</li>
<li><strong>OPEN:</strong> calls are rejected immediately with <code>CallNotPermittedException</code>.</li>
<li><strong>HALF_OPEN:</strong> a few trial calls decide whether to close or re-open.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Use <code>ignoreExceptions</code> for business errors (e.g. 404, validation) so they do not trip the breaker.</li>
<li>The annotation works through a Spring proxy, so self-invocation inside the same class bypasses it.</li>
<li>Expose state via Actuator and Micrometer metrics, and alert on open breakers.</li>
<li>The fallback signature must match, and it should not call the same failing dependency.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker/">Learn more: Circuit Breaker Patterns</a></p>

</details>

---

## 3. Explain the bulkhead pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The bulkhead pattern isolates resources (threads, connections, concurrent-call permits) into separate pools per dependency, like watertight compartments in a ship. If one dependency becomes slow and fills its pool, the others keep working.</p>

<h3>The Problem It Solves</h3>
<p>Without isolation, a slow downstream consumes all threads of a shared pool (for example the Tomcat or HTTP client pool). Then unrelated endpoints that never call that dependency also stall.</p>

<h3>Types in Resilience4j</h3>
<table>
<tr><th>Type</th><th>How it limits</th><th>Notes</th></tr>
<tr><td>Semaphore bulkhead</td><td>Max concurrent calls on the caller thread</td><td>Cheap, default, works with reactive code</td></tr>
<tr><td>Thread-pool bulkhead</td><td>Dedicated thread pool plus queue</td><td>Stronger isolation and timeouts, but extra thread hops and context switches</td></tr>
</table>

```yaml
resilience4j:
  bulkhead:
    instances:
      inventory:
        maxConcurrentCalls: 10
        maxWaitDuration: 100ms   # reject quickly when full
```

<h3>Other Forms</h3>
<ul>
<li>Separate connection pools per downstream service.</li>
<li>Separate deployments or pods for critical vs batch workloads.</li>
<li>Kubernetes resource limits and quotas, per-tenant isolation.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Pools that are too small reject healthy traffic, and too large give no isolation. Size from measured concurrency and latency (Little's law).</li>
<li>Pair with a timeout, otherwise permits are held forever by stuck calls.</li>
<li>Bulkhead limits blast radius, and a circuit breaker stops the calls. They are complementary.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 4. What retry strategy do you use when calling third-party APIs or external services?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Retry only transient failures, only on idempotent operations, with a bounded number of attempts, <strong>exponential backoff with jitter</strong>, and an overall deadline. Combine with a timeout per attempt and a circuit breaker.</p>

<h3>What to Retry</h3>
<table>
<tr><th>Retry</th><th>Do not retry</th></tr>
<tr><td>Connection or read timeouts, 502, 503, 504</td><td>400, 401, 403, 404, 422 (client errors)</td></tr>
<tr><td>429 Too Many Requests (honor <code>Retry-After</code>)</td><td>Business validation failures</td></tr>
<tr><td>GET, PUT, DELETE (idempotent)</td><td>Non-idempotent POST, unless an idempotency key is used</td></tr>
</table>

<h3>Why Jitter</h3>
<p>Without jitter all clients retry at the same instants after an outage and create a thundering herd. Backoff such as 200ms, 400ms, 800ms plus random jitter spreads the load.</p>

<h3>Example</h3>

```yaml
resilience4j:
  retry:
    instances:
      thirdParty:
        maxAttempts: 4
        waitDuration: 200ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        enableRandomizedWait: true
        retryExceptions: [java.io.IOException, java.util.concurrent.TimeoutException]
```

<h3>Gotchas</h3>
<ul>
<li>For payments and other POSTs, send an <strong>idempotency key</strong> so a retry cannot double-charge.</li>
<li>Retry at one layer only, to avoid multiplicative retries.</li>
<li>After attempts are exhausted, use a fallback, queue the work for later (retry topic or outbox) or fail clearly.</li>
<li>Respect the vendor's rate limits and monitor retry counts.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker/">Learn more: Circuit Breaker Patterns</a></p>

</details>

---

## 5. How do you handle network failures in distributed systems?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Assume the network is unreliable. Make every call bounded (timeouts), recoverable (retries with idempotency), isolated (circuit breaker and bulkhead) and observable. Design for degraded operation instead of all-or-nothing.</p>

<h3>Failure Modes</h3>
<ul>
<li>Request lost, response lost, or very slow (you cannot tell a slow service from a dead one).</li>
<li>Partial failure: the operation succeeded but the ack was lost, so a retry may duplicate it.</li>
<li>Partitions: some nodes cannot reach others, so choose consistency or availability (CAP).</li>
</ul>

<h3>Toolbox</h3>
<table>
<tr><th>Technique</th><th>Purpose</th></tr>
<tr><td>Timeouts and deadlines</td><td>Never wait forever</td></tr>
<tr><td>Retry with backoff and jitter</td><td>Survive transient glitches</td></tr>
<tr><td>Idempotency keys, deduplication</td><td>Make retries safe</td></tr>
<tr><td>Circuit breaker, bulkhead</td><td>Stop cascading failures</td></tr>
<tr><td>Fallbacks, caching</td><td>Degrade gracefully</td></tr>
<tr><td>Async messaging, outbox</td><td>Decouple in time, buffer work during outages</td></tr>
<tr><td>Health checks, load balancing</td><td>Route around bad instances</td></tr>
<tr><td>Redundancy across zones</td><td>Survive zone or node loss</td></tr>
</table>

<h3>Example Scenario</h3>
<p>Order service calls Payment, the call times out and the outcome is unknown. Retry with the same idempotency key. If still unknown, mark the order PENDING and reconcile later by querying the payment status or consuming its event, and do not assume failure.</p>

<h3>Gotchas</h3>
<ul>
<li>Test failures deliberately (chaos testing, fault injection, latency injection).</li>
<li>Set timeouts shorter than the caller's own timeout, so deadlines shrink down the chain.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/scalability-resilience/">Learn more: Scalability & Resilience</a></p>

</details>

---
