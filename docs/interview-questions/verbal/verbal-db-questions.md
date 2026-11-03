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

## 9. What is a primary key and what is its significance?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A primary key is a column (or set of columns) that uniquely identifies each row in a table. It must be unique and <strong>NOT NULL</strong>, and a table can have only one.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Uniqueness:</strong> the database rejects duplicate key values.</li>
<li><strong>Indexing:</strong> a unique index is created automatically; in InnoDB (MySQL) and SQL Server it is usually the clustered index, so rows are stored in key order.</li>
<li><strong>Referential integrity:</strong> foreign keys in other tables reference it.</li>
<li><strong>Natural vs surrogate:</strong> natural keys (email, ISBN) can change; surrogate keys (auto-increment, UUID, sequence) are stable and meaningless.</li>
</ul>

<h3>Example</h3>

```sql
CREATE TABLE orders (
    id          BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    created_at  TIMESTAMP NOT NULL
);
```

<h3>Gotchas</h3>
<ul>
<li>Primary key vs unique key: a table may have many unique keys (NULL handling is vendor-dependent) but only one primary key.</li>
<li>Random UUID keys cause page splits on clustered indexes; consider sequential or time-ordered IDs.</li>
<li>Composite primary keys: column order matters for index usage.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 10. What are SQL DML and DDL commands? What is the difference?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>DDL</strong> (Data Definition Language) defines or changes the structure of the database. <strong>DML</strong> (Data Manipulation Language) reads and changes the data inside that structure.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>DDL</th><th>DML</th></tr>
<tr><td>Commands</td><td>CREATE, ALTER, DROP, TRUNCATE, RENAME</td><td>INSERT, UPDATE, DELETE, MERGE (SELECT is often grouped here, sometimes called DQL)</td></tr>
<tr><td>Acts on</td><td>Schema objects (tables, indexes, views)</td><td>Rows</td></tr>
<tr><td>Transaction</td><td>Often auto-commits and cannot be rolled back (Oracle, MySQL); PostgreSQL has transactional DDL</td><td>Can be rolled back until COMMIT</td></tr>
<tr><td>WHERE clause</td><td>No</td><td>Yes</td></tr>
</table>

<h3>Related Groups</h3>
<ul>
<li><strong>DCL:</strong> GRANT, REVOKE (permissions).</li>
<li><strong>TCL:</strong> COMMIT, ROLLBACK, SAVEPOINT.</li>
</ul>

<h3>Gotcha</h3>
<ul>
<li><strong>DELETE vs TRUNCATE vs DROP:</strong> DELETE removes rows one by one (filterable, logged, rollback-able); TRUNCATE removes all rows quickly (DDL, usually no rollback in Oracle/MySQL); DROP removes the table itself.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 11. What are the distinctions between stored procedures and functions?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A function computes and <strong>returns a value</strong> and can be used inside SQL expressions. A stored procedure performs an <strong>action</strong> (possibly many statements) and is invoked with CALL/EXEC. Exact rules vary by database.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Stored Procedure</th><th>Function</th></tr>
<tr><td>Return</td><td>Zero or more OUT params / result sets; return optional</td><td>Must return a value (scalar or table)</td></tr>
<tr><td>Usage in SQL</td><td>Called standalone (CALL / EXEC)</td><td>Callable in SELECT, WHERE, etc.</td></tr>
<tr><td>Transactions</td><td>Can usually COMMIT/ROLLBACK</td><td>Typically restricted (no transaction control, limited DML)</td></tr>
<tr><td>Side effects</td><td>Expected (INSERT/UPDATE/DDL)</td><td>Should be side-effect free (enforced in SQL Server)</td></tr>
</table>

<h3>Pros and Cons of Procedures</h3>
<ul>
<li>Pros: fewer network round trips, precompiled plans, centralized logic, permission control.</li>
<li>Cons: harder to version, test and debug; logic split between app and DB; vendor lock-in.</li>
</ul>

<h3>Example (Java call)</h3>

```java
try (CallableStatement cs = conn.prepareCall("{call transfer_funds(?, ?, ?)}")) {
    cs.setLong(1, fromId);
    cs.setLong(2, toId);
    cs.setBigDecimal(3, amount);
    cs.execute();
}
```

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 12. Explain normalization and why it matters

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Normalization organizes tables to remove redundancy and prevent update, insert and delete anomalies. Each fact is stored once, and tables are linked by keys.</p>

<h3>Normal Forms</h3>
<table>
<tr><th>Form</th><th>Rule</th></tr>
<tr><td>1NF</td><td>Atomic values, no repeating groups or comma-separated lists</td></tr>
<tr><td>2NF</td><td>1NF + no partial dependency on part of a composite key</td></tr>
<tr><td>3NF</td><td>2NF + no transitive dependency (non-key columns depend only on the key)</td></tr>
<tr><td>BCNF</td><td>Every determinant is a candidate key (stricter 3NF)</td></tr>
</table>

<h3>Why It Matters</h3>
<ul>
<li><strong>Consistency:</strong> change a customer's address in one place.</li>
<li><strong>Less storage</strong> and smaller rows.</li>
<li><strong>Cleaner constraints</strong> and integrity.</li>
<li><strong>Cost:</strong> more joins, so reads can be slower.</li>
</ul>

<h3>Example</h3>
<p>Instead of storing <code>customer_name</code> and <code>customer_city</code> on every order row, keep a <code>customers</code> table and store only <code>customer_id</code> in <code>orders</code>.</p>

