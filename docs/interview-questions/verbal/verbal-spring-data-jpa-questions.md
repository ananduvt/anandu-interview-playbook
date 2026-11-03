# Spring Data & JPA Interview Questions

JPA/Hibernate behaviour, Spring Data repositories, fetching strategies, and transaction pitfalls.

---

## 1. Describe the purpose and usage of @EntityScan

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@EntityScan</code> (Spring Boot) tells JPA which packages contain <code>@Entity</code> classes when they are not under the main application class's package, which is the default scan root.</p>

```java
@SpringBootApplication
@EntityScan(basePackages = "com.example.domain")
@EnableJpaRepositories(basePackages = "com.example.persistence")
public class App { }
```

<h3>Key Points</h3>
<ul>
<li>Entity scanning is separate from component scanning. <code>@ComponentScan</code> finds beans; <code>@EntityScan</code> finds entities; <code>@EnableJpaRepositories</code> finds repository interfaces.</li>
<li>Typical symptom of a missing scan: "Not a managed type: class ..." or "Unknown entity".</li>
<li>Useful in multi-module projects where the entities live in a shared library module.</li>
<li>With multiple <code>EntityManagerFactory</code>s, set packages on each factory (<code>packagesToScan</code>) rather than relying on <code>@EntityScan</code>.</li>
<li>Specifying <code>@EntityScan</code> replaces the default (main-package) scan, so list every needed package.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 2. How does @Transactional work? Does it work on a private method?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>@Transactional</code> is implemented with AOP proxies. The proxy intercepts the call, opens or joins a transaction through the <code>PlatformTransactionManager</code>, invokes the real method, then commits, or rolls back on a runtime exception. It <strong>does not work on private methods</strong> (nor on self-invoked calls).</p>

<h3>Why It Fails</h3>
<ul>
<li>The proxy can only intercept calls that come through it. A <code>private</code> method is not visible to the CGLIB subclass proxy, and a JDK proxy only sees interface methods.</li>
<li><strong>Self-invocation:</strong> <code>this.save()</code> inside the same bean bypasses the proxy even if <code>save()</code> is public and annotated.</li>
<li><code>final</code> methods and classes cannot be overridden by CGLIB.</li>
<li>Since Spring 6, <code>protected</code> and package-visible methods on class-based proxies are supported, but <code>public</code> is still the safe choice.</li>
<li>The bean must be Spring-managed (<code>new Service()</code> has no proxy).</li>
</ul>

```java
@Service
class OrderService {
    public void placeOrder(Order o) {
        this.persist(o);            // proxy bypassed: NO transaction from persist()'s annotation
    }
    @Transactional
    public void persist(Order o) { ... }
}
```

<h3>Fixes</h3>
<ul>
<li>Put <code>@Transactional</code> on the public entry method.</li>
<li>Move the method to another bean and inject it.</li>
<li>Use <code>TransactionTemplate</code> for programmatic control.</li>
<li>AspectJ mode (<code>@EnableTransactionManagement(mode = ASPECTJ)</code>) works for private methods and self-calls but is rarely used.</li>
</ul>

<h3>Related</h3>
<p>Rollback rules (unchecked only by default) and propagation are covered in the Spring Boot questions page ("How does @Transactional work and what are its rollback rules?").</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot/">Learn more: Spring Boot</a></p>

</details>

---

## 3. What are fetch types? Explain lazy vs eager loading

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p><code>FetchType</code> controls when an association is loaded. <strong>EAGER</strong>: loaded together with the owner entity. <strong>LAZY</strong>: loaded on first access, via a proxy or lazy collection.</p>

<table>
<tr><th>Association</th><th>JPA default</th></tr>
<tr><td><code>@ManyToOne</code>, <code>@OneToOne</code></td><td>EAGER</td></tr>
<tr><td><code>@OneToMany</code>, <code>@ManyToMany</code></td><td>LAZY</td></tr>
</table>

