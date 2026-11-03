# Application Architecture and Maintainability Interview Questions

Layered, hexagonal, and clean architecture, repository organization, and keeping Java code maintainable as it grows.

---

## 1. What are the key differences between layered, hexagonal, and clean architecture?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>All three separate concerns; they differ in the <strong>direction of dependencies</strong>. Layered depends top-down (web to service to data), so the business logic ends up depending on the persistence layer. Hexagonal and clean architecture <strong>invert</strong> this: the domain sits in the center and infrastructure depends on it.</p>

<table>
<tr><th>Aspect</th><th>Layered</th><th>Hexagonal (Ports and Adapters)</th><th>Clean</th></tr>
<tr><td>Structure</td><td>Stacked layers</td><td>Domain core with ports; adapters outside</td><td>Concentric rings: entities, use cases, interface adapters, frameworks</td></tr>
<tr><td>Dependency direction</td><td>Downward, to the DB layer</td><td>Inward, to the domain</td><td>Inward, to the entities</td></tr>
<tr><td>Framework/DB coupling</td><td>Often leaks into business logic</td><td>Isolated in adapters</td><td>Isolated in the outer ring</td></tr>
<tr><td>Testing business logic</td><td>Usually needs the DB or mocks</td><td>Test the core with fake adapters</td><td>Test use cases without framework</td></tr>
<tr><td>Cost</td><td>Low, familiar</td><td>More interfaces and mapping</td><td>Most ceremony</td></tr>
</table>

<h3>Hexagonal in Practice</h3>

```java
// Port owned by the domain
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    void save(Order order);
}

// Adapter (infrastructure) implements the port
@Repository
class JpaOrderRepository implements OrderRepository { /* uses Spring Data */ }

// Domain/use-case code has no JPA or Spring Web imports
class PlaceOrderUseCase {
    private final OrderRepository orders;
    PlaceOrderUseCase(OrderRepository orders) { this.orders = orders; }
}
```

<h3>Gotchas and When to Use</h3>
<ul>
<li>Hexagonal and clean are essentially the same idea with different vocabulary (ports/adapters vs rings/use cases); both apply Dependency Inversion.</li>
<li>Use layered for simple CRUD services. Choose hexagonal/clean when the domain is rich, long-lived, or must survive framework/DB changes. Extra mapping between DTOs, domain objects, and entities is the price.</li>
<li>Strict layering ("a layer may only call the one below") vs relaxed layering is a common follow-up.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations and Components</a></p>

</details>

---

## 2. Explain the Repository-Service-Controller layered pattern

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>The standard three-layer structure of a Spring application: the <strong>Controller</strong> handles HTTP, the <strong>Service</strong> holds business logic and transactions, and the <strong>Repository</strong> handles data access. Each layer talks only to the one below.</p>

<table>
<tr><th>Layer</th><th>Responsibility</th><th>Spring stereotype</th></tr>
<tr><td>Controller</td><td>Map requests, validate input, convert to/from DTOs, choose status codes</td><td><code>@RestController</code></td></tr>
<tr><td>Service</td><td>Business rules, orchestration, transaction boundary</td><td><code>@Service</code>, <code>@Transactional</code></td></tr>
<tr><td>Repository</td><td>Persistence queries, entity mapping</td><td><code>@Repository</code>, Spring Data</td></tr>
</table>

<h3>Example</h3>

```java
@RestController
@RequestMapping("/api/users")
class UserController {
    private final UserService service;
    UserController(UserService service) { this.service = service; }

    @GetMapping("/{id}")
    UserDto get(@PathVariable Long id) { return service.findUser(id); }
}

@Service
class UserService {
    private final UserRepository repo;
    UserService(UserRepository repo) { this.repo = repo; }

    @Transactional(readOnly = true)
    UserDto findUser(Long id) {
        return repo.findById(id)
                   .map(u -> new UserDto(u.getId(), u.getName()))
                   .orElseThrow(() -> new UserNotFoundException(id));
    }
}

interface UserRepository extends JpaRepository<User, Long> { }
```

<h3>Gotchas</h3>
<ul>
<li>Do not put business logic in controllers or repositories; do not return JPA entities from controllers (use DTOs; avoids lazy-loading errors and leaking the schema).</li>
<li>Put <code>@Transactional</code> on the service layer, not the controller. Self-invocation inside a service bypasses the proxy.</li>
<li>Controllers should not call repositories directly, or logic spreads and transactions fragment.</li>
<li>"Anemic domain model" risk: services full of logic over data-only entities. Move invariants into the domain objects where sensible.</li>
<li>Use constructor injection; map exceptions to HTTP responses in one place (<code>@ControllerAdvice</code>).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-annotations-components/">Learn more: Spring Annotations and Components</a></p>