<h3>When to Use</h3>
<p>Aim for 3NF in transactional (OLTP) systems, then denormalize deliberately where measured read performance requires it.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 13. When would you denormalize a database?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Denormalize when joins have become a measured bottleneck on read-heavy paths and the extra redundancy is acceptable. It trades write complexity and consistency risk for read speed.</p>

<h3>Common Techniques</h3>
<ul>
<li>Copy a frequently read column into another table (e.g., <code>customer_name</code> on orders).</li>
<li>Store precomputed aggregates (order totals, counters).</li>
<li>Materialized views or summary tables for reporting.</li>
<li>Embed related data in documents (NoSQL).</li>
</ul>

<h3>Costs</h3>
<ul>
<li>Updates must change several places (use transactions, triggers, or async jobs).</li>
<li>Risk of inconsistent data; more storage.</li>
</ul>

<h3>Good Candidates</h3>
<ul>
<li>Reporting / analytics and data warehouses (star schema).</li>
<li>Read models in CQRS.</li>
<li>Historical snapshots, e.g. keep the price and address at the time of an order, which is correct as well as fast.</li>
</ul>

<h3>Gotcha</h3>
<p>Profile first: often a missing index or a bad query is the real problem, and caching may be a cheaper fix.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 14. What's the difference between clustering and non-clustering indexes?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A <strong>clustered</strong> index defines the physical order of the table rows (the table is the index leaf level). A <strong>non-clustered</strong> (secondary) index is a separate structure whose leaves hold the key plus a pointer to the row.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Clustered</th><th>Non-clustered</th></tr>
<tr><td>Per table</td><td>Only one</td><td>Many</td></tr>
<tr><td>Leaf contents</td><td>The actual row data</td><td>Key + row locator (clustered key or row ID)</td></tr>
<tr><td>Range scans on key</td><td>Very fast (rows adjacent)</td><td>Needs lookups unless covering</td></tr>
<tr><td>Write cost</td><td>Inserts in the middle can split pages</td><td>Every index adds write overhead</td></tr>
</table>

<h3>Notes</h3>
<ul>
<li>InnoDB: the primary key is the clustered index; secondary indexes store the primary key value, so a wide PK bloats all indexes.</li>
<li>SQL Server: you choose the clustered index (default is the PK). PostgreSQL tables are heaps; <code>CLUSTER</code> reorders once and is not maintained.</li>
<li>A <strong>covering index</strong> (includes all queried columns) avoids the extra lookup.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 15. Would you choose SQL or NoSQL for a payments (financial transaction) system, and why?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Default to a relational (SQL) database for the core ledger and payments: they need ACID transactions, strong consistency, constraints and auditability.</p>

<h3>Why SQL</h3>
<ul>
<li>Multi-row atomic transfers (debit and credit succeed together).</li>
<li>Strong isolation and constraints (unique idempotency keys, foreign keys, CHECK rules).</li>
<li>Mature tooling for reporting, reconciliation, backups and audit.</li>
</ul>

<h3>Where NoSQL Fits</h3>
<ul>
<li>Caches, sessions, rate counters (Redis).</li>
<li>Event/audit logs and high-volume read models (document or wide-column stores).</li>
<li>Search and analytics projections.</li>
</ul>

<h3>Nuance</h3>
<ul>
<li>Some NoSQL stores now offer multi-document transactions, but you give up joins and ad-hoc querying; weigh operational maturity.</li>
<li>Scale a SQL store first with indexes, read replicas and partitioning before changing the data model.</li>
<li>Typical answer: SQL as the system of record, NoSQL as a supporting store (polyglot persistence).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/scaling/">Learn more: Scaling & NoSQL</a></p>

</details>

---

## 16. Which data type would you use in Oracle to store a JSON document?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Oracle 21c and later has a native <strong>JSON</strong> data type (binary storage). On earlier releases (12c+), store JSON in <strong>VARCHAR2</strong>, <strong>CLOB</strong> or <strong>BLOB</strong> with an <code>IS JSON</code> check constraint.</p>

<h3>Choosing the Column Type (pre-21c)</h3>
<ul>
<li><strong>VARCHAR2:</strong> small documents (4000 bytes, or 32767 with extended data types).</li>
<li><strong>CLOB:</strong> large text documents.</li>
<li><strong>BLOB:</strong> binary storage, generally recommended by Oracle for large JSON because it avoids character-set conversion.</li>
</ul>

<h3>Example</h3>

```sql
CREATE TABLE events (
    id      NUMBER PRIMARY KEY,
    payload CLOB CHECK (payload IS JSON)
);

SELECT JSON_VALUE(payload, '$.customer.id') FROM events;
-- Index a field:
CREATE INDEX ev_cust_ix ON events (JSON_VALUE(payload, '$.customer.id'));
```

<h3>Gotchas</h3>
<ul>
<li>Always add the <code>IS JSON</code> constraint to validate on write.</li>
<li>Query with <code>JSON_VALUE</code>, <code>JSON_QUERY</code>, <code>JSON_TABLE</code>; index frequently filtered paths.</li>
<li>Do not store everything as JSON: keep relational columns for fields you join, filter and constrain on. (PostgreSQL equivalent: <code>JSONB</code>.)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

**See also:** [SQL & Relational Databases](../../databases/sql.md), [Interview Preparation](../index.md)
