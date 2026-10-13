# Databases & SQL Interview Questions

Conceptual questions on database design, SQL queries, transactions, optimization, and best practices.

---

## Question Categories

### Database Design & Normalization
- Normalization (1NF, 2NF, 3NF, BCNF)
- Foreign keys, relationships (1-to-1, 1-to-many, many-to-many)
- Denormalization trade-offs

### SQL Queries
- Complex JOINs, subqueries, aggregations
- Query optimization, EXPLAIN plans
- Window functions

### Transactions & ACID
- ACID properties
- Isolation levels, concurrency control
- Locking, deadlock prevention

### Indexing
- Index types, composite indexes
- When to index, trade-offs
- Query optimization using indexes

### Database Performance
- Slow query identification and optimization
- Caching strategies
- Scaling: replication, sharding, partitioning

### NoSQL Databases
- When to use NoSQL vs relational
- Document, key-value, graph databases
- CAP theorem, eventual consistency

---

## Common Questions & Answers

### SQL Joins Deep-Dive
**Q: What's the difference between INNER JOIN and LEFT JOIN? What about UNION?**

**INNER JOIN**: Returns rows present in BOTH tables
```sql
SELECT o.id, c.name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;
-- Only shows customers with orders
```

**LEFT JOIN**: Returns ALL rows from left table + matching rows from right
```sql
SELECT o.id, c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
-- Shows all customers, even without orders (NULL in order fields)
```

**UNION**: Combines result sets from multiple queries (removes duplicates)
```sql
SELECT name FROM customers
UNION
SELECT name FROM vendors;
-- Result: All unique names from both tables
```

**UNION ALL**: Like UNION but keeps duplicates

---

### HAVING vs WHERE
**Q: What's the difference between WHERE and HAVING?**

- **WHERE**: Filters **rows BEFORE aggregation**
- **HAVING**: Filters **groups AFTER aggregation**

```sql
-- WHERE: Filter customers BEFORE grouping
SELECT customer_id, COUNT(*) as order_count
FROM orders
WHERE order_date > '2024-01-01'  -- Filter before grouping
GROUP BY customer_id
HAVING COUNT(*) > 5;  -- Filter groups (aggregated results)
-- Result: Customers with >5 orders since 2024-01-01
```

**Cannot use aggregate functions in WHERE** (e.g., `WHERE COUNT(*) > 5` is invalid)

---

### Caching Patterns
**Q: What are common database caching patterns?**

1. **Cache-Aside (Lazy Loading)**
   - App queries cache first; if miss, fetch DB and populate cache
   - Most common for read-heavy workloads
2. **Read-Through**
   - Cache library handles fetching from DB
   - More transparent
3. **Write-Through**
   - Write to cache AND DB simultaneously
   - Ensures consistency but slower
4. **Write-Behind (Write-Back)**
   - Write to cache immediately; write to DB asynchronously
   - Fast writes, risk of data loss if cache crashes

**Best for interviews:** Cache-Aside with Redis is standard pattern.

---

### CAP Theorem
**Q: Explain the CAP theorem and its implications.**

In a distributed system, you can guarantee only **2 of 3**:
- **Consistency**: All nodes see same data
- **Availability**: System always responds
- **Partition tolerance**: System works despite network partitions

**Real-world implications:**
- **CA**: Relational DBs (no partition tolerance) — useful when network is reliable
- **AP**: NoSQL (e.g., Cassandra, DynamoDB) — eventual consistency
- **CP**: HBase, Zookeeper — sacrifice availability for consistency

**Network partitions always happen** → Most systems are either AP or CP, rarely CA.

---

### ACID vs BASE
**Q: What's the difference between ACID and BASE consistency models?**

**ACID** (Traditional DBs):
- **Atomicity**: Transaction all-or-nothing
- **Consistency**: DB always valid
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data persists

**BASE** (NoSQL, Distributed):
- **Basically Available**: System responds even during failures
- **Soft state**: State changes without input (eventual consistency)
- **Eventually Consistent**: Will reach consistency over time

**When to use:**
- ACID: Financial, inventory systems (need strong guarantees)
- BASE: Social media, analytics (eventual consistency acceptable)

---

### Idempotency & Statelessness
**Q: What are idempotency and statelessness in database systems?**

**Idempotency**: Calling operation multiple times = calling once
```java
// Idempotent: Safe to retry
PUT /api/customers/123 { "name": "John" }
// Called 3x → same result

// NOT idempotent:
POST /api/transfers { "from": "A", "to": "B", "amount": 100 }
// Called 3x → 3 transfers!
```

**Solution:** Idempotent keys (client-generated unique ID)
```
POST /api/transfers
Headers: Idempotent-Key: abc-123-def
// Same key = skip if already processed
```

**Statelessness**: No session data stored on server
- Each request contains all info needed
- Enables horizontal scaling
- Every server can handle any request

---

## Sample Topics (To expand)

- Explain normalization and why it matters
- When would you denormalize a database?
- How do indexes improve query performance?
- What's the difference between clustering and non-clustering indexes?
- How do you handle concurrent writes to the same record?
- When would you choose NoSQL over a relational database?
- How do you optimize a slow query?

---

[More content to be added progressively]

**See also:** [SQL & Relational Databases](../../databases/sql.md), [Interview Questions Overview](../index.md)