</details>

---

## 3. How do you structure a large Spring Boot monorepo vs multi-repo microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Choose by team structure and release independence, not fashion. A <strong>monorepo</strong> keeps many modules or services in one repository; <strong>multi-repo</strong> gives each service its own repository, build, and release pipeline.</p>

<table>
<tr><th>Aspect</th><th>Monorepo</th><th>Multi-repo</th></tr>
<tr><td>Code sharing / refactoring</td><td>Atomic cross-service changes, easy shared libs</td><td>Shared code needs versioned artifacts</td></tr>
<tr><td>Dependency management</td><td>One version policy (BOM)</td><td>Version drift between services</td></tr>
<tr><td>CI/CD</td><td>Needs change detection to build only affected modules</td><td>Simple per-repo pipelines</td></tr>
<tr><td>Ownership and access</td><td>Needs CODEOWNERS; coarse access control</td><td>Clear ownership, fine access control</td></tr>
<tr><td>Risk</td><td>Slow builds, tight coupling by convenience</td><td>Cross-repo changes are slow and need coordination</td></tr>
</table>

<h3>Monorepo Layout (Maven/Gradle multi-module)</h3>

```text
platform/
  pom.xml                  # parent: dependencyManagement / BOM, plugin versions
  libs/
    common-api/            # shared DTOs / contracts (keep small)
    common-observability/
  services/
    order-service/         # each a Spring Boot app with its own main class
    payment-service/
  ci/                      # pipelines: build only changed modules
```

<h3>Guidelines</h3>
<ul>
<li>Keep one parent POM or BOM for consistent dependency and Spring Boot versions.</li>
<li>Services must still be independently deployable and own their data; sharing a database or a bloated "common" module re-creates a distributed monolith.</li>
<li>Within a service, use package-by-feature or modules; enforce boundaries (for example with ArchUnit or Spring Modulith).</li>
<li>Multi-repo suits independent teams with different release cadences; publish shared libraries and API contracts as versioned artifacts and use contract tests.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Cloud and Microservices Principles</a></p>

</details>

---

## 4. How do you ensure your code is both scalable and maintainable in high-performance Java applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Make it correct and clear first, find real bottlenecks by measurement, optimize only those, and protect the result with tests and documentation. Maintainability and scalability support each other when boundaries are clean.</p>

<h3>Practices</h3>
<ul>
<li><strong>Modular design:</strong> clear module/package boundaries, small interfaces, dependencies pointing inward (see layered vs hexagonal).</li>
<li><strong>Design for horizontal scale:</strong> stateless services, externalized state (cache/DB), idempotent operations, back-pressure and timeouts.</li>
<li><strong>Use patterns where they pay off:</strong> Strategy for variability, Builder/immutability for safe objects; avoid speculative abstraction.</li>
<li><strong>Testing:</strong> unit tests for logic, integration tests for boundaries, performance/load tests for hot paths, so refactoring stays safe.</li>
<li><strong>Profile before optimizing:</strong> JFR, async-profiler, GC logs; set measurable targets (p99 latency, throughput).</li>
<li><strong>Observability:</strong> structured logs, metrics, tracing so regressions are visible.</li>
<li><strong>Documentation:</strong> short architecture notes and decision records (ADRs), readable code over comments.</li>
<li><strong>Automation:</strong> CI with static analysis, code review, dependency updates.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Premature optimization hurts both goals; micro-optimizations rarely beat fixing an N+1 query, a missing index, or lock contention.</li>
<li>Isolate any unavoidable low-level optimization behind a clear interface, with a comment on why it exists and a benchmark (JMH) that proves it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging and Performance</a></p>

</details>

---

## 5. Can you give an example where refactoring improved maintainability?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use the STAR shape: situation, the specific smell, the refactoring technique, how you kept it safe, and a measurable result. Here is a generic example you can adapt to your own experience (only claim what you really did).</p>

<h3>Example: Replacing a Type Switch with Strategy</h3>
<ul>
<li><strong>Situation:</strong> a fee calculator had a 300-line method with a growing <code>switch</code> on account type; every new type risked regressions and merge conflicts.</li>
<li><strong>Safety net:</strong> characterization tests that pinned current outputs for representative inputs, before touching anything.</li>
<li><strong>Refactoring:</strong> extracted each branch to a <code>FeePolicy</code> implementation (Replace Conditional with Polymorphism), selected via a map injected by Spring.</li>
<li><strong>Result:</strong> adding a type became one new class plus tests; method size dropped, test coverage rose, and defects in that area fell.</li>
</ul>

