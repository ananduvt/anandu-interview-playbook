# Database Transactions and Concurrency Questions

Verbal interview questions on transactions, isolation levels, locking and concurrent updates.

---

## 1. Discuss the importance of transactions and isolation levels in DB2

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A transaction groups statements into one unit that is atomic and consistent. Isolation levels control how much concurrent transactions can see of each other's work, trading integrity for concurrency.</p>

<h3>ACID</h3>
<ul>
<li><strong>Atomicity:</strong> all or nothing.</li>
<li><strong>Consistency:</strong> constraints remain valid.</li>
<li><strong>Isolation:</strong> concurrent transactions do not corrupt each other.</li>
<li><strong>Durability:</strong> committed data survives crashes (write-ahead logging).</li>
</ul>

<h3>Isolation Levels (standard SQL)</h3>
<table>
<tr><th>Level</th><th>Dirty read</th><th>Non-repeatable read</th><th>Phantom</th></tr>
<tr><td>READ UNCOMMITTED</td><td>Possible</td><td>Possible</td><td>Possible</td></tr>
<tr><td>READ COMMITTED</td><td>No</td><td>Possible</td><td>Possible</td></tr>
<tr><td>REPEATABLE READ</td><td>No</td><td>No</td><td>Possible (DB-specific)</td></tr>
<tr><td>SERIALIZABLE</td><td>No</td><td>No</td><td>No</td></tr>
</table>

<h3>DB2 Specifics</h3>
<ul>
<li>DB2 names its levels differently: <strong>UR</strong> (uncommitted read), <strong>CS</strong> (cursor stability, the default and close to read committed), <strong>RS</strong> (read stability), <strong>RR</strong> (repeatable read, closest to serializable).</li>
<li>Set per statement (<code>WITH UR</code>), per connection/package (bind option), or per session.</li>
<li>Higher levels hold more locks for longer, so lock waits, escalation and deadlocks increase.</li>
</ul>

<h3>Gotcha</h3>
<p>Standard-level behavior differs between vendors (for example, PostgreSQL REPEATABLE READ also prevents phantoms; Oracle has no READ UNCOMMITTED). State the vendor when answering.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 2. How do you decide which isolation level to use in a derivatives clearing application?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Start from the correctness requirement of each operation: use the weakest level that cannot produce a wrong financial result, and enforce critical invariants explicitly rather than relying on a global high level.</p>

<h3>Decision Guide</h3>
<table>
<tr><th>Operation</th><th>Suggested approach</th></tr>
<tr><td>Dashboards, reports, search</td><td>Read committed / cursor stability (or snapshot); stale-by-a-moment is fine</td></tr>
<tr><td>Position or margin update (read-modify-write)</td><td>Row locks (<code>SELECT ... FOR UPDATE</code>) or optimistic version check</td></tr>
<tr><td>Consistent end-of-day valuation or reconciliation</td><td>Repeatable read / snapshot for a stable view across many queries</td></tr>
<tr><td>Invariants spanning several rows (net exposure limit)</td><td>Serializable, or lock a parent row to serialize those updates</td></tr>
</table>

<h3>Trade-offs</h3>
<ul>
<li><strong>Higher isolation:</strong> fewer anomalies, but more blocking, deadlocks, serialization failures and lower throughput.</li>
<li><strong>Lower isolation:</strong> better concurrency, but risks lost updates and inconsistent reads.</li>
</ul>

<h3>Practices</h3>
<ul>
<li>Keep transactions short; access rows in a consistent order.</li>
<li>Retry on deadlock or serialization failure.</li>
<li>Test concurrent scenarios explicitly.</li>
</ul>

<h3>Gotcha</h3>
<p>Never choose READ UNCOMMITTED for money-affecting reads. Lost update is the classic bug at READ COMMITTED when you read, compute, then write.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 3. How do you handle lock contention in a high concurrency system?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Reduce how long and how many locks are held, avoid hot rows, and detect and recover from deadlocks.</p>

