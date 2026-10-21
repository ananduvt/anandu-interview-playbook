# Databases & SQL Interview Questions

Conceptual questions on database design, SQL queries, transactions, optimization, and best practices.

---

## 1. What's the difference between INNER JOIN, LEFT JOIN, and UNION?

<details>
<summary>View Answer</summary>

<h3>INNER JOIN</h3>

<p>Returns rows that exist in BOTH tables (intersection).</p>

```sql
SELECT o.id, c.name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;
```

<p><strong>Result:</strong> Only shows customers WITH orders</p>

<h3>LEFT JOIN</h3>

<p>Returns ALL rows from left table + matching rows from right table.</p>

```sql
SELECT o.id, c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

<p><strong>Result:</strong> All customers shown. Non-matching orders have NULL values.</p>

<h3>UNION</h3>

<p>Combines result sets from multiple queries, removes duplicates.</p>

```sql
SELECT name FROM customers
UNION
SELECT name FROM vendors;
-- Result: All unique names from both tables
```

<p><strong>UNION ALL:</strong> Like UNION but keeps duplicates (faster).</p>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Operation</th>
<th>Result</th>
<th>Duplicates</th>
<th>Use Case</th>
</tr>
<tr>
<td><strong>INNER JOIN</strong></td>
<td>Matching rows only</td>
<td>N/A (join)</td>
<td>Data in both tables</td>
</tr>
<tr>
<td><strong>LEFT JOIN</strong></td>
<td>All left + matching right</td>
<td>N/A (join)</td>
<td>All left + optional right</td>
</tr>
<tr>
<td><strong>UNION</strong></td>
<td>Combined result</td>
<td>Removed</td>
<td>Combining similar datasets</td>
</tr>
<tr>
<td><strong>UNION ALL</strong></td>
<td>Combined result</td>
<td>Kept</td>
<td>Combining, performance matters</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 2. What's the difference between WHERE and HAVING?

<details>
<summary>View Answer</summary>

<h3>Quick Comparison</h3>

<table>
<tr>
<th>Clause</th>
<th>Filters</th>
<th>When Applied</th>
<th>Can Use Aggregates</th>
</tr>
<tr>
<td><strong>WHERE</strong></td>
<td>Individual rows</td>
<td>BEFORE grouping</td>
<td>❌ No</td>
</tr>
<tr>
<td><strong>HAVING</strong></td>
<td>Groups</td>
<td>AFTER grouping/aggregation</td>
<td>✅ Yes</td>
</tr>
</table>

<h3>Example</h3>

```sql
SELECT customer_id, COUNT(*) as order_count
FROM orders
WHERE order_date > '2024-01-01'      -- Filters ROWS before grouping
GROUP BY customer_id
HAVING COUNT(*) > 5;                 -- Filters GROUPS after aggregation
-- Result: Customers with >5 orders since 2024-01-01
```

<h3>Key Point</h3>

<p><strong>Cannot use aggregate functions in WHERE:</strong></p>

```sql
-- ❌ INVALID
SELECT customer_id FROM orders
WHERE COUNT(*) > 5
GROUP BY customer_id;

-- ✅ VALID
SELECT customer_id FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

<h3>Performance Tip</h3>

<p>Use WHERE whenever possible (filters before grouping, faster). Use HAVING only for aggregate filtering.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 3. What are common database caching patterns?

<details>
<summary>View Answer</summary>

<h3>1. Cache-Aside (Lazy Loading)</h3>

<p><strong>Flow:</strong> App checks cache → if miss, fetch DB → populate cache</p>

```
Request → Check Cache → Miss? → Query DB → Store in Cache → Return
```

<p><strong>Pros:</strong> Simple, read-heavy workloads, only needed data cached</p>
<p><strong>Cons:</strong> Cache misses cause DB hits, stale data possible</p>

<h3>2. Read-Through</h3>

<p><strong>Flow:</strong> Cache library handles DB fetching automatically</p>

```
Request → Cache (handles DB fetch internally) → Return
```

