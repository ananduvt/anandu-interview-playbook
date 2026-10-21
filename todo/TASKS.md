# Internal Tasks

Single internal list of pending work (not published): fixes, refactors, splits, and topics still to be written. Work batch by batch; tick items off when done.
Questions to add live in `todo/to-process/QUESTIONS_TODO.json`.

---

## Batch 1 — Quick fixes

- [x] Remove employer name "TSYS EMF" from published pages (`docs/resources/index.md`, `docs/resources/code-review-guidelines/java-code-review-guidelines.md`)
- [x] Fix broken switch example in `docs/java-core/versions-features.md` (~line 406): `obj switch {` → `switch (obj) {`
- [x] `README.md`: remove references to the non-existent `AGENTS.md`, point to `CONTRIBUTING.md` only, and update the folder tree (the "garbled" characters were a terminal display issue; the file is valid UTF-8)
- [x] Register orphan pages in `mkdocs.yml` `nav:` and add each to its section `index.md` (6 of 7 were already listed on their index pages; added Design Patterns to the verbal index):
  - [x] `docs/ai-ml/rag-generative.md` (nav label for `llms-transformers.md` changed from "LLMs, RAG & Generative AI" to "LLMs & Transformers")
  - [x] `docs/interview-prep/question-bank.md` (later removed as a duplicate; items moved to Batch 7)
  - [x] `docs/interview-questions/coding/coding-stream-api-problems.md`
  - [x] `docs/interview-questions/verbal/verbal-design-patterns-questions.md`
  - [x] `docs/java-collections-concurrency/threading.md` (nav label for `concurrency.md` changed to "Concurrency & Threads"; partial overlap logged in Batch 4)
  - [x] `docs/spring-web/api-styles.md`
  - [x] `docs/spring-web/web-services.md` (partial overlap logged in Batch 4)
- [x] Run `mkdocs build --strict` and fix all warnings (12 broken links + 1 bad anchor fixed; build is clean)

## Batch 2 — `todo/` folder clean-up

- [ ] Rename `todo/to process/` → `todo/to-process/` (CONTRIBUTING.md already uses the new name)
- [ ] Add extensions to raw files: `todo/raw/todo`, `todo/raw/todo_questions`, `todo/raw/todo_topics`, `todo/processed/q1`, `todo/processed/q2`, `todo/processed/code_review_guide_lines` → `.txt`
- [ ] Process `todo/raw/todo` (single item: clean up `basics.md`, see Batch 4) and move it to `processed/`
- [ ] Process `todo/raw/todo_questions` and `todo/raw/todo_topics` into `QUESTIONS_TODO.json` (questions) / Batch 8 below (topics)
- [ ] Check `todo/processed/q1` for employer-specific terms (e.g., "CCM") before it is referenced anywhere public

## Batch 3 — `QUESTIONS_TODO.json` & README

- [ ] Recompute `count` per category, `metadata.total_questions`, and `summary` from the actual entries (the file currently claims 375 questions but contains ~179 entries)
- [ ] Migrate status values to the simplified set: `NEW`, `REVIEW`, `DUPLICATE`, `ADDED` (`SIMILAR` → `REVIEW`; `REVIEW_ADDED` / `added_to_repo: true` → `ADDED`)
- [ ] Decide whether to keep the `added_to_repo` field (redundant with `ADDED` status)
- [ ] Slim `todo/to-process/QUESTIONS_TODO_README.md` to the JSON schema + status meanings; remove wrong paths (`cat QUESTIONS_TODO.json` from repo root), the "push to main" team workflow, and the stale 215-question stats; point to CONTRIBUTING.md → Content Intake Pipeline

## Batch 4 — Content clean-up

- [ ] `docs/java-core/basics.md`: clean up formatting and split into subtopics if needed
  - Empty/stub headings: "Static" (duplicate of the Static section), "Abstract" (empty bullet), "Serialization & Deserialization"
  - `==` vs `.equals()` comparison is a broken flat list; convert to a table
  - Bare one-liners with no content: "Java out of memory, heap memory issue", "Annotations, custom annotations", "Immutable class implementation", "Java Casting"
  - Inconsistent heading levels (`**bold**` used as headings alongside `####`)
  - Malformed Baeldung marker-interface link (trailing backslash)
