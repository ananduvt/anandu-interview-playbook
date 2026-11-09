# Spring Data Performance: StatelessSession and HikariCP

Two data-layer tools that interviewers pair with "how do you handle high volume?": Hibernate's `StatelessSession` for bulk work, and **HikariCP** (Spring Boot's default pool) sized and monitored correctly. For persistence-context basics see [JPA and ORM](jpa-orm.md); for transaction boundaries see [Programmatic Transactions](spring-transactions.md).

## Hibernate StatelessSession

A normal `Session` (the JPA `EntityManager`) keeps a **persistence context**: first-level cache, dirty checking, cascades, lazy proxies. Loading a million rows into it exhausts memory. A `StatelessSession` has none of that: each operation goes straight to the database as SQL.

| Feature | Session / EntityManager | StatelessSession |
| :-- | :-- | :-- |
| First-level cache | Yes | No (same row loaded twice gives two objects) |
| Dirty checking | Automatic | None: call `update()` explicitly |
| Cascades | Yes | Ignored |
| Lazy loading | Yes (proxies) | No: associations are not fetched |
| Interceptors / events | Yes | Bypassed |
| Second-level cache | Yes | Bypassed |
| Use for | Business logic | ETL, bulk insert/update, large read-only scans |

```java
@Service
public class ImportService {
    private final EntityManagerFactory emf;
    ImportService(EntityManagerFactory emf) { this.emf = emf; }

    public void bulkInsert(List<Product> rows) {
        SessionFactory sf = emf.unwrap(SessionFactory.class);
        try (StatelessSession ss = sf.openStatelessSession()) {
            Transaction tx = ss.beginTransaction();
            for (Product p : rows) {
                ss.insert(p);                 // INSERT issued at once; no cache growth
            }
            tx.commit();
        }
    }

    public void scan() {
        SessionFactory sf = emf.unwrap(SessionFactory.class);
        try (StatelessSession ss = sf.openStatelessSession();
             ScrollableResults<Product> rs = ss.createQuery("from Product", Product.class)
                     .setFetchSize(500)
                     .scroll(ScrollMode.FORWARD_ONLY)) {
            while (rs.next()) process(rs.get());   // constant memory
        }
    }
}
```

Notes:

- It is a Hibernate API, not JPA. A session opened this way takes its own connection and transaction, not the Spring-managed one.
- Enable JDBC batching: `hibernate.jdbc.batch_size=50`, `hibernate.order_inserts=true`. `IDENTITY` ids disable insert batching, so prefer a sequence.
- With a normal `Session`, the alternative is to `flush()` and `clear()` every N rows.
- Often simpler: `JdbcTemplate.batchUpdate`, PostgreSQL `COPY`, repository `Stream<T>` methods, DTO projections, or Spring Batch chunks.

## HikariCP

A connection pool reuses expensive DB connections. HikariCP is the Boot default and is fast because of a lock-light design.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10          # default 10
      minimum-idle: 10               # default = max (fixed-size pool recommended)
      connection-timeout: 3000       # ms to wait for a connection, default 30000
      idle-timeout: 600000           # only matters when minimum-idle < max
      max-lifetime: 1740000          # retire before DB/LB timeout (default 30 min)
      keepalive-time: 300000
      leak-detection-threshold: 20000   # log if a connection is held over 20 s
      pool-name: orders-pool
```

### Sizing

More connections is not faster: the DB has limited cores and disks, and beyond that connections just queue and add context switching. HikariCP's guide gives a starting point:

`pool size = (core_count * 2) + effective_spindle_count`

That is the capacity of the **database** server, shared by all app instances, so `max-pool-size * instances` must stay below the DB `max_connections`. Treat it as a start and load test.

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| `Connection is not available, request timed out after 30000ms` | Pool exhausted: slow queries, leaks, long transactions | Find who holds connections; do not just raise the size |
| Pool at max, DB CPU low | Transactions hold connections while calling HTTP or idling | Shorten transactions, move I/O outside |
| Many pods, DB refuses connections | `pods * pool size` over `max_connections` | Smaller pools, or a proxy (PgBouncer, RDS Proxy) |
| Random "connection closed" errors | DB or firewall closes idle connections | `max-lifetime` below the infrastructure timeout |
| Connection held through response rendering | `spring.jpa.open-in-view=true` (default) | Set to `false` |

!!! warning "Virtual threads"
    Thousands of virtual threads can all wait on a pool of 10. The pool is the real concurrency limit for DB work; do not enlarge it to match thread count.

### Monitoring

Boot publishes Micrometer metrics (see [Actuator](spring-boot-actuator.md)):

| Metric | Meaning | Alert on |
| :-- | :-- | :-- |
| `hikaricp.connections.active` | In use | Near max for long |
| `hikaricp.connections.pending` | Threads waiting | Above 0 sustained |
| `hikaricp.connections.acquire` | Time to get a connection | p99 rising |
| `hikaricp.connections.usage` | Time a connection is held | Long means slow transactions |
| `hikaricp.connections.timeout` | Acquire timeouts | Any |

Use `leak-detection-threshold` in non-prod, and set `logging.level.com.zaxxer.hikari=DEBUG` when diagnosing.

## Common Interview Questions

- **Why does a bigger pool not help?** The DB is the bottleneck; extra connections only queue and thrash.
- **How do you size it?** Start small (about 10 per instance), load test, watch pending and usage times, respect `max_connections` across all instances.
- **Pool exhausted but DB idle?** Connections held during non-DB work (an HTTP call inside `@Transactional`), leaks, or open-in-view.
- **When use StatelessSession?** Bulk import or scan of many rows where cache, dirty checking and cascades are unwanted.
- **Why set `max-lifetime`?** To retire connections before the DB or network device silently drops them.

Reference: [HikariCP](https://github.com/brettwooldridge/HikariCP), [About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing), [Hibernate batching](https://docs.jboss.org/hibernate/orm/6.4/userguide/html_single/Hibernate_User_Guide.html#batch).
