# Schema Migrations and Seeding (Flyway / Liquibase)

A **migration tool** versions the database schema like source code: every change is a script applied once, in order, and recorded in a history table. This makes every environment (dev, CI, prod) reproducible and replaces manual SQL and `spring.jpa.hibernate.ddl-auto=update`.

## Flyway vs Liquibase

| Aspect | Flyway | Liquibase |
| :-- | :-- | :-- |
| Change format | Plain SQL (or Java) files | XML / YAML / JSON / SQL changelogs |
| Ordering | File name version `V1__init.sql` | Order inside the changelog |
| History table | `flyway_schema_history` | `DATABASECHANGELOG` (+ `DATABASECHANGELOGLOCK`) |
| Rollback | Undo scripts (paid tier); usually roll forward | Rollback blocks, `rollback` command |
| Strength | Simple, SQL-first | DB-agnostic abstraction, contexts, preconditions |

## Flyway in Spring Boot 3

```xml
<dependency><groupId>org.flywaydb</groupId><artifactId>flyway-core</artifactId></dependency>
<!-- Flyway 10+ needs the DB module, e.g. flyway-database-postgresql -->
<dependency><groupId>org.flywaydb</groupId><artifactId>flyway-database-postgresql</artifactId></dependency>
```

```yaml
spring:
  jpa.hibernate.ddl-auto: validate   # Hibernate only checks, never changes the schema
  flyway:
    locations: classpath:db/migration
    baseline-on-migrate: false       # true only when adopting an existing DB
```

```text
src/main/resources/db/migration/
  V1__create_customer.sql      -- versioned: runs once, never edit after release
  V2__add_customer_status.sql
  R__refresh_views.sql         -- repeatable: re-runs when its checksum changes
```

```sql
-- V2__add_customer_status.sql
ALTER TABLE customer ADD COLUMN status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE';
```

Boot runs migrations at startup, before JPA is used. Flyway stores a **checksum** per applied file; editing an applied migration makes startup fail validation. Fix by adding a new migration, not editing the old one.

## Liquibase (YAML)

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - changeSet:
      id: 2-add-status
      author: team
      changes:
        - addColumn:
            tableName: customer
            columns:
              - column: { name: status, type: varchar(20), defaultValue: ACTIVE, constraints: { nullable: false } }
      rollback:
        - dropColumn: { tableName: customer, columnName: status }
```

A changeset is identified by `id + author + file path`; a changed checksum fails validation unless `runOnChange` or `validCheckSum` is set.

## Zero-Downtime: Expand and Contract

During a rolling deploy, old and new app versions run against the **same** schema, so each change must be compatible with both. Split a breaking change into phases (example: rename `name` to `full_name`):

| Phase | Schema change | App version |
| :-- | :-- | :-- |
| 1. Expand | Add nullable `full_name` | v1 still writes `name` |
| 2. Migrate | Backfill `full_name` (in batches) | v2 writes both, reads `full_name` |
| 3. Switch | none | v3 uses only `full_name` |
| 4. Contract | Drop `name` in a later release | none |

Rules of thumb:

- Add columns as nullable or with a default; add `NOT NULL` last.
- Never rename or drop in the same release that stops using the column.
- Backfill large tables in batches to avoid long locks. In PostgreSQL use `CREATE INDEX CONCURRENTLY`, which cannot run inside a transaction (set Flyway `executeInTransaction=false` for that file).
- Prefer **roll forward** (a new fixing migration) over rollback; data changes are hard to undo.

The same idea applies to APIs: see [API Evolution](../spring-web/api-evolution.md).

## Seeding Data

| Kind | Where | Notes |
| :-- | :-- | :-- |
| Reference data (roles, countries) | Versioned or repeatable migration | Idempotent: `INSERT ... ON CONFLICT DO NOTHING` / `MERGE` |
| Dev/demo data | Extra Flyway location per profile: `spring.flyway.locations=classpath:db/migration,classpath:db/dev` | Never ship to prod |
| Test data | Testcontainers + migrations, or `@Sql` scripts | Per test, rolled back |
| One-off app bootstrap | `ApplicationRunner` guarded by an existence check | Runs on every instance, so must be idempotent |

## Operational Gotchas

- With several instances the tool takes a DB lock so only one migrates; slow migrations delay startup and health checks. Large systems run migrations as a separate step (init container or CI job) before rollout.
- The app user needs DDL rights only for migrations; use a separate migration user if policy requires.
- Hibernate `ddl-auto=update/create` is for prototypes only; use `validate` or `none` in production.
- Hand-editing the DB causes drift; detect with `flyway validate` / `liquibase diff`.
- `baselineOnMigrate` marks an existing schema as the baseline version when adopting Flyway on a legacy DB.

## Common Interview Questions

- **Why not `ddl-auto=update`?** Unpredictable, cannot drop or rename safely, no history or review; use versioned migrations.
- **A migration already ran in prod and is wrong. What now?** Add a new corrective migration; never edit an applied one (checksum fails).
- **How do you rename a column with zero downtime?** Expand and contract across several releases.
- **Flyway or Liquibase?** Flyway for SQL-first simplicity; Liquibase for multi-DB abstraction, preconditions and built-in rollback.
- **Many pods start at once?** The tool locks so one migrates; or run it as a dedicated pre-deploy job.

See also: [SQL](sql.md), [Microservices principles](../microservices-distributed/principles.md), [JPA and ORM](../spring-web/jpa-orm.md).

Reference: [Flyway docs](https://documentation.red-gate.com/flyway), [Liquibase docs](https://docs.liquibase.com/), [Spring Boot database initialization](https://docs.spring.io/spring-boot/how-to/data-initialization.html).
