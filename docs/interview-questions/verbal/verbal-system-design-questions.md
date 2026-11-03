# System Design Interview Questions

Designing large-scale systems: scalability, reliability, performance, and trade-offs.

---

## 1. How do you approach a system design problem?

<details>
<summary>View Answer</summary>

<h3>Step-by-Step Approach</h3>

<ol>
<li><strong>Clarify requirements (functional)</strong>
  <ul>
  <li>What features? Scale? User base?</li>
  </ul>
</li>
<li><strong>Non-functional requirements (NFR)</strong>
  <ul>
  <li>Latency? Throughput? Availability? Consistency?</li>
  </ul>
</li>
<li><strong>Back-of-envelope calculations</strong>
  <ul>
  <li>QPS, bandwidth, storage needed</li>
  </ul>
</li>
<li><strong>High-level architecture</strong>
  <ul>
  <li>Database, cache, queue, load balancer</li>
  </ul>
</li>
<li><strong>Dive into components</strong>
  <ul>
  <li>Database schema, API design, scaling strategy</li>
  </ul>
</li>
<li><strong>Discuss trade-offs</strong>
  <ul>
  <li>Why this choice? Alternatives?</li>
  </ul>
</li>
</ol>

<h3>Recommendation: Start monolithic, scale when needed</h3>

<p>📚 <a href="/anandu-interview-playbook/system-design/worked-examples/">Learn more: System Design Examples</a></p>

</details>

---

## 2. How would you approach diagnosing intermittent latency spikes in a production clearing system?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Work systematically from symptom to cause: define and quantify the spike, find the pattern (when, where, which requests), narrow down the layer with metrics and traces, then confirm with profiling or query analysis. Do not guess or change several things at once.</p>

<h3>Step by Step</h3>
<ul>
<li><strong>1. Scope it:</strong> which endpoints, which percentile (p99 vs p50), start time, duration, how often. Check recent deployments, config changes and traffic changes.</li>
<li><strong>2. Look for a pattern:</strong> periodic (cron jobs, GC, cache expiry, batch window), load-correlated, per instance, per tenant, or tied to certain payloads.</li>
<li><strong>3. Locate the layer:</strong> use distributed traces to see which span grew (app, DB, downstream call, queue wait).</li>
<li><strong>4. Check the usual suspects:</strong>
<ul>
<li>JVM: GC pauses, safepoints, heap pressure, thread pool exhaustion, lock contention.</li>
<li>Database: slow queries, lock waits, connection pool wait, missing index or plan change, replication lag.</li>
<li>Downstream and network: slow dependency, retries, DNS, TCP retransmits, TLS handshakes.</li>
<li>Infrastructure: CPU throttling in containers, noisy neighbors, disk I/O, autoscaling cold starts.</li>
</ul></li>
<li><strong>5. Confirm:</strong> take a thread dump during the spike, a profile or flame graph, and the slow query log.</li>
<li><strong>6. Fix, then verify:</strong> change one thing, check the p99 on the dashboard, add an alert, and write it up.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Averages hide spikes, so use percentiles and heatmaps.</li>
<li>Intermittent problems need continuous data (always-on metrics, tracing, low-overhead profiling like JFR), because you cannot reproduce them on demand.</li>
<li>Beware coordinated omission when load testing, which understates latency.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/observability/">Learn more: Observability</a></p>

</details>

---

## 3. How would you prioritize investigative steps in production issue diagnosis?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>First stabilize, then investigate. Mitigate customer impact (rollback, failover, scale out, disable a feature flag) before you find the root cause. Then rank hypotheses by impact, likelihood and cost of checking, and test the cheap and probable ones first.</p>

<h3>Prioritization Factors</h3>
<table>
<tr><th>Factor</th><th>Question</th></tr>
<tr><td>Impact</td><td>How many users or how much revenue is affected? Is data at risk?</td></tr>
<tr><td>Probability</td><td>How likely is this cause given the evidence?</td></tr>
<tr><td>Ease</td><td>How quickly can I check it? Is it safe?</td></tr>
<tr><td>Reversibility</td><td>Can I undo the mitigation easily?</td></tr>
</table>

