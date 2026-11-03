# Database Scaling and Architecture Questions

Verbal interview questions on schema migrations, multi-tenancy, partitioning, replication and sharding.

---

## 1. What are database schema evolution patterns?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Schema evolution means changing the database structure safely over time using versioned migrations, applied by tools such as Flyway or Liquibase, and designed so old and new application versions can both run during a rollout.</p>

<h3>Tools</h3>
<ul>
<li><strong>Flyway:</strong> ordered SQL (or Java) migrations like <code>V12__add_status.sql</code>; tracks applied versions in a history table.</li>
<li><strong>Liquibase:</strong> changelogs in XML/YAML/SQL with changesets and rollback support.</li>
</ul>

<h3>Expand / Contract (Backward Compatible) Pattern</h3>
<ol>
<li><strong>Expand:</strong> add the new nullable column or table; the old code still works.</li>
<li><strong>Deploy</strong> code that writes both and reads the new one; backfill existing rows in batches.</li>
<li><strong>Contract:</strong> in a later release, drop the old column once nothing uses it.</li>
</ol>

<h3>Rules of Thumb</h3>
<ul>
<li>Never edit an applied migration (checksum failure); add a new one.</li>
<li>Avoid renames and type changes in one step; use add/copy/switch/drop.</li>
<li>Beware long locks: adding a column with a default or an index on a huge table can block writes (use online or concurrent options where available).</li>
<li>Run migrations separately from app startup in multi-instance deployments, or rely on the tool's locking.</li>
<li>Test migrations against a production-like copy; plan roll-forward over rollback.</li>
</ul>

<h3>Example</h3>

```sql
-- V12__add_order_status.sql  (expand step)
ALTER TABLE orders ADD COLUMN status VARCHAR(20);
```

<p>📚 <a href="/anandu-interview-playbook/databases/sql/">Learn more: SQL & Relational Databases</a></p>

</details>

---

## 2. How to implement multi-tenant database design?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Multi-tenancy serves many customers (tenants) from one application. The data isolation model ranges from fully separate databases to shared tables with a tenant column.</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Model</th><th>Isolation</th><th>Cost / Ops</th><th>Notes</th></tr>
<tr><td>Database per tenant</td><td>Strongest</td><td>Highest; many databases to manage</td><td>Per-tenant backup, restore, residency</td></tr>
<tr><td>Schema per tenant</td><td>Strong</td><td>Medium; migrations run per schema</td><td>Limits at thousands of tenants</td></tr>
<tr><td>Shared tables + <code>tenant_id</code> (row-level)</td><td>Logical only</td><td>Lowest; scales to many tenants</td><td>Every query must filter by tenant</td></tr>
</table>

<h3>Implementing Row-Level Isolation</h3>
<ul>
<li>Resolve the tenant from the request (JWT claim, subdomain) into a context.</li>
<li>Enforce filtering centrally: Hibernate <code>@Filter</code>/<code>@TenantId</code> or a multi-tenancy connection provider, never ad hoc in each query.</li>
<li>Defense in depth: PostgreSQL Row-Level Security policies, composite indexes starting with <code>tenant_id</code>, and tests that try cross-tenant access.</li>
<li>For DB/schema per tenant, route the <code>DataSource</code> or set the schema per request.</li>
</ul>

<h3>Trade-offs</h3>
<ul>
<li>Noisy neighbor: one big tenant can slow others in shared tables.</li>
<li>Hybrid is common: shared model for small tenants, dedicated database for large or regulated ones.</li>
</ul>

<h3>Gotcha</h3>
<p>A missing tenant filter is a data leak; make isolation automatic and test for it.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/scaling/">Learn more: Scaling & NoSQL</a></p>

</details>

---

## 3. What are database partitioning strategies?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Partitioning splits one large table into smaller pieces (partitions) inside a database so queries and maintenance touch less data. The common strategies are range, list and hash.</p>

<h3>Strategies</h3>
<table>
<tr><th>Strategy</th><th>Rule</th><th>Good for</th><th>Watch out</th></tr>
<tr><td>Range</td><td>By value range (e.g., month of <code>created_at</code>)</td><td>Time-series, archiving old data by dropping a partition</td><td>Hot latest partition</td></tr>
<tr><td>List</td><td>By explicit values (region, status)</td><td>Natural categories</td><td>Uneven sizes</td></tr>
<tr><td>Hash</td><td>Hash of key modulo N</td><td>Even distribution, avoid hot spots</td><td>No range pruning; resizing is costly</td></tr>
</table>