<p><strong>Pros:</strong> Transparent to app, consistent logic</p>
<p><strong>Cons:</strong> More complex, tight coupling with cache library</p>

<h3>3. Write-Through</h3>

<p><strong>Flow:</strong> Write to cache AND DB simultaneously</p>

```
Write Request → Cache → DB (synchronous) → Return
```

<p><strong>Pros:</strong> Data consistency guaranteed</p>
<p><strong>Cons:</strong> Slower writes, must wait for DB</p>

<h3>4. Write-Behind (Write-Back)</h3>

<p><strong>Flow:</strong> Write to cache immediately, write to DB asynchronously</p>

```
Write Request → Cache → Return (async write to DB)
```

<p><strong>Pros:</strong> Fast writes, good for write-heavy workloads</p>
<p><strong>Cons:</strong> Data loss risk if cache crashes before DB write</p>

<h3>Interview Favorite</h3>

<p><strong>Cache-Aside with Redis</strong> is the most common pattern for interviews. It's simple, proven, and widely used.</p>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/caching/">Learn more: Caching</a></p>

</details>

---

## 4. Explain the CAP theorem and its implications

<details>
<summary>View Answer</summary>

<h3>The CAP Theorem</h3>

<p>In a distributed system, you can guarantee only <strong>2 of 3</strong> properties:</p>

<ul>
<li><strong>Consistency (C):</strong> All nodes see the same data at the same time</li>
<li><strong>Availability (A):</strong> System always responds to requests</li>
<li><strong>Partition Tolerance (P):</strong> System works despite network partitions</li>
</ul>

<h3>Trade-offs in Practice</h3>

<table>
<tr>
<th>Type</th>
<th>Consistency</th>
<th>Availability</th>
<th>Partition Tolerance</th>
<th>Examples</th>
</tr>
<tr>
<td><strong>CA</strong></td>
<td>✅</td>
<td>✅</td>
<td>❌</td>
<td>Traditional relational DBs</td>
</tr>
<tr>
<td><strong>CP</strong></td>
<td>✅</td>
<td>❌</td>
<td>✅</td>
<td>HBase, Zookeeper, MongoDB</td>
</tr>
<tr>
<td><strong>AP</strong></td>
<td>❌</td>
<td>✅</td>
<td>✅</td>
<td>Cassandra, DynamoDB, Riak</td>
</tr>
</table>

<h3>Key Insight</h3>

<p><strong>Network partitions ALWAYS happen.</strong> Modern distributed systems must choose between:</p>

<ul>
<li><strong>CP:</strong> Reject writes during partition (consistent but unavailable)</li>
<li><strong>AP:</strong> Accept writes, reconcile later (available but inconsistent)</li>
</ul>

<h3>Real-World Implications</h3>

<ul>
<li><strong>Banking (CP):</strong> Cannot afford inconsistency; accept downtime</li>
<li><strong>Social Media (AP):</strong> Can tolerate eventual consistency; prioritize availability</li>
<li><strong>E-commerce (AP with backup):</strong> Availability first, eventual consistency</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/fundamentals/">Learn more: System Design Fundamentals</a></p>

</details>

---

## 5. What's the difference between ACID and BASE?

<details>
<summary>View Answer</summary>

<h3>ACID (Traditional Databases)</h3>

<table>
<tr>
<th>Property</th>
<th>Meaning</th>
<th>Example</th>
</tr>
<tr>
<td><strong>Atomicity</strong></td>
<td>Transaction all-or-nothing (no partial updates)</td>
<td>Transfer $100: debit one, credit other (both or neither)</td>
</tr>
<tr>
<td><strong>Consistency</strong></td>
<td>Database always in valid state</td>
<td>Foreign key constraints always satisfied</td>
</tr>
<tr>
<td><strong>Isolation</strong></td>
<td>Concurrent transactions don't interfere</td>
<td>Two simultaneous transfers don't corrupt data</td>
</tr>
<tr>
<td><strong>Durability</strong></td>
<td>Committed data persists (crash-safe)</td>
<td>Committed transaction survives power failure</td>
</tr>
</table>

<h3>BASE (NoSQL, Distributed)</h3>