- [ ] Consolidate overlapping threading pages: `java-collections-concurrency/concurrency.md` and `threading.md` both cover Runnable vs Callable, synchronization, and thread pools. Keep each topic in one place and cross-link (`threading.md` uniquely has virtual threads and concurrency patterns; `concurrency.md` has CompletableFuture and IPC)
- [ ] Consolidate overlapping web/API pages: SOAP, REST, GraphQL, and gRPC appear in `foundations/web-fundamentals.md`, `spring-web/api-styles.md`, and `spring-web/web-services.md`. Pick one home per topic and cross-link. `web-services.md` uniquely has the authentication methods section
- [ ] `docs/spring-web/index.md`: the Web Services description (WSDL, content negotiation, versioning) doesn't match the page (SOAP vs REST, GraphQL, authentication methods); rewrite it. Also missing entries for WebFlux and Actuator, which are in the nav

## Batch 5 — Split oversized files (limits in CONTRIBUTING.md → Size Limits)

Learning pages (max 800 lines): none over the limit right now. Watch `docs/spring-web/jpa-orm.md` (673), `docs/spring-web/spring-boot-advanced.md` (640), and `docs/devops/kubernetes-advanced.md` (637).

Question banks (max 25 questions per file), counts to verify:
- [ ] `docs/interview-questions/verbal/verbal-spring-boot-questions.md` (786 lines)
- [ ] `docs/interview-questions/verbal/verbal-design-patterns-questions.md` (645 lines)
- [ ] `docs/interview-questions/verbal/verbal-microservices-questions.md` (602 lines)
- [ ] Count questions in all other `verbal-*` and `coding-*` files; split any over 25

Long-form references (`docs/resources/`) are exempt.

## Batch 6 — Navigation structure (max 8 pages per learning nav group; question groups exempt)

- [ ] Spring (14 pages): add sub-groups in `mkdocs.yml`, e.g. Core (Overview, Spring Boot, Annotations, Advanced, Exceptions) / Security (Security & RBAC, JWT & OAuth2) / Data (JPA & ORM) / Resilience & Async (Circuit Breaker, Threading, WebFlux, Actuator)
- [ ] DevOps (9 pages): add sub-groups, e.g. Build & Delivery (Git, Build Tools, CI/CD) / Containers & Kubernetes (Docker, Kubernetes, Deployment Strategies, Advanced) / Observability
- [ ] Update section `index.md` pages to reflect the sub-groups

## Batch 7 — Question bank intake

- [ ] Review `REVIEW` items in `QUESTIONS_TODO.json` against the existing `docs/interview-questions/` files
- [ ] Add `NEW` questions with answers, batch by batch (one category per batch), following CONTRIBUTING.md → Adding Interview Questions
- [ ] Create or extend learning pages for new concept areas the questions introduce (e.g., Kafka deep-dive, secrets management, rate limiting)
- [ ] Coverage check for items from the removed `docs/interview-prep/question-bank.md` (one-line revision prompts). For each, confirm a full answer exists in the matching `verbal-*` / `coding-*` file; add a question if it doesn't:
  - Java (`verbal-java-core-questions.md`):
    - `==` vs `equals()` (identity vs logical equality; override `equals` + `hashCode` together)
    - HashMap vs ConcurrentHashMap (thread safety, bucket-level locking, no null keys in CHM)
    - String immutability (thread-safe, poolable, safe map keys; `StringBuilder` to mutate)
    - Integer overflow: `Integer.MAX_VALUE + 1` wraps to `MIN_VALUE`
    - `a++ + ++a` for `a = 5` → `5 + 7 = 12`, then `a = 7` (post vs pre increment)
    - `0.0 / 0.0` → `NaN`; `1 / 0` (int) → `ArithmeticException`
    - final vs finally vs finalize
    - Fail-fast vs fail-safe iterators; Comparable vs Comparator
  - Concurrency (`verbal-java-advanced-questions.md`):
    - volatile vs synchronized vs Atomic (visibility vs mutual exclusion vs CAS)
    - Runnable vs Callable; Future vs CompletableFuture
    - Thread-safe singleton (enum, double-checked locking with `volatile`) and why double-checked locking breaks without `volatile` (publishes a partially constructed object)
  - Spring / Web (`verbal-spring-boot-questions.md`):
    - DI types and why constructor injection
    - `@Transactional` rollback rules (unchecked exceptions by default)
    - Bean scope and lifecycle; singletons must be stateless
    - Securing an API; API versioning; HTTP `202 Accepted` for async work
  - System / DB (`verbal-system-design-questions.md`, `verbal-db-questions.md`):
    - CAP; ACID vs BASE; idempotency; statelessness
    - Caching patterns; circuit breaker (Hystrix / Resilience4j); failover
    - SQL joins; `HAVING` vs `WHERE`; indexing; sharding
    - Designing a backend for 1M users
  - Coding (`coding-problems.md`): max subarray (Kadane), palindrome check, print 1..N without loops

