# JPA & ORM

## What & why
- **ORM** maps objects ↔ relational tables. **JPA** is the spec; **Hibernate** is the common implementation.
- Reduces boilerplate JDBC; manages entity state + transactions.

## Entities & mapping
- `@Entity`, `@Id`, `@GeneratedValue`, `@Column`, `@Table`.
- Relationships: `@OneToOne`, `@OneToMany`/`@ManyToOne`, `@ManyToMany` (with `@JoinColumn`/`@JoinTable`).

## Fetching
- **LAZY** (load on access, default for collections) vs **EAGER** (load immediately).
- **N+1 problem** — one query per associated row; fix with `JOIN FETCH`, entity graphs, or batch size.

## Persistence context
- First-level cache per `EntityManager`; entity states: transient → managed → detached → removed.
- Dirty checking auto-flushes changes to managed entities at commit.

## Transactions
`@Transactional`; rollback on unchecked exceptions; propagation + isolation. Keep transactions short.

## JPA vs MyBatis
| | JPA/Hibernate | MyBatis |
|--|---------------|---------|
| Style | ORM (objects) | SQL mapper (you write SQL) |
| Control | less boilerplate | full SQL control |
| Best for | domain-model CRUD | complex/tuned queries, legacy schemas |

## Spring Data JPA
Repository interfaces (`JpaRepository`), derived queries (`findByEmail`), `@Query` for custom JPQL/native SQL.