<h3>Practical Order</h3>
<ul>
<li><strong>1. What changed?</strong> Deployment, config, feature flag, traffic, dependency, certificate expiry. This is the highest-yield question.</li>
<li><strong>2. Dashboards and alerts:</strong> error rate, latency, saturation (CPU, memory, pools, queues).</li>
<li><strong>3. Logs and traces</strong> around the first occurrence.</li>
<li><strong>4. Dependencies:</strong> database, cache, broker, downstream APIs, network.</li>
<li><strong>5. Deep inspection</strong> (thread dumps, heap dumps, profiling) when the cheap checks do not explain it.</li>
</ul>

<h3>Process Tips</h3>
<ul>
<li>Declare an incident owner, keep a timeline, and communicate status regularly.</li>
<li>Keep a hypothesis list and cross each one off with evidence. Avoid tunnel vision.</li>
<li>Capture evidence (dumps, logs) before restarting, because a restart destroys it.</li>
<li>Finish with a blameless post-mortem, action items and better alerts.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/devops/observability/">Learn more: Observability</a></p>

</details>

---

## 4. What tools would you use to pinpoint performance issues in a Java system?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Match the tool to the layer: metrics and traces to find where, JVM tools to see what the process is doing, database tools for slow queries, and network tools for connectivity and latency.</p>

<h3>Toolbox by Layer</h3>
<table>
<tr><th>Layer</th><th>Tools</th></tr>
<tr><td>Metrics and dashboards</td><td>Micrometer, Prometheus, Grafana, APM agents</td></tr>
<tr><td>Tracing and logs</td><td>OpenTelemetry, Zipkin or Jaeger, ELK or Loki with trace IDs</td></tr>
<tr><td>JVM profiling</td><td>Java Flight Recorder (JFR) with JDK Mission Control, async-profiler (flame graphs), VisualVM</td></tr>
<tr><td>Threads and memory</td><td><code>jstack</code> or <code>jcmd Thread.print</code> (thread dumps), <code>jmap</code> or <code>jcmd GC.heap_dump</code>, Eclipse MAT</td></tr>
<tr><td>GC</td><td>GC logs (<code>-Xlog:gc*</code>), GCViewer or GCeasy, <code>jstat</code></td></tr>
<tr><td>Database</td><td><code>EXPLAIN ANALYZE</code>, slow query log, <code>pg_stat_statements</code>, connection pool metrics (HikariCP)</td></tr>
<tr><td>Network and OS</td><td><code>curl -w</code>, <code>tcpdump</code> or Wireshark, <code>top</code>, <code>iostat</code>, <code>ss</code></td></tr>
<tr><td>Load testing</td><td>JMeter, Gatling, k6</td></tr>
</table>

<h3>Typical Workflow</h3>
<ul>
<li>Dashboard shows p99 rising, then a trace shows time inside a DB span.</li>
<li>The slow query log and <code>EXPLAIN</code> show a missing index.</li>
<li>If the time is in the app, take 3 thread dumps a few seconds apart (<code>jcmd PID Thread.print</code>) to find blocked or busy threads, then a JFR or async-profiler flame graph to find hot methods.</li>
</ul>

```bash
jcmd <pid> JFR.start duration=60s filename=/tmp/spike.jfr settings=profile
jcmd <pid> Thread.print > threads-1.txt
```

<h3>Gotchas</h3>
<ul>
<li>JFR has low overhead and is safe in production. Heap dumps pause the JVM and are large.</li>
<li>Thread dumps taken once are weak evidence. Take several and compare.</li>
<li>Profile under realistic load, and keep the sampling overhead low.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging & Performance</a></p>

</details>

---

## 5. How would you design a backend for 1M users?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Clarify what "1M users" means (registered, daily active, concurrent), turn it into requests per second, then scale in steps: stateless app servers behind a load balancer, a primary database with read replicas, a cache for hot reads, and queues for slow work. Add sharding only when measurements demand it.</p>