<table>
<tr><th></th><th>LAZY</th><th>EAGER</th></tr>
<tr><td>Queries</td><td>Extra query on access</td><td>Joined or extra queries always</td></tr>
<tr><td>Risk</td><td><code>LazyInitializationException</code> outside a session; N+1</td><td>Loads unneeded data; cannot turn it off per query</td></tr>
<tr><td>Memory</td><td>Lower</td><td>Higher</td></tr>
</table>

```java
@Entity
class Order {
    @ManyToOne(fetch = FetchType.LAZY)   // override the EAGER default
    private Customer customer;

    @OneToMany(mappedBy = "order")       // LAZY by default
    private List<Line> lines = new ArrayList<>();
}
```

<h3>Best Practice</h3>
<ul>
<li>Map everything LAZY, then fetch what a use case needs with <code>JOIN FETCH</code>, <code>@EntityGraph</code>, or DTO projections.</li>
<li>Do not "fix" <code>LazyInitializationException</code> with EAGER or open-session-in-view; fetch explicitly inside the transaction. (Boot enables open-in-view by default; many teams disable it with <code>spring.jpa.open-in-view=false</code>.)</li>
<li><code>@Lob</code> and basic attributes can be lazy only with bytecode enhancement.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 4. How does using an ORM framework influence how you write code?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>An ORM like Hibernate lets you think in objects and relationships instead of SQL and result sets, but you must understand its persistence context, fetching, and generated SQL to avoid performance and correctness traps.</p>

<h3>Effects on Your Code</h3>
<ul>
<li><strong>Entities and state:</strong> managed entities are tracked; changes are flushed automatically (dirty checking), so a setter call inside a transaction updates the DB without <code>save()</code>.</li>
<li><strong>Mapping:</strong> entities need a no-arg constructor, identity (<code>@Id</code>), and careful <code>equals/hashCode</code>.</li>
<li><strong>Transactions</strong> define the unit of work and session lifetime; lazy loading depends on them.</li>
<li><strong>Query style:</strong> derived queries, JPQL/HQL, Criteria, with native SQL as an escape hatch.</li>
<li><strong>Less boilerplate:</strong> no manual mapping, DB portability, caching (first-level, optional second-level).</li>
</ul>

<h3>Costs</h3>
<ul>
<li>Hidden SQL: N+1, over-fetching, unexpected flushes. Enable SQL logging in dev.</li>
<li>Object-relational mismatch with inheritance, bulk operations, and complex reporting queries (use projections or native SQL).</li>
<li>Bidirectional relationships need helper methods to keep both sides in sync.</li>
</ul>

<h3>When to Use</h3>
<p>Good for CRUD-heavy domain models. For SQL-centric or reporting-heavy workloads consider jOOQ, JdbcTemplate, or MyBatis (see the JPA vs MyBatis question in the Spring Boot questions page).</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 5. How do entities and DTOs relate? Why separate them?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>An <strong>entity</strong> models persistent state and is tied to the persistence context; a <strong>DTO</strong> models the data crossing an API or layer boundary. Raw entities should generally <strong>not</strong> be returned from REST endpoints.</p>

<h3>Why Separate</h3>
<ul>
<li><strong>Security:</strong> avoid exposing internal fields (password hash, audit columns) and mass-assignment on input.</li>
<li><strong>Lazy loading:</strong> serialising an entity can trigger lazy loads (N+1) or <code>LazyInitializationException</code>.</li>
<li><strong>Cycles:</strong> bidirectional relations cause infinite JSON recursion.</li>
<li><strong>Decoupling:</strong> schema changes should not break API contracts; versioning is easier with DTOs.</li>
<li><strong>Shape:</strong> API often needs a subset or aggregation, not the table layout.</li>
</ul>

```java
public record UserDto(Long id, String name, String email) {
    static UserDto from(User u) { return new UserDto(u.getId(), u.getName(), u.getEmail()); }
}

// Projection avoids loading the full entity
interface UserSummary { Long getId(); String getName(); }
List<UserSummary> findByActiveTrue();
```

