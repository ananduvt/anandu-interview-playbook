# Database Performance Questions

Verbal interview questions on query tuning, connection pooling and diagnosing slow queries.

---

## 1. How do you optimize a slow query?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Measure first, then fix the cause: read the execution plan, check indexes and row estimates, reduce the data touched, and re-measure.</p>

<h3>Step by Step</h3>
<ol>
<li><strong>Find it:</strong> slow query log, APM, <code>pg_stat_statements</code>, AWR.</li>
<li><strong>Read the plan:</strong> <code>EXPLAIN</code> / <code>EXPLAIN ANALYZE</code>. Look for full table scans, large sorts, nested loops over big sets, bad row estimates.</li>
<li><strong>Index:</strong> add or fix indexes for WHERE, JOIN and ORDER BY columns; use composite and covering indexes.</li>
<li><strong>Rewrite:</strong> select only needed columns, avoid <code>SELECT *</code>, filter early, paginate with keyset instead of large OFFSET.</li>
<li><strong>Avoid index killers:</strong> functions on indexed columns (<code>WHERE YEAR(d)=2024</code>), leading wildcards (<code>LIKE '%x'</code>), implicit type conversion.</li>
<li><strong>Update statistics</strong> so the optimizer chooses well.</li>
<li><strong>Last resorts:</strong> denormalize, cache, partition, add read replicas.</li>
</ol>

<h3>Example</h3>

```sql
-- Slow: function on the column prevents index use
SELECT * FROM orders WHERE DATE(created_at) = '2025-01-15';
-- Fast: sargable range
SELECT id, total FROM orders
WHERE created_at >= '2025-01-15' AND created_at < '2025-01-16';
```

<h3>Gotcha</h3>
<p>Every index slows writes; do not index blindly. Always verify with the plan, not by guessing.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 2. How to optimize database queries for high throughput APIs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Reduce the number of queries per request, make each one cheap, and protect the database from overload.</p>

<h3>Key Techniques</h3>
<ul>
<li><strong>Connection pooling:</strong> a right-sized pool (HikariCP) instead of opening connections per request.</li>
<li><strong>Detect N+1:</strong> one query for the list plus one per row. Fix with join fetch, batch fetching (<code>@BatchSize</code>), or <code>IN</code> queries. Enable SQL logging or Hibernate statistics in tests.</li>
<li><strong>Indexing:</strong> index the actual access paths; verify with <code>EXPLAIN</code>.</li>
<li><strong>Batching:</strong> JDBC batch inserts/updates (<code>hibernate.jdbc.batch_size</code>).</li>
<li><strong>Pagination and projections:</strong> return only needed columns (DTO projections) and limit result sizes.</li>
<li><strong>Caching:</strong> cache hot reads (Redis, second-level cache) with clear invalidation.</li>
<li><strong>Read replicas:</strong> route read-only traffic away from the primary.</li>
<li><strong>Short transactions:</strong> do no remote calls while holding a transaction open.</li>
</ul>

<h3>Example (N+1 fix in JPA)</h3>

```java
@Query("select o from Order o join fetch o.items where o.customerId = :id")
List<Order> findWithItems(@Param("id") Long id);
```

<h3>Gotcha</h3>
<p>Join-fetching several collections causes a cartesian product; fetch collections separately or use batch size.</p>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging & Performance</a></p>

</details>

---

## 3. What are connection pooling best practices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A connection pool reuses a fixed set of open database connections, avoiding the high cost of connect/authenticate per request. HikariCP is the default in Spring Boot.</p>