## Batch 8 — Content gaps (topics to write)

Moved from the former `docs/backlog.md`. Items marked *verify* have partial mentions in `docs/`; read the existing coverage before writing, and expand rather than duplicate.

**Java Core** (`docs/java-core/`)
- [ ] Basic CLI: `java`, `javac`, project folder layout (`bin`, `build`, `META-INF`, `resources`) — *verify*: `basics.md` mentions `javac`
- [ ] Pass-by-value deep-dive (primitives vs object references)
- [ ] Bitwise operations and common tricks
- [ ] Conversions: List ↔ array ↔ Stream, int ↔ String
- [ ] Marker annotations (alongside the existing marker-interface stub in `basics.md`)

**Java Collections & Concurrency** (`docs/java-collections-concurrency/`)
- [ ] `hashCode` collisions and how `HashSet` works internally (backed by `HashMap`) — *verify*: covered in `verbal-java-core-questions.md`, missing from `collections.md`
- [ ] `AtomicReference` — *verify*: one mention in `threading.md`
- [ ] Handling transactions without `@Transactional` (`TransactionTemplate`, programmatic transactions)

**Java Patterns & Advanced** (`docs/java-patterns-advanced/`)
- [ ] Thread-safe Singleton (double-checked locking, holder idiom, enum) — *verify*: `verbal-design-patterns-questions.md`
- [ ] Lombok annotations (`@Data`, `@Builder`, `@Value`, `@Slf4j`, pitfalls) — *verify*: passing mentions only
- [ ] Reflection basics and annotation processing — *verify*: passing mentions only

**Spring & Web** (`docs/spring-web/`)
- [ ] Spring Security config for role-based login; securing an API (`hasRole`, method security) — *verify*: `security.md`
- [ ] API versioning strategies; returning `202 Accepted` for async work — *verify*: `foundations/web-fundamentals.md` has "Async Operations (202 Accepted)"; `spring-web/index.md` lists versioning options
- [ ] HTTP methods deep-dive; `PATCH` vs `PUT` (idempotency, partial updates) — *verify*: `spring-web/api-styles.md` has "HTTP Methods Deep-Dive"
- [ ] Server-side templating: Thymeleaf, FreeMarker (FTL)

**Microservices & Distributed Systems** (`docs/microservices-distributed/`)
- [ ] gRPC / RPC calls (vs REST, protobuf, streaming) — *verify*: `spring-web/api-styles.md` has a gRPC section
- [ ] Distributed consensus (Raft, Paxos, leader election) and eventual consistency
- [ ] Sidecar pattern — *verify*: `devops/kubernetes-advanced.md` (Istio context)

**Cloud Services** (`docs/distributed-cloud/`)
- [ ] Infrastructure as Code (Terraform, CloudFormation)
- [ ] Multi-cloud and hybrid cloud patterns
- [ ] Cloud cost optimization
- [ ] Disaster recovery and backup strategies (RPO/RTO) — *verify*: `system-design/scalability-resilience.md` failover section

**DevOps** (`docs/devops/`)
- [ ] Git: `stash`, amending the author (`git commit --amend --author="Name <email>"`) — merge vs rebase is already in `git.md`
- [ ] Common environments: dev / QA / UAT / prod and promotion between them
- [ ] SonarQube (quality gates, running in CI without IDE integration) — *verify*: one mention in `ci-cd.md`

**Coding Exercises** (`docs/interview-questions/coding/`)
- [ ] Print 1–100 without loops (recursion, streams)
- [ ] Sort students by marks and assign rank (including ties)
- [ ] Dataset operations; regex practice (link to `foundations/regex.md`)

**Interview Question Sets** (`todo/to-process/QUESTIONS_TODO.json`)
- [ ] Add a question set covering Java collections, threads, synchronization, Spring Boot, microservices, and Kafka (source: a past interview). Don't name the company in `docs/`.
