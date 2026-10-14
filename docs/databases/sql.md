# SQL & Relational Databases

Comprehensive guide to SQL: DDL, DML, query patterns, joins, aggregations, transactions, and optimization.

---

## SQL Basics

**SQL Statement Types:**
- **DDL** (Data Definition): CREATE, ALTER, DROP, TRUNCATE
- **DML** (Data Manipulation): SELECT, INSERT, UPDATE, DELETE
- **DCL** (Data Control): GRANT, REVOKE
- **TCL** (Transaction Control): COMMIT, ROLLBACK, SAVEPOINT

**Query Clause Order:**
```sql
SELECT ... 
FROM ... 
WHERE ... 
GROUP BY ... 
HAVING ... 
ORDER BY ... 
LIMIT;
```

---

## CREATE TABLE (DDL)

Define table structure.

```sql
-- Basic table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table with foreign key
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Table with composite primary key
CREATE TABLE order_items (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id)
);

-- Create index for faster queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

## INSERT (DML)

Add rows to table.

```sql
-- Single row
INSERT INTO users (name, email, age) 
VALUES ('Alice', 'alice@example.com', 30);

-- Multiple rows
INSERT INTO users (name, email, age) 
VALUES 
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35),
    ('Diana', 'diana@example.com', 28);

-- Insert from another table
INSERT INTO users_backup (name, email, age)
SELECT name, email, age FROM users WHERE age > 30;

-- Insert with auto-generated ID (can retrieve with LAST_INSERT_ID())
INSERT INTO orders (user_id, amount) VALUES (1, 99.99);
SELECT LAST_INSERT_ID();  -- Returns: 1
```

---

## UPSERT (INSERT or UPDATE)

**UPSERT** — Insert if not exists, update if already exists. Use case: sync data from external sources.

### MySQL: INSERT ... ON DUPLICATE KEY UPDATE

```sql
-- If unique key/primary key exists, update; otherwise insert
INSERT INTO users (id, name, email, age)
VALUES (1, 'John', 'john@example.com', 30)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    email = VALUES(email),
    age = VALUES(age);

-- Update only specific columns
INSERT INTO users (email, name)
VALUES ('alice@example.com', 'Alice')
ON DUPLICATE KEY UPDATE
    name = 'Alice',
    updated_at = NOW();

-- MySQL 8.0.20+: Use NEW keyword
INSERT INTO users (id, name, email)
VALUES (1, 'John', 'john@example.com')
ON DUPLICATE KEY UPDATE
    name = NEW.name,
    email = NEW.email;

-- Bulk upsert multiple rows
INSERT INTO users (id, name, email, age)
VALUES 
    (1, 'John', 'john@example.com', 30),
    (2, 'Jane', 'jane@example.com', 25),
    (3, 'Bob', 'bob@example.com', 35)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    email = VALUES(email),
    age = VALUES(age);
```

### PostgreSQL: INSERT ... ON CONFLICT DO UPDATE

```sql
-- Insert, or update on conflict with unique/primary key
INSERT INTO users (id, name, email, age)
VALUES (1, 'John', 'john@example.com', 30)
ON CONFLICT (id) DO UPDATE SET
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    age = EXCLUDED.age;

-- Conflict on composite unique key
INSERT INTO user_settings (user_id, setting_key, setting_value)
VALUES (1, 'theme', 'dark')
ON CONFLICT (user_id, setting_key) DO UPDATE SET
    setting_value = EXCLUDED.setting_value,
    updated_at = NOW();

-- Don't update, just ignore conflict
INSERT INTO users (id, name, email)
VALUES (1, 'John', 'john@example.com')
ON CONFLICT (id) DO NOTHING;
```

### SQL Server: MERGE Statement

```sql
-- Most flexible: can INSERT, UPDATE, DELETE in one statement
MERGE INTO users AS target
USING (SELECT 1 AS id, 'John' AS name, 'john@example.com' AS email, 30 AS age) AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET 
        target.name = source.name,
        target.email = source.email,
        target.age = source.age
WHEN NOT MATCHED THEN
    INSERT (id, name, email, age)
    VALUES (source.id, source.name, source.email, source.age);