```java
// Before
double fee(Account a, double amount) {
    switch (a.type()) {
        case BASIC:   return amount * 0.02;
        case PREMIUM: return amount * 0.01;
        default: throw new IllegalArgumentException();
    }
}

// After
interface FeePolicy { double fee(double amount); }

class FeeService {
    private final Map<AccountType, FeePolicy> policies;
    FeeService(Map<AccountType, FeePolicy> policies) { this.policies = policies; }
    double fee(Account a, double amount) { return policies.get(a.type()).fee(amount); }
}
```

<h3>Other Good Candidates</h3>
<ul>
<li>Extract Method on a long method, splitting a god class (SRP), replacing inheritance with composition, removing duplicated code, introducing a parameter object.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/design-patterns/">Learn more: Design Patterns</a></p>

</details>

---

## 6. How do you balance performance enhancement and code clarity?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Default to clarity; optimize only what measurement proves is a bottleneck, and when you do, contain and document the complexity.</p>

<h3>Approach</h3>
<ol>
<li><strong>Set a goal:</strong> a measurable target (latency percentile, throughput, memory).</li>
<li><strong>Measure:</strong> profile (JFR, async-profiler) and benchmark (JMH, not hand-rolled timers) to find the hot path.</li>
<li><strong>Fix algorithms and I/O first:</strong> data structures, batching, caching, indexes, N+1 queries give far bigger wins than micro-tweaks.</li>
<li><strong>Isolate:</strong> put the optimized code behind a clearly named method or class with a comment explaining <em>why</em>, plus a benchmark or test that guards it.</li>
<li><strong>Re-measure</strong> and keep the change only if the gain is worth the complexity.</li>
</ol>

<h3>Example</h3>

```java
// Clear first: usually fast enough
String csv = rows.stream().map(Row::toCsv).collect(Collectors.joining("\n"));

// Optimized only after profiling showed this as a hot path (and documented):
StringBuilder sb = new StringBuilder(rows.size() * 64);   // pre-sized: avoids repeated resizing
for (Row r : rows) sb.append(r.toCsv()).append('\n');
```

<h3>Gotchas</h3>
<ul>
<li>"Premature optimization is the root of all evil" applies to <em>unmeasured</em> optimization; critical paths in latency-sensitive systems are still designed for performance up front (data layout, allocation, locking).</li>
<li>The JIT already optimizes simple code well; clever code can even defeat it. Micro-benchmarks without JMH are often misleading.</li>
<li>Trade-offs to name: readability vs speed, memory vs CPU (caching), consistency vs latency.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-patterns-advanced/logging-performance/">Learn more: Logging and Performance</a></p>

</details>

---

## 7. How do you approach a refactoring task while implementing a new feature?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Separate the two activities: <strong>make the change easy, then make the easy change</strong>. Refactor in small, behavior-preserving steps under test, ideally in separate commits (or PRs) from the feature work.</p>

<h3>Steps</h3>
<ol>
<li><strong>Understand the area:</strong> read the code and clarify scope; refactor only what the feature touches.</li>
<li><strong>Add safety:</strong> characterization/regression tests around the current behavior.</li>
<li><strong>Refactor first (no behavior change):</strong> extract methods or interfaces, remove duplication, create a seam for the new behavior. Run tests after each step.</li>
<li><strong>Implement the feature</strong> on the cleaner structure, with new tests.</li>
<li><strong>Keep commits separate:</strong> a "refactor" commit and a "feature" commit make review and rollback easy.</li>
</ol>

<h3>Risk Management</h3>
<ul>
<li>Use feature flags to ship incomplete work safely; use branch by abstraction for larger changes.</li>
<li>Keep PRs small; avoid unrelated cleanup that bloats the diff.</li>
<li>If the refactor is large, agree on it with the team and schedule it (or split into a follow-up) instead of blocking the feature.</li>
<li>Strangler approach for legacy: route new behavior to new code and migrate old paths gradually.</li>
</ul>

<h3>Follow-up: When Not to Refactor</h3>
<ul>
<li>No tests and no time to add any, imminent deadline with a hotfix, or code that is about to be deleted. Record technical debt instead.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/design-principles/">Learn more: Software Design Principles</a></p>

</details>

---
