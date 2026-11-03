# System Design: Issue Tracker Interview Questions

Design questions for a Jira-like issue tracking system: scaling, service decomposition, testing and API contracts.

---

## 1. How would you improve performance to handle 2M+ users in a Jira-like system?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Start with requirements and access patterns (the workload is read-heavy: boards, search and issue views), find the bottlenecks, and scale in layers: stateless app tier, caching, database optimization, search offloading and asynchronous work.</p>

<h3>Estimate First</h3>
<p>2M registered users does not mean 2M concurrent. A rough guess is 5 to 10 percent daily active and a small fraction of those at peak, so peak traffic is perhaps hundreds to a few thousand requests per second. State your assumptions out loud.</p>

<h3>Scaling Levers</h3>
<table>
<tr><th>Layer</th><th>Technique</th></tr>
<tr><td>Edge</td><td>CDN for static assets, load balancer, rate limiting</td></tr>
<tr><td>App tier</td><td>Stateless services behind a load balancer, horizontal autoscaling, sessions or JWT out of memory</td></tr>
<tr><td>Caching</td><td>Redis for hot issues, project metadata, permissions and board views. Cache-aside with TTL and event-based invalidation. HTTP caching with ETags</td></tr>
<tr><td>Database</td><td>Right indexes (project, assignee, status, updated), read replicas for reads, connection pooling, partition or shard by project or tenant ID</td></tr>
<tr><td>Search</td><td>Move JQL-style search and filters to Elasticsearch or OpenSearch, fed by change events</td></tr>
<tr><td>Async</td><td>Notifications, webhooks, indexing, reports and audit logs through Kafka or queues</td></tr>
<tr><td>API</td><td>Pagination (cursor based), field selection, avoid N+1 queries, batch endpoints</td></tr>
</table>

<h3>Trade-offs</h3>
<ul>
<li>Caches and search indexes are eventually consistent. Make the issue view read your own writes (read from the primary or invalidate on write).</li>
<li>Sharding by project keeps most queries single-shard but creates hot projects. Big tenants may need dedicated shards.</li>
<li>Heavy features (reports, bulk edits, dashboards) are isolated in their own services so they cannot slow interactive traffic.</li>
</ul>

<h3>Validate</h3>
<p>Load test with realistic traffic mixes, set SLOs (for example p95 under 300 ms), and monitor p99, cache hit rate and DB load as you grow.</p>

<p>📚 <a href="/anandu-interview-playbook/system-design/scalability-resilience/">Learn more: Scalability & Resilience</a></p>

</details>

---

## 2. What microservices would a Jira-like system have?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Split by business capability (bounded context): each service owns its data and exposes an API or events. Put an API gateway in front for routing, authentication and rate limiting.</p>

<h3>Services</h3>
<table>
<tr><th>Service</th><th>Responsibility</th><th>Owns</th></tr>
<tr><td>Auth / identity</td><td>Login, SSO, tokens, permissions and roles</td><td>Users, roles, permission schemes</td></tr>
<tr><td>Project service</td><td>Projects, boards, components, versions, membership</td><td>Project metadata</td></tr>
<tr><td>Issue service</td><td>Create, update and link issues, fields, comments, attachments metadata</td><td>Issues (core write model)</td></tr>
<tr><td>Workflow service</td><td>States, transitions, validators and rules</td><td>Workflow definitions</td></tr>
<tr><td>Search service</td><td>Full-text and filter queries</td><td>Search index (read model)</td></tr>
<tr><td>Notification service</td><td>Email, in-app and webhook notifications</td><td>Subscriptions, delivery log</td></tr>
<tr><td>Attachment service</td><td>File upload and download (object storage)</td><td>Blobs</td></tr>
<tr><td>Audit / reporting</td><td>History, dashboards and analytics</td><td>Event store, aggregates</td></tr>
</table>

<h3>Communication</h3>
<ul>
<li><strong>Sync (REST or gRPC):</strong> issue service asks the project and permission services for validation.</li>
<li><strong>Async events:</strong> <code>IssueCreated</code>, <code>IssueTransitioned</code> feed search indexing, notifications and audit. Use the outbox pattern for reliable publishing.</li>
</ul>

<h3>Gotchas</h3>
<ul>
<li>Do not over-split. Issues, comments and fields change together and can live in one service.</li>
<li>Workflow transitions touching several services need a saga or a single owner, not a distributed transaction.</li>
<li>Permission checks are on the hot path, so cache them and invalidate on change events.</li>
<li>Start as a modular monolith and extract services when scale or team boundaries demand it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/worked-examples/">Learn more: System Design Examples</a></p>