-- Bulk merge
MERGE INTO users AS target
USING (VALUES 
    (1, 'John', 'john@example.com', 30),
    (2, 'Jane', 'jane@example.com', 25)
) AS source(id, name, email, age)
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET name = source.name, email = source.email, age = source.age
WHEN NOT MATCHED THEN
    INSERT (id, name, email, age) VALUES (source.id, source.name, source.email, source.age);

-- Also allow DELETE
MERGE INTO users AS target
USING source_data AS source
ON target.id = source.id
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...
WHEN NOT MATCHED BY SOURCE AND created_before = '2020-01-01' THEN DELETE;
```

### Alternative: Check & Do-If Pattern

For databases without native UPSERT:

```sql
-- Check if exists, then decide
SET @id = 1;
SET @name = 'John';
SET @email = 'john@example.com';

IF EXISTS(SELECT 1 FROM users WHERE id = @id)
    UPDATE users SET name = @name, email = @email WHERE id = @id;
ELSE
    INSERT INTO users (id, name, email) VALUES (@id, @name, @email);
```

### Performance Considerations

- **Single UPSERT** — Atomic, no race conditions
- **Bulk UPSERT** — Much faster than row-by-row inserts/updates
- **Indexes** — Ensure unique key is indexed for UPSERT performance

```sql
-- Create unique index for UPSERT to work efficiently
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Now upsert by email
INSERT INTO users (email, name)
VALUES ('john@example.com', 'John')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

---

## SELECT (DML)

Query and retrieve data.

```sql
-- Basic SELECT
SELECT id, name, email FROM users;

-- SELECT all columns
SELECT * FROM users;

-- WITH aliases
SELECT 
    id AS user_id,
    name AS full_name,
    email AS user_email
FROM users;

-- DISTINCT values (remove duplicates)
SELECT DISTINCT city FROM users;

-- LIMIT and OFFSET (pagination)
SELECT * FROM users LIMIT 10 OFFSET 20;  -- Skip 20, take 10
SELECT * FROM users LIMIT 20, 10;  -- MySQL shorthand (offset, limit)
```

---

## WHERE (Filtering)

Filter rows based on conditions.

```sql
-- Equality
SELECT * FROM users WHERE age = 30;

-- Comparison operators
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE age >= 25;
SELECT * FROM users WHERE age < 30;
SELECT * FROM users WHERE age <= 30;
SELECT * FROM users WHERE age <> 30;  -- NOT equal

-- AND / OR
SELECT * FROM users WHERE age > 25 AND city = 'New York';
SELECT * FROM users WHERE age > 25 OR city = 'New York';
SELECT * FROM users WHERE (age > 25 OR age < 20) AND city = 'New York';

-- IN (multiple values)
SELECT * FROM users WHERE id IN (1, 3, 5);
SELECT * FROM orders WHERE status IN ('PENDING', 'PROCESSING', 'SHIPPED');

-- BETWEEN (inclusive)
SELECT * FROM users WHERE age BETWEEN 25 AND 35;

-- LIKE (pattern matching)
SELECT * FROM users WHERE name LIKE 'A%';        -- Starts with A
SELECT * FROM users WHERE name LIKE '%a';        -- Ends with a
SELECT * FROM users WHERE name LIKE '%li%';      -- Contains li
SELECT * FROM users WHERE email LIKE '%.com';    -- Email ends with .com

-- IS NULL / IS NOT NULL
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;
```

---

## JOIN (Combining Tables)

Combine rows from multiple tables.

```sql
-- INNER JOIN: Only matching rows
SELECT 
    users.id,
    users.name,
    orders.id AS order_id,
    orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id
WHERE orders.amount > 50;

-- LEFT JOIN: All left rows + matched right (NULL for non-matching)
SELECT 
    users.name,
    orders.amount,
    orders.created_at
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- Result: All users, even if they have no orders (NULL in order columns)

-- RIGHT JOIN: All right rows + matched left
SELECT 
    users.name,
    orders.amount
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
-- Result: All orders, with NULL if user deleted

-- FULL OUTER JOIN: All rows from both tables
SELECT 
    users.name,
    orders.amount
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;

-- Multiple JOINs
SELECT 
    u.name,
    o.amount,
    oi.quantity,
    p.name AS product_name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.created_at >= '2024-01-01';

-- SELF JOIN (table joined to itself)
SELECT 
    e1.name AS employee,
    e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

---

## GROUP BY & Aggregations

Group rows and calculate aggregate values.

```sql
-- COUNT: number of rows
SELECT COUNT(*) AS total_users FROM users;
SELECT COUNT(email) FROM users;  -- NULL emails excluded

