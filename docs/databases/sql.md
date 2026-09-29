# SQL & Relational Databases

## SQL basics
- **DDL** (CREATE/ALTER/DROP), **DML** (SELECT/INSERT/UPDATE/DELETE), **DCL** (GRANT/REVOKE), **TCL** (COMMIT/ROLLBACK).
- Clause order: `SELECT … FROM … WHERE … GROUP BY … HAVING … ORDER BY … LIMIT`.
- **`WHERE` vs `HAVING`** — `WHERE` filters rows *before* grouping; `HAVING` filters *after* aggregation.

## Joins
| Join | Returns |
|------|---------|
| INNER JOIN | rows matching in both tables |
| LEFT (OUTER) JOIN | all left rows + matched right (NULLs otherwise) |
| RIGHT (OUTER) JOIN | all right rows + matched left |
| FULL OUTER JOIN | all rows from both, matched where possible |
| CROSS JOIN | Cartesian product |
| SELF JOIN | table joined to itself |

`UNION` (distinct) vs `UNION ALL` (keeps duplicates).

## Normalization
Reduce redundancy & anomalies:
- **1NF** — atomic values, no repeating groups.
- **2NF** — 1NF + no partial dependency on part of a composite key.
- **3NF** — 2NF + no transitive dependency (non-key → non-key).
- **BCNF** — stricter 3NF.
- **Denormalization** — deliberately add redundancy for read performance (reporting, caching).

## Indexing
- Speeds up reads (`O(log n)` via B-tree) at the cost of slower writes + storage.
- **Clustered** (defines physical row order; one per table) vs **non-clustered** (separate structure + pointer).
- **Composite** index — multiple columns (leftmost-prefix rule). **Covering** index — includes all queried columns.
- Index columns used in `WHERE`, `JOIN`, `ORDER BY`. Avoid over-indexing (write cost).
- Watch for full table scans; use `EXPLAIN`/execution plans to tune.

## Transactions & isolation levels
ACID (see [Overview](index.md)). Isolation levels trade consistency vs concurrency:
| Level | Dirty read | Non-repeatable read | Phantom read |
|-------|-----------|---------------------|--------------|
| READ UNCOMMITTED | ✓ | ✓ | ✓ |
| READ COMMITTED | ✗ | ✓ | ✓ |
| REPEATABLE READ | ✗ | ✗ | ✓ |
| SERIALIZABLE | ✗ | ✗ | ✗ |
Locking (shared/exclusive) vs **MVCC** (Postgres/Oracle keep row versions for readers).

## Query optimization
- Select only needed columns; index `WHERE`/`JOIN` keys; avoid `SELECT *` and `N+1` queries.
- Use `EXPLAIN` to read the plan; watch for full scans, missing indexes, bad join order.
- Batch writes; paginate with keyset (cursor) for deep pages.