<h3>Techniques</h3>
<ul>
<li><strong>Short transactions:</strong> no user input or remote calls inside; commit promptly.</li>
<li><strong>Consistent lock ordering:</strong> always update accounts in the same order (e.g., by id) to avoid deadlocks.</li>
<li><strong>Good indexes:</strong> without an index, an UPDATE/DELETE may lock many more rows (or escalate).</li>
<li><strong>Lock granularity and escalation:</strong> many row locks can escalate to a table lock (DB2, SQL Server) and block everyone; commit in smaller batches and tune lock limits.</li>
<li><strong>Lowest suitable isolation level</strong>; MVCC databases (PostgreSQL, Oracle) let readers not block writers.</li>
<li><strong>Optimistic locking</strong> for low-conflict updates.</li>
<li><strong>Hot-row relief:</strong> shard counters, queue updates, or append-only ledger entries instead of updating one balance row.</li>
<li><strong>Timeouts and retries:</strong> set lock timeouts; retry on deadlock victim error with backoff.</li>
<li><strong>Skip locked:</strong> <code>FOR UPDATE SKIP LOCKED</code> for work queues.</li>
</ul>

<h3>Deadlock Detection</h3>
<p>The database detects cycles and aborts one transaction (the victim). Use deadlock logs/monitors to find the conflicting statements, then fix ordering or scope.</p>

<h3>Example</h3>

```sql
-- Consistent order: lock lower id first
SELECT * FROM accounts WHERE id IN (:a, :b) ORDER BY id FOR UPDATE;
```

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 4. How do you handle concurrent writes to the same record?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Prevent the lost update: two transactions read the same value, both modify it, and the second write overwrites the first. Use optimistic or pessimistic locking, or make the update atomic.</p>

<h3>Options</h3>
<table>
<tr><th>Approach</th><th>How</th><th>Best for</th></tr>
<tr><td>Optimistic locking</td><td>Version column; update fails if version changed</td><td>Low conflict, web apps</td></tr>
<tr><td>Pessimistic locking</td><td><code>SELECT ... FOR UPDATE</code> blocks others</td><td>High conflict, short critical sections</td></tr>
<tr><td>Atomic update</td><td><code>UPDATE ... SET x = x + 1</code></td><td>Counters, balances</td></tr>
<tr><td>Serializable isolation</td><td>DB aborts conflicting transaction</td><td>Complex invariants (be ready to retry)</td></tr>
</table>

<h3>Example (JPA optimistic locking)</h3>

```java
@Entity
class Account {
    @Id Long id;
    BigDecimal balance;
    @Version long version;   // incremented on every update
}
// A stale write throws ObjectOptimisticLockingFailureException -> retry or return 409
```

```sql
UPDATE accounts SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
-- check affected row count
```

<h3>Gotchas</h3>
<ul>
<li>Handle the conflict: retry with fresh data or tell the user.</li>
<li>Across multiple app instances, in-JVM <code>synchronized</code> does not help; the database (or a distributed lock) must arbitrate.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 5. How do you control concurrent updates in REST APIs?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use optimistic concurrency with a version carried through the HTTP contract (ETag and <code>If-Match</code>), backed by a version column in the database. Use pessimistic locking only for short, high-contention sections.</p>

<h3>Flow with ETags</h3>
<ol>
<li><code>GET /orders/42</code> returns the body plus <code>ETag: "7"</code> (the version).</li>
<li>Client sends <code>PUT /orders/42</code> with <code>If-Match: "7"</code>.</li>
<li>Server runs <code>UPDATE ... WHERE id=42 AND version=7</code>. If zero rows are updated, return <strong>412 Precondition Failed</strong> (or 409 Conflict).</li>
<li>Missing <code>If-Match</code> can be rejected with <strong>428 Precondition Required</strong>.</li>
</ol>

<h3>Example (Spring)</h3>

```java
@PutMapping("/orders/{id}")
ResponseEntity<Order> update(@PathVariable Long id,
        @RequestHeader("If-Match") long version,
        @RequestBody OrderUpdate req) {
    Order saved = service.update(id, version, req); // throws on version mismatch
    return ResponseEntity.ok().eTag("\"" + saved.getVersion() + "\"").body(saved);
}
```

<h3>Related Concerns</h3>
<ul>
<li><strong>Idempotency keys</strong> for POST so retries do not create duplicates.</li>
<li><strong>Pessimistic locking</strong> (<code>FOR UPDATE</code>) for hot rows, with timeouts.</li>
<li>Atomic SQL updates for counters; distributed locks only when the data is outside one database.</li>
</ul>

<h3>Gotcha</h3>
<p>Do not hold database locks across HTTP requests; the version check is what spans the read and the write.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---