-- SUM: total
SELECT SUM(amount) AS total_revenue FROM orders;

-- AVG: average
SELECT AVG(amount) AS avg_order_value FROM orders;

-- MIN / MAX
SELECT MIN(age) AS youngest, MAX(age) AS oldest FROM users;

-- GROUP BY: group rows by column(s)
SELECT 
    city,
    COUNT(*) AS user_count,
    AVG(age) AS avg_age
FROM users
GROUP BY city;

-- GROUP BY multiple columns
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    status,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount
FROM orders
GROUP BY DATE_TRUNC('month', created_at), status;

-- HAVING: filter AFTER grouping (use WHERE before grouping, HAVING after)
SELECT 
    city,
    COUNT(*) AS user_count
FROM users
WHERE age > 25  -- Filter before grouping
GROUP BY city
HAVING COUNT(*) > 5;  -- Filter after grouping (cities with >5 users)

-- Common aggregates with grouping
SELECT 
    user_id,
    COUNT(*) AS num_orders,
    SUM(amount) AS total_spent,
    AVG(amount) AS avg_order_value,
    MIN(amount) AS min_order,
    MAX(amount) AS max_order
FROM orders
GROUP BY user_id
HAVING SUM(amount) > 1000;  -- Only users who spent > 1000
```

---

## ORDER BY (Sorting)

Sort results.

```sql
-- Ascending (default)
SELECT * FROM users ORDER BY age;
SELECT * FROM users ORDER BY age ASC;

-- Descending
SELECT * FROM users ORDER BY age DESC;

-- Multiple columns
SELECT * FROM users ORDER BY age DESC, name ASC;
-- Sorts by age (descending), then by name (ascending)

-- Order by expression
SELECT 
    name,
    age,
    (2024 - birth_year) AS calculated_age
FROM users
ORDER BY (2024 - birth_year) DESC;

-- LIMIT with ORDER BY (top N)
SELECT * FROM orders ORDER BY amount DESC LIMIT 5;  -- Top 5 orders
```

---

## UPDATE (DML)

Modify existing rows.

```sql
-- Update single row
UPDATE users 
SET age = 31, email = 'newemail@example.com'
WHERE id = 1;

-- Update multiple rows
UPDATE users 
SET active = true
WHERE created_at >= '2024-01-01';

-- Update with calculation
UPDATE orders 
SET amount = amount * 1.1  -- 10% increase
WHERE created_at < '2024-01-01';

-- UPDATE with JOIN (update based on related table)
UPDATE users u
SET subscription_level = 'premium'
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.id 
    AND o.amount > 1000
);

-- Be careful! Missing WHERE updates all rows
UPDATE users SET age = 25;  -- ❌ DANGEROUS: updates ALL users!
```

---

## DELETE (DML)

Remove rows.

```sql
-- Delete single row
DELETE FROM users WHERE id = 5;

-- Delete multiple rows
DELETE FROM orders 
WHERE created_at < '2020-01-01';

-- Delete with JOIN
DELETE FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM users u WHERE u.id = o.user_id
);  -- Delete orphaned orders

-- Be careful! Missing WHERE deletes all rows
DELETE FROM users;  -- ❌ DANGEROUS: deletes ALL users!