<h3>Why / Context</h3>
<p>One million registered users is modest. For example, 100k daily actives making 50 requests each is about 5M requests per day, roughly 60 requests per second on average and maybe 5 to 10 times that at peak. A well-tuned single database handles this, so the design goal is availability and headroom more than extreme scale.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Concern</th><th>Approach</th></tr>
<tr><td>Compute</td><td>Stateless services (session in token or Redis), horizontal autoscaling, health checks</td></tr>
<tr><td>Traffic</td><td>Load balancer, CDN for static content, rate limiting</td></tr>
<tr><td>Data</td><td>Primary plus read replicas, proper indexes, connection pooling; partition or shard later</td></tr>
<tr><td>Latency</td><td>Cache-aside with Redis, HTTP caching, pagination</td></tr>
<tr><td>Async work</td><td>Queue (Kafka, SQS) for email, reports, notifications; idempotent consumers</td></tr>
<tr><td>Resilience</td><td>Timeouts, retries with backoff, circuit breakers, multi-AZ deployment</td></tr>
<tr><td>Operations</td><td>Metrics, tracing, alerts, load tests, backups and restore drills</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```
Client -> CDN -> Load balancer -> App instances (stateless, N replicas)
                                   |-> Redis (cache, sessions)
                                   |-> Primary DB -> read replicas
                                   |-> Queue -> workers (async jobs)
```

<h3>When to Use</h3>
<ul>
<li>Start simple (modular monolith or a few services) and evolve; do not shard or split services before you have evidence.</li>
<li>Likely follow-ups: where is the bottleneck first (usually the database), how to handle replica lag, how to deploy without downtime, what to do at 10x.</li>
<li>State assumptions and trade-offs out loud; it matters more than the exact diagram.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/scalability-resilience/">Learn more: Scalability & Resilience</a></p>

</details>

---

## 6. Theory recall: ACID vs BASE, CAP, SOLID, idempotency, delivery guarantees

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>These are the foundational ideas behind most design discussions. Know the definition, the trade-off, and one example of each.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Concept</th><th>Meaning</th><th>Example or trade-off</th></tr>
<tr><td>ACID</td><td>Atomicity, Consistency, Isolation, Durability for transactions</td><td>Bank transfer: both updates commit or neither</td></tr>
<tr><td>BASE</td><td>Basically Available, Soft state, Eventually consistent</td><td>Social feed counters; favors availability and scale</td></tr>
<tr><td>CAP</td><td>During a network partition choose consistency (reject or block requests) or availability (serve possibly stale data)</td><td>CP: coordination stores. AP: Cassandra-style stores. Partitions are not optional in distributed systems.</td></tr>
<tr><td>SOLID</td><td>Single responsibility, Open/closed, Liskov, Interface segregation, Dependency inversion</td><td>Inject an interface, not a concrete class</td></tr>
<tr><td>DRY / KISS / YAGNI</td><td>Do not duplicate knowledge; keep it simple; do not build what you do not need yet</td><td>DRY misapplied couples unrelated code</td></tr>
<tr><td>Idempotency</td><td>Repeating an operation gives the same result as doing it once</td><td>PUT and DELETE; make POST safe with an idempotency key</td></tr>
<tr><td>At-least-once</td><td>Never lose a message, may deliver duplicates</td><td>Pair with idempotent consumers (the usual default)</td></tr>
<tr><td>At-most-once</td><td>No duplicates, may lose messages</td><td>Fine for non-critical metrics</td></tr>
<tr><td>Exactly-once</td><td>Each message has its effect once</td><td>In practice at-least-once plus dedupe or transactions (for example Kafka transactions within Kafka)</td></tr>
</table>

<h3>Example / Real-World Use</h3>

```java
// Idempotent consumer: at-least-once delivery, exactly-once effect
void handle(PaymentEvent e) {
    if (!processedIds.add(e.id())) return;   // duplicate, ignore (use a DB unique key in real code)
    applyPayment(e);
}
```

<h3>When to Use</h3>
<ul>
<li>CAP is about behaviour during a partition, not a permanent choice of two of three; say that clearly.</li>
<li>Choose ACID for money and inventory, BASE where staleness is acceptable and scale dominates.</li>
<li>Gotcha: the "C" in ACID (valid state) is not the "C" in CAP (all nodes see the same data).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/fundamentals/">Learn more: System Design Fundamentals</a></p>

</details>

---

**See also:** [System Design Fundamentals](../../system-design/fundamentals.md), [Building Blocks](../../system-design/building-blocks.md), [Interview Preparation](../index.md)
