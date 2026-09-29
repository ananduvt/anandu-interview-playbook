# Question Bank

Rapid Q&A for revision. Expand from the [Backlog](../backlog.md) over time.

## Java
- **`==` vs `equals()`?** identity vs logical equality; override `equals`+`hashCode` together.
- **HashMap vs ConcurrentHashMap?** thread-safety; bucket-level locking vs none; no null keys in CHM.
- **String immutability?** thread-safe, poolable, safe keys; use `StringBuilder` to mutate.
- **Integer overflow** `MAX_VALUE + 1`? wraps to `MIN_VALUE`.
- **`a++ + ++a`** for `a=5`? `5 + 7 = 12`, then `a=7`. (post vs pre increment)
- **`0.0/0.0`?** `NaN`; `1/0` (int) → `ArithmeticException`.
- **final vs finally vs finalize** — [see Exceptions](../java/exceptions.md).
- **fail-fast vs fail-safe iterators**; **Comparable vs Comparator**.

## Concurrency
- **volatile vs synchronized vs Atomic** — visibility vs mutual exclusion vs CAS.
- **Runnable vs Callable**; **Future vs CompletableFuture**.
- **Thread-safe singleton** — enum / double-checked locking with `volatile`.
- **Breaking double-checked locking** — needs `volatile` or it can publish a partially-constructed object.

## Spring / Web
- **DI types & why constructor injection?**
- **@Transactional** rollback rules (unchecked by default).
- **Bean scope / lifecycle**; singleton must be stateless.
- **Securing an API**, **API versioning**, **HTTP 202 Accepted** for async.

## System / DB
- **CAP**, **ACID vs BASE**, **idempotency**, **statelessness**.
- **Caching patterns**, **circuit breaker / Hystrix / Resilience4j**, **failover**.
- **SQL joins**, **`HAVING` vs `WHERE`**, **indexing**, **sharding**.
- **1M-user backend design** → [System Design](../system-design/fundamentals.md).

## Coding
- Max subarray (**Kadane**), palindrome, print 1..N without loop → [Coding Problems](../dsa/coding-problems.md).