-- Truncate: delete all rows (faster than DELETE, can't rollback)
TRUNCATE TABLE users;  -- ⚠️ Use with caution
```

---

## Advanced Patterns

### Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM users 
WHERE id IN (
    SELECT user_id FROM orders WHERE amount > 100
);

-- Subquery in FROM
SELECT user_summary.user_id, user_summary.total_orders
FROM (
    SELECT user_id, COUNT(*) AS total_orders
    FROM orders
    GROUP BY user_id
) AS user_summary
WHERE user_summary.total_orders > 5;

-- Correlated subquery (references outer query)
SELECT u.name, u.email
FROM users u
WHERE (
    SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id
) > 3;  -- Users with >3 orders
```

### CASE (Conditional Logic)

```sql
-- Simple CASE
SELECT 
    name,
    age,
    CASE age
        WHEN 18 THEN 'Young Adult'
        WHEN 25 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group
FROM users;

-- Searched CASE (more flexible)
SELECT 
    name,
    age,
    CASE 
        WHEN age < 18 THEN 'Minor'
        WHEN age BETWEEN 18 AND 25 THEN 'Young Adult'
        WHEN age BETWEEN 26 AND 65 THEN 'Adult'
        ELSE 'Senior'
    END AS age_category
FROM users;

-- CASE with aggregation
SELECT 
    CASE status
        WHEN 'SHIPPED' THEN 'Completed'
        WHEN 'PENDING' THEN 'Incomplete'
        ELSE 'Other'
    END AS order_status,
    COUNT(*) AS count
FROM orders
GROUP BY CASE status
    WHEN 'SHIPPED' THEN 'Completed'
    WHEN 'PENDING' THEN 'Incomplete'
    ELSE 'Other'
END;
```

### UNION (Combine Results)

```sql
-- UNION: combine with DISTINCT
SELECT name, email, 'user' AS source FROM users
UNION
SELECT company_name, contact_email, 'company' FROM companies;

-- UNION ALL: keep duplicates
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

---

## Normalization

Reduce redundancy and anomalies.

- **1NF**: Atomic values, no repeating groups
- **2NF**: 1NF + no partial dependency on composite key
- **3NF**: 2NF + no transitive dependency (non-key doesn't depend on non-key)
- **BCNF**: Stricter 3NF

**Example: Normalize to 3NF**
```sql
-- ❌ Non-normalized (redundant data)
CREATE TABLE orders_bad (
    order_id INT,
    user_id INT,
    user_name VARCHAR(100),  -- Redundant (repeats for each order)
    user_email VARCHAR(100),  -- Redundant
    product_name VARCHAR(100),  -- Redundant
    amount DECIMAL(10, 2)
);

-- ✅ Normalized (3NF)
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10, 2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id)
);
```

---

## Indexing Strategy

Speed up queries at cost of slower writes.

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id_created ON orders(user_id, created_at);

-- Query uses index if condition matches
SELECT * FROM users WHERE email = 'alice@example.com';  -- ✅ Uses idx_users_email

-- Composite index (leftmost-prefix rule)
SELECT * FROM orders 
WHERE user_id = 1 AND created_at > '2024-01-01';  -- ✅ Uses composite index

SELECT * FROM orders 
WHERE created_at > '2024-01-01';  -- ❌ Doesn't use composite index (created_at not first)

-- Check query plan (MySQL)
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
-- Look for: Using index, Index range, no Full table scan

-- Drop index if not needed
DROP INDEX idx_users_email ON users;
```

---

## WHERE vs HAVING

| Clause | When | Filters |
|--------|------|---------|
| **WHERE** | Before grouping | Individual rows |
| **HAVING** | After grouping | Grouped rows / aggregates |

```sql
SELECT 
    city,
    COUNT(*) AS user_count
FROM users
WHERE age > 25          -- ← Filters rows BEFORE grouping
GROUP BY city
HAVING COUNT(*) > 5;    -- ← Filters groups AFTER aggregation
```

---

## Transactions & Isolation

```sql
-- Transaction: all-or-nothing
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both succeed together
-- Or: ROLLBACK; (undo all changes)

-- Isolation levels (trade consistency vs performance)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- READ UNCOMMITTED: dirty reads (see uncommitted data)
-- READ COMMITTED: no dirty reads (see committed data only)
-- REPEATABLE READ: no non-repeatable reads (consistent within transaction)
-- SERIALIZABLE: full isolation (safest, slowest)
```

---

## Interview Talking Points

- **INNER vs LEFT JOIN?** INNER: only matching rows; LEFT: all left rows + matched right (NULLs otherwise)
- **WHERE vs HAVING?** WHERE filters before grouping; HAVING filters after aggregation
- **N+1 query problem?** Running 1 query + N more for each result (slow); use JOIN instead
- **Normalization trade-off?** Reduces redundancy but requires more JOINs; denormalize for read performance
- **Index pros/cons?** Pros: faster queries; Cons: slower writes, extra storage
- **Composite index leftmost-prefix?** Only first column used if WHERE doesn't match first column order
- **Transaction isolation?** Higher isolation = safer (no dirty reads) but slower; lower isolation = faster but riskier
- **UNION vs UNION ALL?** UNION removes duplicates; UNION ALL keeps them (UNION ALL faster)

---

**See also:** [Databases Overview](index.md), [Scaling & NoSQL](scaling.md)