<h3>Best Practices</h3>
<ul>
<li><strong>Size small:</strong> more connections do not mean more throughput. A common starting point is <code>(core_count * 2) + effective_spindle_count</code>, then load test. Total across all app instances must stay under the database's max connections.</li>
<li><strong>Fixed-size pool:</strong> set <code>minimumIdle</code> equal to <code>maximumPoolSize</code> for steady load (HikariCP's recommendation).</li>
<li><strong>Timeouts:</strong> <code>connectionTimeout</code> (fail fast when exhausted), <code>maxLifetime</code> a few minutes shorter than any DB/firewall limit, <code>idleTimeout</code>.</li>
<li><strong>Always release connections:</strong> try-with-resources; avoid long transactions and remote calls inside them.</li>
<li><strong>Leak detection:</strong> <code>leakDetectionThreshold</code>.</li>
<li><strong>Monitor:</strong> active, idle, pending threads and acquire time via Micrometer/Actuator (<code>hikaricp.connections.*</code>).</li>
</ul>

<h3>Example</h3>

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      connection-timeout: 3000
      max-lifetime: 1500000
      leak-detection-threshold: 20000
```

<h3>Gotcha</h3>
<p>Symptoms of exhaustion: threads waiting on <code>getConnection</code>, timeouts under load. The cause is usually slow queries or long transactions, not too small a pool.</p>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging & Performance</a></p>

</details>

---

## 4. How do you optimize SQL queries for performance in a high-volume financial transaction system like DB2?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The principles are the same as for any relational database: understand the optimizer's access plan, give it good indexes and statistics, and minimize data and locks per transaction.</p>

<h3>Techniques</h3>
<ul>
<li><strong>Explain plans:</strong> use <code>EXPLAIN</code> (Visual Explain, <code>db2expln</code>) to spot table scans, expensive sorts and join order issues.</li>
<li><strong>Statistics:</strong> keep them current (<code>RUNSTATS</code>) and rebind static SQL packages afterwards.</li>
<li><strong>Indexing:</strong> composite and covering indexes matching predicates and ORDER BY; drop unused indexes to speed writes.</li>
<li><strong>Fewer, narrower joins:</strong> select only needed columns; filter early; consider summary tables or materialized query tables (MQTs) for reporting.</li>
<li><strong>Parameterized statements:</strong> use host variables/prepared statements so plans are reused and are injection-safe.</li>
<li><strong>Batching:</strong> multi-row inserts and batched commits for bulk work.</li>
<li><strong>Stored procedures:</strong> for multi-step logic, to cut network round trips.</li>
<li><strong>Partitioning:</strong> range-partition large tables by date so queries prune partitions.</li>
<li><strong>Keep transactions short</strong> and access rows in a consistent order to reduce lock contention.</li>
<li><strong>Isolation:</strong> use the lowest level that is still correct.</li>
</ul>

<h3>Gotcha</h3>
<p>Optimization is measure-driven: baseline, change one thing, compare plans and timings under realistic data volume.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 5. Can you describe a scenario where query optimization significantly improved performance?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Structure the story as: symptom, diagnosis, fix, measured result. Below is a generic example you can adapt to your own experience (use real numbers you can defend).</p>

<h3>Example Story</h3>
<ul>
<li><strong>Symptom:</strong> an order-history endpoint degraded from about 200 ms to several seconds as the table grew to tens of millions of rows.</li>
<li><strong>Diagnosis:</strong> slow query log showed one query; <code>EXPLAIN</code> showed a full table scan plus a filesort. The app also issued an N+1 pattern for line items.</li>
<li><strong>Fix:</strong>
 <ul>
 <li>Added a composite index <code>(customer_id, created_at DESC)</code> matching filter and sort.</li>
 <li>Replaced the per-row item lookups with one join fetch.</li>
 <li>Replaced <code>OFFSET</code> pagination with keyset pagination.</li>
 </ul>
</li>
<li><strong>Result:</strong> back to tens of milliseconds, with far lower database CPU. Verified before and after with the plan and load test.</li>
</ul>

<h3>Tips for the Interview</h3>
<ul>
<li>Show the method (measure, plan, change, re-measure), not just the trick.</li>
<li>Mention trade-offs (extra index write cost) and how you guarded against regressions (monitoring, tests).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 6. How do you monitor and diagnose slow queries in DB2?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Combine built-in monitoring (monitoring table functions and event monitors) with explain tools, then act on the worst statements by total cost.</p>

<h3>Tools</h3>
<ul>
<li><strong>Monitoring table functions:</strong> <code>MON_GET_PKG_CACHE_STMT</code> lists statements in the package cache with execution counts, CPU time, rows read vs returned, lock wait time. Sort by total time to find top offenders.</li>
<li><strong>Event monitors:</strong> activity or statement event monitors capture individual executions, and lock/deadlock event monitors capture contention.</li>
<li><strong>Explain:</strong> <code>EXPLAIN</code> statement, <code>db2expln</code>, <code>db2exfmt</code>, or Data Studio Visual Explain.</li>
<li><strong>Advisor:</strong> <code>db2advis</code> suggests indexes.</li>
<li><strong>Statistics:</strong> check <code>RUNSTATS</code> freshness; stale stats cause bad plans.</li>
<li><strong>Application side:</strong> APM and slow query timing in the pool/driver.</li>
</ul>

<h3>What to Look For</h3>
<ul>
<li>High rows read vs rows returned (missing index).</li>
<li>Table scans, sort overflows, long lock waits.</li>
<li>Plans changed after data growth or a rebind.</li>
</ul>

<h3>Equivalents</h3>
<p>PostgreSQL: <code>pg_stat_statements</code>, <code>auto_explain</code>. MySQL: slow query log, <code>performance_schema</code>. Oracle: AWR, SQL Monitor.</p>

<h3>Gotcha</h3>
<p>Exact view and function names vary by DB2 version and platform (LUW vs z/OS); check the documentation for your release.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---