<h3>Benefits</h3>
<ul>
<li><strong>Partition pruning:</strong> the optimizer scans only relevant partitions.</li>
<li>Faster maintenance (drop or detach an old partition instead of a huge DELETE).</li>
<li>Smaller indexes per partition.</li>
</ul>

<h3>Example (PostgreSQL)</h3>

```sql
CREATE TABLE events (id BIGINT, created_at DATE NOT NULL, data TEXT)
PARTITION BY RANGE (created_at);
CREATE TABLE events_2025_01 PARTITION OF events
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

<h3>Gotchas</h3>
<ul>
<li>Queries must include the partition key to benefit.</li>
<li>Unique constraints generally must include the partition key.</li>
<li>Partitioning is within one database; spreading across servers is <em>sharding</em>.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/scaling/">Learn more: Scaling & NoSQL</a></p>

</details>

---

## 4. How to implement replication and high availability in databases?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Replication keeps copies of data on multiple nodes. High availability adds automatic failure detection and failover so the service survives a node or zone loss.</p>

<h3>Topologies</h3>
<table>
<tr><th>Topology</th><th>How it works</th><th>Trade-off</th></tr>
<tr><td>Primary-replica (master-slave)</td><td>One writer; replicas apply its log and serve reads</td><td>Simple; replica lag; single write point</td></tr>
<tr><td>Multi-primary (master-master)</td><td>Several nodes accept writes</td><td>Write availability; conflict resolution is hard</td></tr>
<tr><td>Read replicas</td><td>Replicas used to offload reads</td><td>Reads may be stale</td></tr>
</table>

<h3>Synchronous vs Asynchronous</h3>
<ul>
<li><strong>Async:</strong> fast commits, but a failover can lose the most recent transactions.</li>
<li><strong>Sync (or semi-sync):</strong> no data loss on failover, higher write latency.</li>
</ul>

<h3>High Availability Pieces</h3>
<ul>
<li>Health checks plus automated failover and promotion (managed services such as RDS Multi-AZ, or Patroni, orchestrators).</li>
<li>Stable endpoint (virtual IP, DNS, proxy) so clients reconnect to the new primary.</li>
<li>Spread nodes across availability zones; take backups and test restores; avoid split brain with quorum or fencing.</li>
<li>Define RPO/RTO targets.</li>
</ul>

<h3>Application Impact</h3>
<ul>
<li>Route writes to the primary, reads to replicas; handle read-your-own-writes (read from primary right after a write).</li>
<li>Use connection retry logic for failover.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/databases/scaling/">Learn more: Scaling & NoSQL</a></p>

</details>

---

## 5. What's the difference between replication and sharding?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><strong>Replication</strong> copies the same data to multiple nodes (availability and read scale). <strong>Sharding</strong> splits the data across nodes so each holds a subset (write and storage scale).</p>

<h3>Comparison Table</h3>
<table>
<tr><th>Aspect</th><th>Replication</th><th>Sharding</th></tr>
<tr><td>Data per node</td><td>Full copy</td><td>Subset</td></tr>
<tr><td>Main goal</td><td>HA, read scaling, backups</td><td>Scale writes and storage beyond one node</td></tr>
<tr><td>Writes</td><td>Still go to one primary (single-leader)</td><td>Spread across shards</td></tr>
<tr><td>Complexity</td><td>Moderate (lag, failover)</td><td>High (shard key, cross-shard joins/transactions, rebalancing)</td></tr>
<tr><td>Failure</td><td>Replica takes over</td><td>Lose a shard's data unless that shard is also replicated</td></tr>
</table>

<h3>Key Points</h3>
<ul>
<li>They are complementary: production systems shard for scale and replicate each shard for availability.</li>
<li><strong>Shard key choice</strong> is critical: high cardinality, even distribution, matches query patterns. Hash sharding spreads load; range sharding supports range queries but risks hot shards. Consistent hashing eases rebalancing.</li>
<li>Cross-shard joins and transactions are hard; avoid them via design, or use sagas.</li>
<li>Partitioning is the single-database cousin of sharding.</li>
</ul>

<h3>When to Use</h3>
<p>Exhaust vertical scaling, indexing, caching and read replicas first; shard only when write volume or data size demands it.</p>

<p>📚 <a href="/anandu-interview-playbook/databases/scaling/">Learn more: Scaling & NoSQL</a></p>

</details>

---