</details>

---

## 3. How would you test microservices in a Jira-like system?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use the test pyramid with an extra layer for contracts: many fast unit tests, component and integration tests per service with real infrastructure, contract tests between services, and a few end-to-end journeys. Add non-functional tests and production monitoring.</p>

<h3>Layers</h3>
<table>
<tr><th>Layer</th><th>What to test (example)</th><th>Tools</th></tr>
<tr><td>Unit</td><td>Workflow transition rules, permission logic, JQL parsing</td><td>JUnit 5, Mockito</td></tr>
<tr><td>Integration / component</td><td>Issue service with a real DB, repository queries, Kafka publishing</td><td>Spring Boot Test, Testcontainers</td></tr>
<tr><td>Contract</td><td>Issue service and Project service agree on API and event schemas</td><td>Pact, Spring Cloud Contract</td></tr>
<tr><td>End-to-end</td><td>Create project, create issue, transition it, see it in search and a notification</td><td>REST-assured, Playwright or Selenium</td></tr>
<tr><td>Non-functional</td><td>Load, soak and resilience tests (dependency down, slow)</td><td>Gatling, JMeter, k6, chaos tools</td></tr>
</table>

<h3>Specific Concerns</h3>
<ul>
<li><strong>Async flows:</strong> wait for the search index or notification with polling (Awaitility), not fixed sleeps.</li>
<li><strong>Contracts:</strong> consumers publish expectations and the provider pipeline verifies them, so a breaking change fails before deployment.</li>
<li><strong>Test data:</strong> create and clean data per test, and isolate per tenant or project.</li>
<li><strong>Security:</strong> test permission rules, since a mistake leaks private projects.</li>
</ul>

<h3>Gotchas</h3>
<p>Keep E2E tests few and stable, because they are slow and flaky. Use canary releases, feature flags and good observability to catch what tests miss.</p>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 4. What API contracts would you define between a Jira-like system and its clients?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A resource-oriented, versioned REST API (JSON over HTTPS) described with OpenAPI, with consistent conventions for pagination, errors, authentication and concurrency, and a clear backward-compatibility policy.</p>

<h3>Example Endpoints</h3>

```text
POST   /api/v1/projects/{key}/issues            create issue
GET    /api/v1/issues/{id}                      get issue
PATCH  /api/v1/issues/{id}                      partial update
POST   /api/v1/issues/{id}/transitions          move through workflow
GET    /api/v1/issues?jql=...&cursor=...&limit=50   search with pagination
POST   /api/v1/issues/{id}/comments             add comment
```

<h3>Contract Conventions</h3>
<ul>
<li><strong>Auth:</strong> OAuth 2.0 / OIDC bearer tokens or API tokens, with scopes and per-project permission checks.</li>
<li><strong>Pagination:</strong> cursor based for large, changing lists, with a max page size.</li>
<li><strong>Errors:</strong> consistent shape (RFC 9457 problem details), correct status codes (400, 401, 403, 404, 409, 429).</li>
<li><strong>Concurrency:</strong> ETag with <code>If-Match</code>, or a version field, to prevent lost updates (409 or 412 on conflict).</li>
<li><strong>Idempotency:</strong> <code>Idempotency-Key</code> header on create operations.</li>
<li><strong>Rate limiting:</strong> 429 with <code>Retry-After</code>.</li>
<li><strong>Webhooks or events</strong> for pushing changes, with signed payloads.</li>
</ul>

<h3>Versioning and Compatibility</h3>
<table>
<tr><th>Change</th><th>Compatible?</th></tr>
<tr><td>Add optional field or new endpoint</td><td>Yes</td></tr>
<tr><td>Remove or rename a field, change a type or meaning</td><td>No. Needs a new version</td></tr>
<tr><td>Add a new enum value</td><td>Risky. Tell clients to tolerate unknown values</td></tr>
</table>
<ul>
<li>Version in the URI (<code>/v1</code>) or a header. Deprecate with the <code>Sunset</code> header and a long notice period.</li>
<li>Clients should ignore unknown fields (tolerant reader). Check compatibility with contract tests and OpenAPI diff tools in CI.</li>
<li>Publish the OpenAPI spec and generate SDKs and docs from it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/api-styles/">Learn more: API Styles</a></p>

</details>

---