<h3>Mapping Options</h3>
<ul>
<li>Manual mapping or records for small models; MapStruct for compile-time generated mappers.</li>
<li>Spring Data projections (interface or class-based) or JPQL constructor expressions read DTOs directly.</li>
</ul>

<h3>Trade-offs</h3>
<p>More classes and mapping code; for tiny internal CRUD apps returning entities can be pragmatic, but it is a trap as the app grows.</p>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 6. How does Spring Data JPA repository derivation work?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Spring Data parses a repository method name (<code>findByLastNameAndAgeGreaterThan</code>) into a query tree at startup, builds the JPQL/Criteria query, and provides the implementation through a runtime proxy. No implementation class is written.</p>

<h3>Mechanics</h3>
<ul>
<li>Subject prefixes: <code>find...By</code>, <code>read</code>, <code>get</code>, <code>query</code>, <code>count...By</code>, <code>exists...By</code>, <code>delete...By</code>; <code>Distinct</code>, <code>Top3</code>/<code>First</code> limit results.</li>
<li>Predicates on property paths: <code>And</code>, <code>Or</code>, <code>Between</code>, <code>LessThan</code>, <code>Like</code>, <code>In</code>, <code>IsNull</code>, <code>IgnoreCase</code>, <code>OrderBy...Desc</code>; nested paths like <code>findByAddressCity</code>.</li>
<li>Parameters bind in order; special parameters <code>Pageable</code>, <code>Sort</code>.</li>
<li>Method names are validated at startup, so a typo fails application start (fail fast).</li>
</ul>

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByLastNameAndAgeGreaterThan(String lastName, int age);
    Optional<User> findFirstByEmailIgnoreCase(String email);
    Page<User> findByActiveTrue(Pageable pageable);
    boolean existsByEmail(String email);

    @Query("select u from User u where u.status = :s")   // explicit alternative
    List<User> byStatus(@Param("s") Status s);
}
```

<h3>Lookup Order and Alternatives</h3>
<ul>
<li>Strategy: <code>@Query</code> (or named query) first, otherwise derive from the name.</li>
<li>Long names get unreadable; switch to <code>@Query</code>, Specifications/Querydsl for dynamic filters, or native queries.</li>
<li>Derived queries do not avoid N+1; add <code>@EntityGraph</code> where needed.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---

## 7. Explain the N+1 query problem and how to solve it

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>N+1 occurs when loading N parent entities triggers one extra query per parent to load a lazy association: 1 query for the list plus N queries for children, which scales badly.</p>

```java
List<Order> orders = orderRepo.findAll();            // 1 query
for (Order o : orders) {
    o.getLines().size();                             // +1 query per order (N)
}
```

<h3>Solutions</h3>
<table>
<tr><th>Approach</th><th>Example</th><th>Note</th></tr>
<tr><td>Fetch join</td><td><code>select o from Order o join fetch o.lines</code></td><td>One query; with collections, use <code>distinct</code> and avoid pagination (in-memory paging warning)</td></tr>
<tr><td>Entity graph</td><td><code>@EntityGraph(attributePaths = "lines")</code></td><td>Declarative, reusable on repository methods</td></tr>
<tr><td>Batch fetching</td><td><code>@BatchSize(size = 50)</code> or <code>hibernate.default_batch_fetch_size</code></td><td>Turns N queries into N/50 using <code>IN</code> lists; works with paging</td></tr>
<tr><td>DTO projection</td><td>Select only needed columns</td><td>Best for read-only views</td></tr>
</table>

<h3>Gotchas</h3>
<ul>
<li>Switching to EAGER does not fix it; it can still issue N extra queries and removes control.</li>
<li>Fetching two <code>List</code> collections at once causes <code>MultipleBagFetchException</code> and a cartesian product; use Sets, separate queries, or batch fetching.</li>
<li>Detect it: enable <code>hibernate.show_sql</code>/statistics, use datasource-proxy or Hibernate statistics, and assert query counts in tests.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/jpa-orm/">Learn more: JPA & ORM</a></p>

</details>

---