<table>
<tr>
<th>Property</th>
<th>Meaning</th>
<th>Example</th>
</tr>
<tr>
<td><strong>Basically Available</strong></td>
<td>System responds even during failures</td>
<td>Social media post succeeds even if replica lags</td>
</tr>
<tr>
<td><strong>Soft State</strong></td>
<td>State changes without input (eventual consistency)</td>
<td>Like and count doesn't match immediately</td>
</tr>
<tr>
<td><strong>Eventually Consistent</strong></td>
<td>Will reach consistency over time</td>
<td>All replicas eventually have same data</td>
</tr>
</table>

<h3>When to Use</h3>

<ul>
<li><strong>ACID:</strong> Financial systems, inventory (strong guarantees critical)</li>
<li><strong>BASE:</strong> Social media, analytics, real-time feeds (eventual consistency OK)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/fundamentals/">Learn more: System Design</a></p>

</details>

---

## 6. What are idempotency and statelessness?

<details>
<summary>View Answer</summary>

<h3>Idempotency</h3>

<p><strong>Definition:</strong> Calling an operation multiple times = calling it once. Safe to retry.</p>

<h3>Examples</h3>

<table>
<tr>
<th>Operation</th>
<th>Idempotent?</th>
<th>Why?</th>
</tr>
<tr>
<td><code>PUT /users/123 {"name": "John"}</code></td>
<td>✅ Yes</td>
<td>Called 3x = same result</td>
</tr>
<tr>
<td><code>GET /users/123</code></td>
<td>✅ Yes</td>
<td>Always returns same data</td>
</tr>
<tr>
<td><code>POST /transfers {"from": "A", "to": "B", "amount": 100}</code></td>
<td>❌ No</td>
<td>Called 3x = 3 transfers</td>
</tr>
<tr>
<td><code>DELETE /users/123</code></td>
<td>✅ Yes</td>
<td>Deleting twice = same result</td>
</tr>
</table>

<h3>Ensuring Idempotency for POST</h3>

<p>Use idempotent keys (client-generated unique ID):</p>

```
POST /transfers
Headers: 
  Idempotent-Key: abc-123-def

If same key sent again, server skips if already processed
```

<h3>Statelessness</h3>

<p><strong>Definition:</strong> Server stores no session data. Each request contains all info needed.</p>

<h3>Benefits</h3>

<ul>
<li><strong>Horizontal scaling:</strong> Any server can handle any request</li>
<li><strong>No sticky sessions:</strong> Load balancer can route anywhere</li>
<li><strong>Fault tolerance:</strong> Server crash doesn't lose session data</li>
</ul>

<h3>Implementation</h3>

<p>Use tokens (JWT) instead of session storage:</p>

```
Client sends: Authorization: Bearer jwt-token
Server validates token (no session needed)
```

<p>📚 <a href="/anandu-interview-playbook/spring-web/security/">Learn more: Security & Authentication</a></p>

</details>

---

## 7. How do indexes improve query performance?

<details>
<summary>View Answer</summary>

<h3>What Indexes Do</h3>

<p>Indexes create a data structure (usually B-tree) that allows faster lookup than scanning every row.</p>

<h3>Without Index (Table Scan)</h3>

```
Query: SELECT * FROM users WHERE id = 5
Result: Scan ALL rows → O(n) time
```

<h3>With Index</h3>

```
Query: SELECT * FROM users WHERE id = 5
Result: Look up in B-tree index → O(log n) time
```

<h3>Index Types</h3>

<table>
<tr>
<th>Type</th>
<th>Use Case</th>
<th>Trade-off</th>
</tr>
<tr>
<td><strong>Primary Key Index</strong></td>
<td>Unique lookups (id=5)</td>
<td>Unique, fast</td>
</tr>
<tr>
<td><strong>Composite Index</strong></td>
<td>Multi-column queries (name, age)</td>
<td>Must match column order</td>
</tr>
<tr>
<td><strong>Full-Text Index</strong></td>
<td>Text search queries</td>
<td>Slower writes, fast searches</td>
</tr>
</table>

<h3>Index Trade-offs</h3>

<ul>
<li><strong>Pros:</strong> Faster reads, faster WHERE/JOIN/ORDER BY</li>
<li><strong>Cons:</strong> Slower writes (must update index), extra disk space, must maintain</li>
</ul>

<h3>When to Index</h3>

<ul>
<li><strong>DO index:</strong> Columns in WHERE, JOIN, ORDER BY, with high selectivity (many unique values)</li>
<li><strong>DON'T index:</strong> Columns with low selectivity (few unique values), columns rarely queried</li>
</ul>

<h3>Example: Composite Index</h3>

```sql
-- Query: Find users by last_name, then first_name
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

-- Create composite index (order matters!)
CREATE INDEX idx_name ON users(last_name, first_name);

-- This index helps the above query
-- But NOT: SELECT * FROM users WHERE first_name = 'John'
```

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Databases</a></p>

</details>

---

## 8. When would you choose NoSQL over a relational database?

<details>
<summary>View Answer</summary>

<h3>Choose NoSQL When</h3>

<table>
<tr>
<th>Scenario</th>
<th>NoSQL Advantage</th>
<th>Example</th>
</tr>
<tr>
<td><strong>Massive scale needed</strong></td>
<td>Designed for horizontal scaling</td>
<td>Billions of events/documents</td>
</tr>
<tr>
<td><strong>Flexible schema</strong></td>
<td>Schema-less; evolve without migrations</td>
<td>User profiles with varying fields</td>
</tr>
<tr>
<td><strong>High write throughput</strong></td>
<td>Optimized for fast writes</td>
<td>Real-time logs, metrics</td>
</tr>
<tr>
<td><strong>Non-relational data</strong></td>
<td>Stores documents, graphs, key-values naturally</td>
<td>JSON documents, social graphs</td>
</tr>
<tr>
<td><strong>Eventual consistency OK</strong></td>
<td>Prioritizes availability over consistency</td>
<td>Social media, caches</td>
</tr>
</table>

<h3>Choose Relational When</h3>

<ul>
<li><strong>Complex relationships:</strong> JOINs across many tables</li>
<li><strong>ACID guarantees needed:</strong> Financial, inventory</li>
<li><strong>Schema stability:</strong> Fixed structure, validated constraints</li>
<li><strong>Complex queries:</strong> Aggregations, reports</li>
</ul>

<h3>NoSQL Types & Use Cases</h3>

<table>
<tr>
<th>Type</th>
<th>Example</th>
<th>Best For</th>
</tr>
<tr>
<td><strong>Document</strong></td>
<td>MongoDB, Firebase</td>
<td>JSON-like data, flexible schema</td>
</tr>
<tr>
<td><strong>Key-Value</strong></td>
<td>Redis, DynamoDB</td>
<td>Caching, sessions, fast lookups</td>
</tr>
<tr>
<td><strong>Graph</strong></td>
<td>Neo4j, ArangoDB</td>
<td>Social networks, recommendations</td>
</tr>
<tr>
<td><strong>Time-Series</strong></td>
<td>InfluxDB, TimescaleDB</td>
<td>Metrics, logs, sensor data</td>
</tr>
</table>

<h3>Real-World Example</h3>

<p><strong>Social Media Platform:</strong></p>

<ul>
<li><strong>User profiles:</strong> MongoDB (flexible schema)</li>
<li><strong>Real-time feed:</strong> Cassandra (high write throughput, AP)</li>
<li><strong>Cache layer:</strong> Redis (key-value)</li>
<li><strong>Relationships:</strong> Neo4j (graph for recommendations)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/redis/">Learn more: NoSQL & Distributed Databases</a></p>

</details>

---

## Additional Topics

- Explain normalization and why it matters
- When would you denormalize a database?
- What's the difference between clustering and non-clustering indexes?
- How do you handle concurrent writes to the same record?
- How do you optimize a slow query?
- What's the difference between replication and sharding?

---

**See also:** [SQL & Relational Databases](../../databases/sql.md), [Interview Preparation](../index.md)
