# Microservices & Distributed Systems Interview Questions

Architecture patterns, scalability, resilience, messaging, and distributed computing principles.

---

## 1. When should you use microservices vs monolith?

<details>
<summary>View Answer</summary>

<h3>Monolith: Single Unified Application</h3>

<p><strong>All features in one codebase, one deployment unit.</strong></p>

<p><strong>Advantages:</strong></p>
<ul>
<li>✅ Simple to build initially</li>
<li>✅ Easier debugging (single process)</li>
<li>✅ Single database (ACID transactions)</li>
<li>✅ Better performance (in-process calls)</li>
</ul>

<p><strong>Disadvantages:</strong></p>
<ul>
<li>❌ Hard to scale (scale entire app)</li>
<li>❌ Tight coupling (changes affect entire system)</li>
<li>❌ Technology lock-in (one language/framework)</li>
<li>❌ Deployment risk (one bug breaks everything)</li>
</ul>

<h3>Microservices: Distributed Services</h3>

<p><strong>Multiple independent services, each with own responsibility.</strong></p>

<p><strong>Advantages:</strong></p>
<ul>
<li>✅ Independent scaling (scale what you need)</li>
<li>✅ Loose coupling (change one without affecting others)</li>
<li>✅ Technology freedom (different stacks per service)</li>
<li>✅ Faster deployment (smaller units)</li>
<li>✅ Fault isolation (one service down ≠ entire system down)</li>
</ul>

<p><strong>Disadvantages:</strong></p>
<ul>
<li>❌ Complex distributed system (network issues, latency)</li>
<li>❌ Harder debugging (logs across multiple services)</li>
<li>❌ Data consistency challenges (eventual consistency)</li>
<li>❌ Operational overhead (monitoring, deployment, versioning)</li>
<li>❌ Network latency (inter-service calls)</li>
</ul>

<h3>Choose Monolith When</h3>

<ul>
<li>MVP/startup (speed to market)</li>
<li>Small team (coordination overhead)</li>
<li>Simple, well-defined domain</li>
<li>ACID transactions critical</li>
<li>Low scalability requirements</li>
</ul>

<h3>Choose Microservices When</h3>

<ul>
<li>Large team (independent development)</li>
<li>Scalability required (different services scale differently)</li>
<li>Technology diversity needed</li>
<li>Independent deployment important</li>
<li>Fault isolation critical</li>
</ul>

<h3>Recommendation</h3>

<p><strong>Start with monolith, migrate to microservices when you outgrow it.</strong> Most companies shouldn't start with microservices.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 2. What's the difference between synchronous and asynchronous communication?

<details>
<summary>View Answer</summary>

<h3>Synchronous Communication</h3>

<p><strong>Caller waits for response (blocking).</strong></p>

<p><strong>Examples:</strong> REST API calls, gRPC, direct method calls</p>

```
Client calls Service A
Client blocks, waiting for response
Service A processes and returns result
Client receives response
```

<p><strong>Advantages:</strong></p>
<ul>
<li>✅ Simple to implement</li>
<li>✅ Immediate feedback</li>
<li>✅ Error handling straightforward</li>
</ul>

<p><strong>Disadvantages:</strong></p>
<ul>
<li>❌ Tight coupling (client waits for server)</li>
<li>❌ Cascading failures (if Service A slow, client slow)</li>
<li>❌ Less resilient (network issues block caller)</li>
<li>❌ Poor scalability (threads waiting)</li>
</ul>

<h3>Asynchronous Communication</h3>

<p><strong>Caller doesn't wait for response (fire-and-forget or callback).</strong></p>

<p><strong>Examples:</strong> Message queues (Kafka, RabbitMQ), event-driven, webhooks</p>

```
Client sends message to Queue
Client continues immediately
Service A consumes message when ready
Service A processes and publishes result event
Client receives event via callback/subscription
```

<p><strong>Advantages:</strong></p>
<ul>
<li>✅ Loose coupling (caller doesn't wait)</li>
<li>✅ Better resilience (queue buffers load)</li>
<li>✅ Scalability (no thread blocking)</li>
<li>✅ Can handle spikes (queue absorbs load)</li>
</ul>

<p><strong>Disadvantages:</strong></p>
<ul>
<li>❌ Complex (eventual consistency, ordering issues)</li>
<li>❌ Debugging harder (distributed flow)</li>
<li>❌ Guaranteeing delivery complex</li>
<li>❌ Error handling less obvious</li>
</ul>

<h3>Comparison Table</h3>

<table>
<tr>
<th>Aspect</th>
<th>Synchronous</th>
<th>Asynchronous</th>
</tr>
<tr>
<td><strong>Latency</strong></td>
<td>Immediate but may be slow</td>
<td>Delayed but responsive</td>
</tr>
<tr>
<td><strong>Coupling</strong></td>
<td>Tight</td>
<td>Loose</td>
</tr>
<tr>
<td><strong>Complexity</strong></td>
<td>Simple</td>
<td>Complex</td>
</tr>
<tr>
<td><strong>Resilience</strong></td>
<td>Poor (cascading failures)</td>
<td>Good (buffering, retry)</td>
</tr>
<tr>
<td><strong>Best For</strong></td>
<td>Request-response (GET, queries)</td>
<td>Long-running operations, events</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Events</a></p>

</details>

---

## 3. Explain the saga pattern for distributed transactions

<details>
<summary>View Answer</summary>

<h3>The Problem: Distributed Transactions</h3>

<p>ACID transactions don't work across multiple databases/services. Need alternative pattern.</p>

<p><strong>Example: Money Transfer</strong></p>
<ul>
<li>Service A: Debit account from_account</li>
<li>Service B: Credit account to_account</li>
<li>If Service B fails, Service A's debit is stuck!</li>
</ul>

<h3>The Saga Pattern: Two Approaches</h3>

<h4>Orchestration (Centralized Coordinator)</h4>

<p><strong>One service orchestrates the saga steps.</strong></p>

```
Saga Orchestrator
  ↓ Step 1: Debit Account A
  ← Success
  ↓ Step 2: Credit Account B
  ← Failure → Compensate (refund A)
```

<p><strong>Code Example:</strong></p>

```java
@Service
public class TransferSaga {
    public void transfer(String fromId, String toId, BigDecimal amount) {
        try {
            Step 1: accountService.debit(fromId, amount);
            Step 2: accountService.credit(toId, amount);
        } catch (Exception e) {
            // Compensate: reverse the debit
            accountService.credit(fromId, amount);
        }
    }
}
```

<h4>Choreography (Event-Driven)</h4>

<p><strong>Each service listens for events and publishes new events.</strong></p>

```
AccountService: "TransferInitiated" event
  → (publishes) "MoneyDebited"
    → LedgerService listens
    → (publishes) "MoneyCredited"
      → If fails: (publishes) "TransferFailed"
        → AccountService listens and refunds
```

<h3>Comparison</h3>

<table>
<tr>
<th>Aspect</th>
<th>Orchestration</th>
<th>Choreography</th>
</tr>
<tr>
<td><strong>Coordination</strong></td>
<td>Centralized (Saga Orchestrator)</td>
<td>Decentralized (event listeners)</td>
</tr>
<tr>
<td><strong>Complexity</strong></td>
<td>Simpler to understand</td>
<td>Complex (distributed logic)</td>
</tr>
<tr>
<td><strong>Coupling</strong></td>
<td>Tight (orchestrator knows all steps)</td>
<td>Loose (services listen to events)</td>
</tr>
<tr>
<td><strong>Testing</strong></td>
<td>Easier (centralized logic)</td>
<td>Harder (distributed)</td>
</tr>
</table>

<h3>Compensating Transactions (Rollback)</h3>

<p>For each forward step, define a compensating transaction:</p>

<ul>
<li><strong>Debit:</strong> Compensate with Credit</li>
<li><strong>Create Order:</strong> Compensate with Cancel Order</li>
<li><strong>Reserve Inventory:</strong> Compensate with Release Inventory</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Patterns</a></p>

</details>

---

## 4. How does the circuit breaker pattern work?

<details>
<summary>View Answer</summary>

<h3>The Problem</h3>

<p>If a service is down, repeatedly calling it wastes resources and causes timeouts.</p>

```
Normal:  Request → Service → Success
Broken:  Request → Service (timeout) → Retry → Timeout → ...
```

<h3>Circuit Breaker Solution</h3>

<p><strong>Three states:</strong> CLOSED → OPEN → HALF_OPEN → CLOSED</p>

```
┌─────────────┐
│   CLOSED    │ ← All requests pass through
├─────────────┤
│ Failure count exceeds threshold
├─────────────┤
│    OPEN     │ ← Requests fail immediately (no call to service)
├─────────────┤
│ After timeout
├─────────────┤
│ HALF_OPEN   │ ← Allow single test request
├─────────────┤
│ If success: Back to CLOSED
│ If failure: Back to OPEN
```

<h3>Implementation (Spring Cloud)</h3>

```java
@Service
public class PaymentService {
    
    @CircuitBreaker(
        name = "paymentService",
        fallback = "paymentFallback"
    )
    public Payment processPayment(Order order) {
        return paymentGateway.charge(order);
    }
    
    // Fallback when circuit is OPEN
    public Payment paymentFallback(Order order, Exception e) {
        return new Payment("PENDING", "Service temporarily unavailable");
    }
}
```

<h3>States Explained</h3>

<table>
<tr>
<th>State</th>
<th>Behavior</th>
<th>Requests Pass?</th>
</tr>
<tr>
<td><strong>CLOSED</strong></td>
<td>Normal operation, count failures</td>
<td>✅ Yes</td>
</tr>
<tr>
<td><strong>OPEN</strong></td>
<td>Reject all requests immediately</td>
<td>❌ No (return fallback)</td>
</tr>
<tr>
<td><strong>HALF_OPEN</strong></td>
<td>Allow single test request</td>
<td>⚠️ One only</td>
</tr>
</table>

<h3>Benefits</h3>

<ul>
<li><strong>Fail fast:</strong> Don't wait for timeouts</li>
<li><strong>Resource protection:</strong> Stop wasting threads/connections</li>
<li><strong>Time to recover:</strong> Gives failing service time to recover</li>
<li><strong>Cascading failure prevention:</strong> Stop propagating errors</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker/">Learn more: Circuit Breaker Patterns</a></p>

</details>

---

## 5. What's eventual consistency and when is it acceptable?

<details>
<summary>View Answer</summary>

<h3>Eventual Consistency Definition</h3>

<p><strong>Data across services will eventually be consistent, but not immediately.</strong></p>

<p>Short-term: Different services may have different views of data
Long-term: Data will synchronize</p>

```
Time 0: Account A has $100
Time 0: Transfer $50 initiated
Time 1: Service A shows $50 (debited)
Time 1: Service B still shows old $0 (not credited yet)
Time 2: Service B syncs and shows $50 (eventually consistent)
```

<h3>Strong Consistency vs Eventual Consistency</h3>

<table>
<tr>
<th>Property</th>
<th>Strong Consistency</th>
<th>Eventual Consistency</th>
</tr>
<tr>
<td><strong>Data Sync</strong></td>
<td>Immediate</td>
<td>After some delay</td>
</tr>
<tr>
<td><strong>Implementation</strong></td>
<td>Distributed transactions, locks</td>
<td>Event-driven, message queue</td>
</tr>
<tr>
<td><strong>Performance</strong></td>
<td>Slower (must wait for sync)</td>
<td>Faster (no waiting)</td>
</tr>
<tr>
<td><strong>Availability</strong></td>
<td>Lower (may be unavailable during sync)</td>
<td>Higher (always available)</td>
</tr>
</table>

<h3>When Eventual Consistency is Acceptable</h3>

<ul>
<li><strong>✅ Social Media:</strong> Like count doesn't need to be perfect immediately</li>
<li><strong>✅ Analytics:</strong> Reports don't need real-time accuracy</li>
<li><strong>✅ Caching:</strong> Cache stale data is acceptable</li>
<li><strong>✅ Read-heavy:</strong> Most users just reading, not writing</li>
</ul>

<h3>When Strong Consistency is Required</h3>

<ul>
<li><strong>❌ Banking:</strong> Money transfer must be atomic</li>
<li><strong>❌ Inventory:</strong> Cannot oversell</li>
<li><strong>❌ Payments:</strong> Charge must debit account correctly</li>
</ul>

<h3>Strategies for Eventual Consistency</h3>

<ul>
<li><strong>Event-driven:</strong> Publish events, consumers update independently</li>
<li><strong>Message queues:</strong> Async updates via Kafka, RabbitMQ</li>
<li><strong>CQRS:</strong> Separate read/write models, reconcile later</li>
<li><strong>Compensating transactions:</strong> Fix inconsistencies with sagas</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/fundamentals/">Learn more: System Design Fundamentals</a></p>

</details>

---

## 6. Explain the database per service pattern and its challenges

<details>
<summary>View Answer</summary>

<h3>Pattern: Database Per Service</h3>

<p><strong>Each microservice has its own private database. No shared database.</strong></p>

```
UserService      OrderService      PaymentService
    ↓                  ↓                  ↓
  UserDB          OrderDB            PaymentDB
(Private)       (Private)           (Private)
```

<h3>Advantages</h3>

<ul>
<li><strong>✅ Loose coupling:</strong> Database schema changes don't affect other services</li>
<li><strong>✅ Technology choice:</strong> Each service can use optimal DB (SQL, NoSQL, etc.)</li>
<li><strong>✅ Scalability:</strong> Scale each database independently</li>
<li><strong>✅ Data privacy:</strong> Services can't accidentally access other service's data</li>
</ul>

<h3>Challenges</h3>

<h4>1. Distributed Transactions</h4>

<p><strong>Cannot use ACID transactions across databases.</strong></p>

<ul>
<li>Solution: Saga pattern (orchestration or choreography)</li>
</ul>

<h4>2. Data Consistency</h4>

<p><strong>Services may have inconsistent views of data.</strong></p>

```
User updates: UserService DB updated
But OrderService doesn't know yet
```

<ul>
<li>Solution: Event-driven architecture, eventual consistency</li>
</ul>

<h4>3. Complex Queries Across Services</h4>

<p><strong>Cannot JOIN tables across databases.</strong></p>

```sql
-- IMPOSSIBLE: User and Order in different DBs
SELECT u.name, COUNT(o.id)
FROM users u
JOIN orders o ON u.id = o.user_id;
```

<ul>
<li>Solution: CQRS (materialized views), API calls to aggregate data</li>
</ul>

<h4>4. Reporting & Analytics</h4>

<p><strong>Difficult to aggregate data from multiple services.</strong></p>

<ul>
<li>Solution: Data warehouse, event streaming, analytics DB</li>
</ul>

<h3>Database Selection Per Service</h3>

```
UserService → PostgreSQL (relational)
OrderService → MongoDB (documents)
PaymentService → Redis (key-value cache)
AnalyticsService → Elasticsearch (search)
```

<h3>Recommendation</h3>

<p><strong>Start monolithic (shared DB), migrate to database-per-service as you scale.</strong> The operational overhead is significant.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Patterns</a></p>

</details>

---

## 7. What's a consumer group and how do offsets work in Kafka?

<details>
<summary>View Answer</summary>

<h3>Kafka Topic Structure</h3>

```
Topic: order-events
├─ Partition 0: [msg0] [msg1] [msg2] ...
├─ Partition 1: [msg0] [msg1] [msg2] ...
└─ Partition 2: [msg0] [msg1] [msg2] ...
```

<p>Each partition is an immutable log of messages in order.</p>

<h3>Offset: Message Position</h3>

<p><strong>Offset = unique sequential identifier for each message within a partition.</strong></p>

```
Partition 0:
  Offset 0: {"order": "123", "amount": "$100"}
  Offset 1: {"order": "124", "amount": "$200"}
  Offset 2: {"order": "125", "amount": "$150"}
  ↑
  Consumer reads from here (offset tracking)
```

<h3>Consumer Group: Multiple Consumers</h3>

<p><strong>Multiple consumers reading same topic in parallel.</strong></p>

```
Topic: orders (3 partitions)

Consumer Group A:
  Consumer 1 → Partition 0
  Consumer 2 → Partition 1
  Consumer 3 → Partition 2
  (Each reads independently, parallel processing)

Consumer Group B:
  Consumer 1 → All 3 partitions
  (Different group, reads entire topic)
```

<h3>How It Works</h3>

```
Kafka stores: Consumer Group → Topic → Partition → Offset

Consumer Group A:
  Topic=orders, Partition=0, Offset=42 (last message read)
  Topic=orders, Partition=1, Offset=150
  Topic=orders, Partition=2, Offset=88
```

<h3>Offset Commit Strategies</h3>

<table>
<tr>
<th>Strategy</th>
<th>When</th>
<th>Semantics</th>
</tr>
<tr>
<td><strong>Auto-commit</strong></td>
<td>After polling</td>
<td>At-most-once (may lose messages)</td>
</tr>
<tr>
<td><strong>Manual commit after processing</strong></td>
<td>After business logic</td>
<td>At-least-once (may duplicate)</td>
</tr>
<tr>
<td><strong>Transactional</strong></td>
<td>With database transaction</td>
<td>Exactly-once (best)</td>
</tr>
</table>

<h3>Example: Consumer Group</h3>

```java
// Consumer 1: Process Partition 0
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord record : records) {
        // Process message
        processOrder(record.value());
    }
    consumer.commitSync();  // Update offset
}
```

<h3>Rebalancing</h3>

<p><strong>If a consumer dies, its partitions reassigned to others.</strong></p>

```
Before: C1→P0, C2→P1, C3→P2
C2 crashes
After:  C1→P0+P1, C3→P2 (rebalance)
```

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Kafka & Messaging</a></p>

</details>

---

## 8. How do you handle distributed transactions?

<details>
<summary>View Answer</summary>

<h3>The Problem: Distributed Transactions</h3>

<p>Traditional ACID transactions don't work when data is spread across multiple services/databases.</p>

```
Service A: Debit $50
Service B: Credit $50
If B fails, A's change is stuck!
```

<h3>Solutions</h3>

<h4>1. Saga Pattern (Primary Approach)</h4>

<p><strong>Orchestrate multi-step transactions with compensating transactions.</strong></p>

<p>See Question 3 for detailed explanation.</p>

<h4>2. Event Sourcing</h4>

<p><strong>Store all state changes as immutable events.</strong></p>

```
Events Log:
  - TransferInitiated(from=A, to=B, amount=50)
  - MoneyDebited(account=A, amount=50)
  - MoneyCredited(account=B, amount=50)

Replay events to rebuild state
```

<h4>3. CQRS (Command Query Responsibility Segregation)</h4>

<p><strong>Separate read and write models.</strong></p>

```
Write:  Update primary database
        Publish event
Read:   Update materialized views
        Eventually consistent
```

<h4>4. Two-Phase Commit (❌ Avoid)</h4>

<p><strong>Coordinator says "prepare to commit", then "commit".</strong></p>

<p>Theoretical but rarely used in practice:</p>
<ul>
<li>❌ Blocking (locks held during coordination)</li>
<li>❌ Slow (requires multiple round trips)</li>
<li>❌ Coordinator failure = deadlock</li>
</ul>

<h3>Practical Approach</h3>

<table>
<tr>
<th>Scenario</th>
<th>Solution</th>
</tr>
<tr>
<td>Money transfer</td>
<td>Saga with compensating transactions</td>
</tr>
<tr>
<td>Inventory + Order</td>
<td>Event-driven with eventual consistency</td>
</tr>
<tr>
<td>Audit trail needed</td>
<td>Event Sourcing</td>
</tr>
<tr>
<td>Complex queries needed</td>
<td>CQRS with materialized views</td>
</tr>
</table>

<h3>Guarantees</h3>

<ul>
<li><strong>At-most-once:</strong> Messages may be lost</li>
<li><strong>At-least-once:</strong> Messages may be duplicated</li>
<li><strong>Exactly-once:</strong> Perfect, but expensive (use for critical operations)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Distributed Patterns</a></p>

</details>

---

## 9. What is event-driven microservice architecture?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Services communicate by publishing <strong>events</strong> (facts that already happened, e.g. <code>OrderPlaced</code>) to a broker such as Kafka or RabbitMQ. Interested services subscribe and react on their own, so the publisher does not know or wait for its consumers.</p>

<h3>Why / Context</h3>
<ul>
<li><strong>Loose coupling:</strong> add a new consumer without changing the producer.</li>
<li><strong>Resilience:</strong> the broker buffers events while a consumer is down.</li>
<li><strong>Scalability:</strong> consumers scale independently (consumer groups).</li>
<li><strong>Cost:</strong> eventual consistency, harder debugging, duplicate and out-of-order delivery.</li>
</ul>

<h3>Key Building Blocks</h3>
<ul>
<li><strong>Event notification:</strong> small event, consumers call back for details.</li>
<li><strong>Event-carried state transfer:</strong> event holds the data, consumers keep local copies.</li>
<li><strong>Event sourcing + CQRS:</strong> store events as the source of truth, build read models from them.</li>
<li><strong>Transactional outbox:</strong> write the event to an outbox table in the same DB transaction as the state change, then relay it. This avoids the dual-write problem (DB commit succeeds, publish fails).</li>
<li><strong>Idempotent consumers:</strong> delivery is usually at-least-once, so dedupe by event ID.</li>
</ul>

<h3>Example</h3>

```java
// Producer: Order Service
kafkaTemplate.send("orders.placed", order.getId(), new OrderPlaced(order.getId(), order.getTotal()));

// Consumer: Inventory Service (independent deployment)
@KafkaListener(topics = "orders.placed", groupId = "inventory")
public void on(OrderPlaced e) { inventory.reserve(e.orderId()); }
```

<h3>When to Use</h3>
<p>Use it for workflows spanning many services, fan-out notifications and audit trails. Prefer synchronous calls when the caller needs an immediate answer. Follow-up: how do you handle failures? Use retries, a dead letter topic and a saga for compensation.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 10. What is a bounded context in microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>A bounded context is a Domain-Driven Design concept: an explicit boundary inside which a domain model and its vocabulary have one consistent meaning. The same word can mean different things in different contexts.</p>

<h3>Why / Context</h3>
<ul>
<li>"Customer" in Sales (leads, discounts) is not "Customer" in Billing (invoices, tax ID) or Shipping (address).</li>
<li>Forcing one shared model across the company creates a big ball of mud and tight coupling.</li>
<li>Each bounded context gets its own model, and often its own database and team. That makes it a natural microservice boundary.</li>
</ul>

<h3>How Contexts Relate</h3>
<table>
<tr><th>Pattern</th><th>Meaning</th></tr>
<tr><td>Anti-corruption layer</td><td>Translates another context's model so it does not leak into yours</td></tr>
<tr><td>Published language / events</td><td>Shared, versioned contract between contexts</td></tr>
<tr><td>Shared kernel</td><td>Small model owned jointly (use sparingly, it couples teams)</td></tr>
<tr><td>Context map</td><td>Diagram of contexts and their relationships</td></tr>
</table>

<h3>Example</h3>
<p>Ordering owns <code>Order</code> and <code>OrderLine</code>. Payment holds only <code>orderId</code> and <code>amount</code>. They exchange <code>OrderPlaced</code> and <code>PaymentCompleted</code> events and never share tables.</p>

<h3>Gotchas</h3>
<ul>
<li>A bounded context is not always one microservice: one context can contain a few services, but one service should not span contexts.</li>
<li>Find boundaries with event storming and by watching where terminology changes.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 11. How do you define service boundaries?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Split by <strong>business capability / bounded context</strong>, not by technical layer. A good boundary gives high cohesion inside the service, low coupling between services, and lets one team own and deploy it independently.</p>

<h3>Heuristics</h3>
<ul>
<li><strong>Business capability:</strong> Orders, Payments, Catalog, not "DAO service" or "UI service".</li>
<li><strong>Data ownership:</strong> each service owns its data. If two services constantly join the same tables, the boundary is wrong.</li>
<li><strong>Change together, stay together:</strong> features that always change in the same release belong in one service.</li>
<li><strong>Team ownership:</strong> one team per service (Conway's law).</li>
<li><strong>Different scaling or availability needs</strong> can justify separating a component.</li>
<li><strong>Transaction boundaries:</strong> if an operation needs a strong ACID guarantee, keep it inside one service.</li>
</ul>

<h3>Warning Signs of Bad Boundaries</h3>
<ul>
<li>Chatty synchronous calls between two services for every request.</li>
<li>Lock-step deployments, where service A cannot release without B.</li>
<li>A shared database or shared library holding domain logic.</li>
<li>Distributed monolith: all the costs of microservices with none of the independence.</li>
</ul>

<h3>When to Use</h3>
<p>Start with a well-structured modular monolith if the domain is unclear, then extract services along proven seams (strangler fig approach). Splitting too early is the common mistake, and wrong boundaries are costly to fix.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 12. How do you handle configuration in microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Externalize configuration from the build artifact (12-factor: config in the environment) so one immutable image runs in every environment. Use environment variables, Kubernetes ConfigMaps/Secrets, or a central config server such as Spring Cloud Config.</p>

<h3>Options</h3>
<table>
<tr><th>Option</th><th>Strength</th><th>Weakness</th></tr>
<tr><td>Env variables / ConfigMap</td><td>Simple, platform-native</td><td>Needs restart or rollout to change</td></tr>
<tr><td>Spring Cloud Config (Git-backed)</td><td>Central, versioned, audited, per-profile</td><td>Extra component to run and secure</td></tr>
<tr><td>Vault / cloud secret managers</td><td>Secrets, rotation, access control</td><td>Operational complexity</td></tr>
</table>

<h3>Example</h3>

```yaml
# client service application.yml
spring:
  application:
    name: order-service
  config:
    import: "optional:configserver:http://config-server:8888"
  profiles:
    active: prod
```

<h3>Gotchas and Follow-ups</h3>
<ul>
<li>Never store secrets in Git in plain text. Use encryption or a secrets manager.</li>
<li>Runtime refresh: <code>@RefreshScope</code> with <code>/actuator/refresh</code>, or Spring Cloud Bus to broadcast to all instances.</li>
<li>The config server is a dependency: make it highly available and let clients fall back to local defaults or cached values.</li>
<li>Feature flags are a separate concern from static config.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/spring-cloud/">Learn more: Spring Cloud & Microservices</a></p>

</details>

---

## 13. What is service discovery and why do you need it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Service discovery lets services find each other's current network locations at runtime through a <strong>service registry</strong>, instead of using fixed hosts and ports. It is needed because instances are dynamic: autoscaling, crashes and redeployments change IPs constantly.</p>

<h3>How It Works</h3>
<ul>
<li><strong>Register:</strong> an instance registers its name and address on startup.</li>
<li><strong>Heartbeat / health check:</strong> the registry removes instances that stop responding.</li>
<li><strong>Lookup:</strong> callers resolve a logical name (<code>payment-service</code>) to healthy instances and load balance.</li>
</ul>

<h3>Client-Side vs Server-Side</h3>
<table>
<tr><th>Style</th><th>Who picks the instance</th><th>Example</th></tr>
<tr><td>Client-side</td><td>Caller queries the registry and balances</td><td>Eureka + Spring Cloud LoadBalancer</td></tr>
<tr><td>Server-side</td><td>A router or proxy does it</td><td>Kubernetes Service (DNS + kube-proxy), cloud load balancers</td></tr>
</table>

<h3>Tools</h3>
<ul>
<li><strong>Eureka:</strong> Netflix OSS, AP-leaning, client-side, common in Spring Cloud.</li>
<li><strong>Consul:</strong> registry + health checks + KV store, DNS and HTTP APIs.</li>
<li><strong>Kubernetes:</strong> built in via Services and DNS. No separate registry is needed on K8s.</li>
</ul>

<h3>Gotchas</h3>
<p>The registry must be highly available. Clients should cache the registry data so a registry outage does not stop traffic. Stale entries can send calls to dead instances, so combine discovery with retries and circuit breakers.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/service-discovery/">Learn more: Service Discovery</a></p>

</details>

---

## 14. How does service discovery work compared with hardcoded service URLs, including in containerized environments?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Hardcoded URLs bind a caller to one address, which breaks as soon as instances scale, move or restart. With discovery the caller uses a stable logical name and the platform resolves it to healthy instances at call time.</p>

<h3>Comparison</h3>
<table>
<tr><th>Aspect</th><th>Hardcoded URL</th><th>Service discovery</th></tr>
<tr><td>Scaling</td><td>Manual config per new instance</td><td>Automatic registration</td></tr>
<tr><td>Failure handling</td><td>Calls keep hitting a dead host</td><td>Unhealthy instances removed</td></tr>
<tr><td>Load balancing</td><td>External, manual</td><td>Built in</td></tr>
<tr><td>Environments</td><td>URL differs per env</td><td>Same logical name everywhere</td></tr>
<tr><td>Complexity</td><td>None</td><td>Registry or platform needed</td></tr>
</table>

<h3>In Containers and Kubernetes</h3>
<p>Container IPs are ephemeral, so hardcoding is unworkable. Kubernetes gives each Service a stable virtual IP and DNS name that fronts the pod set, selected by labels and filtered by readiness probes.</p>

```yaml
# Caller config: stable DNS name, not an IP
payment:
  base-url: http://payment-service.shop.svc.cluster.local:8080
```

<h3>Gotchas</h3>
<ul>
<li>Only <strong>ready</strong> pods receive traffic, so define a proper readiness probe.</li>
<li>Client DNS or connection caching can keep stale endpoints. Use sane TTLs and connection pool eviction.</li>
<li>A hardcoded URL pointing at a stable DNS name or gateway is fine. The problem is hardcoding instance addresses.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/service-discovery/">Learn more: Service Discovery</a></p>

</details>

---

## 15. How do you use microservices patterns in system design?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Pick patterns that answer a specific problem in the design, rather than adding all of them. Name the problem, the pattern, and its trade-off.</p>

<h3>Pattern Map</h3>
<table>
<tr><th>Problem</th><th>Pattern</th><th>Trade-off</th></tr>
<tr><td>Many clients, many services</td><td>API Gateway (routing, auth, rate limit, aggregation)</td><td>Extra hop, can become a bottleneck or SPOF</td></tr>
<tr><td>Cross-cutting network concerns (mTLS, retries, telemetry)</td><td>Service mesh (sidecar proxies)</td><td>Operational complexity, latency overhead</td></tr>
<tr><td>Cross-service data consistency</td><td>Saga + transactional outbox</td><td>Eventual consistency, compensation logic</td></tr>
<tr><td>Audit trail, replay, temporal queries</td><td>Event sourcing</td><td>Complex queries, event schema evolution</td></tr>
<tr><td>Read and write loads differ</td><td>CQRS (separate read models)</td><td>Stale reads, more moving parts</td></tr>
<tr><td>Cascading failures</td><td>Circuit breaker, bulkhead, timeouts</td><td>Needs tuning and fallbacks</td></tr>
<tr><td>Migrating from a monolith</td><td>Strangler fig</td><td>Two systems run side by side for a while</td></tr>
</table>

<h3>How to Present It</h3>
<ul>
<li>Start from requirements (scale, latency, consistency), then add patterns.</li>
<li>Each service owns its data (database per service). Services talk through sync APIs for queries and events for state changes.</li>
<li>Put observability in from day one: correlation IDs, metrics, centralized logs.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Microservices Principles</a></p>

</details>

---

## 16. How do you maintain unique identifiers across pods in microservices?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Do not rely on a single database auto-increment, because many instances create IDs concurrently. Use a distributed ID strategy that needs no coordination on each call: UUID, Snowflake-style IDs, or ID ranges.</p>

<h3>Options</h3>
<table>
<tr><th>Approach</th><th>Pros</th><th>Cons</th></tr>
<tr><td>UUID v4</td><td>No coordination, trivially unique</td><td>128-bit, random order hurts B-tree index locality</td></tr>
<tr><td>UUID v7 / ULID</td><td>Time-ordered, index friendly</td><td>Larger than a long</td></tr>
<tr><td>Snowflake (64-bit)</td><td>Compact, sortable, high throughput</td><td>Needs unique worker IDs, depends on clock</td></tr>
<tr><td>DB sequence / ticket server</td><td>Simple, ordered</td><td>Bottleneck and SPOF, network hop</td></tr>
<tr><td>Segment / range allocation</td><td>Few DB calls</td><td>Gaps, not strictly ordered across nodes</td></tr>
</table>

<h3>Snowflake Layout</h3>
<p>Typically about 41 bits of timestamp, 10 bits of worker/machine ID and 12 bits of per-millisecond sequence (roughly 4096 IDs per ms per worker). Worker IDs can come from a StatefulSet ordinal, a registry like ZooKeeper or etcd, or config.</p>

```java
UUID id = UUID.randomUUID(); // v4, fine for most services
```

<h3>Gotchas</h3>
<ul>
<li>Clock moving backwards can create duplicates in Snowflake. Detect it and wait or fail.</li>
<li>Two pods with the same worker ID will collide.</li>
<li>Do not use sequential public IDs if enumeration is a security concern.</li>
<li>Time-ordered IDs make inserts faster than random UUIDs on clustered indexes.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/system-design/building-blocks/">Learn more: System Design Building Blocks</a></p>

</details>

---

## 17. What challenges arise in microservices testing?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Services depend on each other, the network and shared infrastructure, so end-to-end tests become slow, flaky and hard to set up. The strategy is to push most confidence into fast, isolated tests and use contract tests to verify the integration points.</p>

<h3>Challenges</h3>
<ul>
<li><strong>Service dependencies:</strong> running dozens of services for one test is costly.</li>
<li><strong>Flaky E2E tests:</strong> network, timing, shared test data.</li>
<li><strong>Integration drift:</strong> a provider changes its API and breaks consumers unnoticed.</li>
<li><strong>Async flows:</strong> events and eventual consistency need polling or awaiting.</li>
<li><strong>Environment parity</strong> and test data management.</li>
</ul>

<h3>Test Layers</h3>
<table>
<tr><th>Layer</th><th>Scope</th><th>Tools</th></tr>
<tr><td>Unit</td><td>Class or domain logic</td><td>JUnit 5, Mockito</td></tr>
<tr><td>Component / integration</td><td>One service with real DB or broker</td><td>Spring Boot Test, Testcontainers, WireMock</td></tr>
<tr><td>Contract</td><td>Consumer and provider agree on API or event schema</td><td>Pact, Spring Cloud Contract</td></tr>
<tr><td>E2E</td><td>A few critical user journeys</td><td>Deployed staging environment</td></tr>
</table>

<h3>Contract Testing</h3>
<p>The consumer publishes the requests and responses it expects. The provider build verifies them, so breaking changes fail in CI without deploying both services together.</p>

<h3>Tips</h3>
<ul>
<li>Follow the test pyramid: many unit tests, some component and contract tests, few E2E tests.</li>
<li>Use Testcontainers for real databases and Kafka instead of mocks that drift.</li>
<li>In production, rely on canary releases, feature flags and observability as a safety net.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/foundations/testing/">Learn more: Testing / QA</a></p>

</details>

---

## 18. How do you monitor and trace requests across services?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use the three pillars together: <strong>metrics</strong> (what is wrong), <strong>logs</strong> (details) and <strong>distributed traces</strong> (where in the call chain). A trace ID is generated at the edge and propagated through every call so one request can be followed across services.</p>

<h3>How Tracing Works</h3>
<ul>
<li>A <strong>trace</strong> is the whole request. A <strong>span</strong> is one unit of work (an HTTP call, a DB query).</li>
<li>The trace ID travels in headers, such as W3C <code>traceparent</code>, and in Kafka message headers for async hops.</li>
<li>Put the trace ID in every log line (MDC) so you can pivot from a trace to the logs.</li>
</ul>

<h3>Typical Stack</h3>
<table>
<tr><th>Concern</th><th>Tooling</th></tr>
<tr><td>Instrumentation</td><td>Micrometer Tracing (Spring Boot 3), OpenTelemetry</td></tr>
<tr><td>Trace backend</td><td>Zipkin, Jaeger, Tempo</td></tr>
<tr><td>Metrics</td><td>Micrometer with Prometheus and Grafana</td></tr>
<tr><td>Logs</td><td>ELK or Loki with centralized, structured JSON logs</td></tr>
<tr><td>Health</td><td>Actuator health, liveness and readiness probes</td></tr>
</table>

<h3>What to Watch</h3>
<ul>
<li>RED metrics per service: Rate, Errors, Duration (use p95 and p99, not averages).</li>
<li>Saturation: thread pools, connection pools, queue depth, consumer lag.</li>
<li>Alert on SLOs and symptoms, not on every metric.</li>
</ul>

<h3>Gotchas</h3>
<p>Sampling reduces cost but can drop the one trace you need, so consider tail-based sampling. Context propagation breaks across thread pools and async boundaries unless instrumented.</p>

<p>📚 <a href="/anandu-interview-playbook/devops/observability/">Learn more: Observability</a></p>

</details>

---

## 19. How do you wrap legacy SOAP services with a REST API and aggregate or transform their data?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Build a thin <strong>adapter (facade) service</strong>, also called an anti-corruption layer, that exposes a clean REST/JSON API. Internally it calls the SOAP service through a generated client and maps the SOAP model to your own domain DTOs, so legacy structures do not leak out.</p>

<h3>Steps</h3>
<ul>
<li>Generate a client from the WSDL (JAX-WS <code>wsimport</code>, Apache CXF, or Spring Web Services <code>WebServiceTemplate</code>).</li>
<li>Define REST resources and DTOs around consumer needs, not the SOAP schema.</li>
<li>Map with MapStruct or manual mappers (dates, enums, nulls, SOAP faults to HTTP status codes).</li>
<li>Aggregate: call several SOAP operations or other services (in parallel where independent) and compose one response.</li>
<li>Add resilience: timeouts, retries for idempotent calls, circuit breaker, caching of slow-changing data.</li>
</ul>

<h3>Example</h3>

```java
@GetMapping("/customers/{id}")
public CustomerDto get(@PathVariable String id) {
    GetCustomerResponse r = soapClient.getCustomer(new GetCustomerRequest(id)); // generated client
    return mapper.toDto(r.getCustomer());   // legacy model stays behind the adapter
}
```

<h3>Gotchas</h3>
<ul>
<li>SOAP calls are often slow: set short timeouts and use a bulkhead so they cannot exhaust your threads.</li>
<li>Translate SOAP faults to meaningful HTTP errors (4xx vs 502/503).</li>
<li>Handle WS-Security and large XML payloads (use streaming if needed).</li>
<li>Use the strangler fig approach to retire the legacy service gradually.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/spring-web/web-services/">Learn more: Web Services (SOAP vs REST)</a></p>

</details>

---

## 20. What is a Two-Phase Commit (2PC) in distributed systems?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>2PC is an atomic commit protocol where a <strong>coordinator</strong> makes all participants (databases or services) either commit or abort a distributed transaction together.</p>

<h3>The Two Phases</h3>
<ul>
<li><strong>Phase 1, prepare (voting):</strong> the coordinator asks each participant to prepare. Each one locks resources, writes to its log and votes yes or no.</li>
<li><strong>Phase 2, commit or abort:</strong> if all voted yes, the coordinator tells everyone to commit. Any no (or timeout) means abort for everyone.</li>
</ul>

<h3>Problems</h3>
<ul>
<li><strong>Blocking:</strong> if the coordinator crashes after prepare, participants hold locks until it recovers.</li>
<li><strong>Latency and low throughput:</strong> locks are held across network round trips.</li>
<li><strong>Availability:</strong> every participant must be up (CAP trade-off favors consistency).</li>
<li>Not supported by many NoSQL stores, message brokers and REST services.</li>
</ul>

<h3>2PC vs Saga</h3>
<table>
<tr><th>Aspect</th><th>2PC</th><th>Saga</th></tr>
<tr><td>Consistency</td><td>Strong (atomic)</td><td>Eventual</td></tr>
<tr><td>Locking</td><td>Held during the protocol</td><td>No cross-service locks</td></tr>
<tr><td>Failure handling</td><td>Rollback by protocol</td><td>Compensating transactions</td></tr>
<tr><td>Fit</td><td>Few DBs in one trust domain (XA/JTA)</td><td>Microservices</td></tr>
</table>

<h3>When to Use</h3>
<p>Occasionally for a small number of XA-capable resources in one system. In microservices prefer sagas and the outbox pattern. A variant, 3PC, reduces blocking but is rarely used in practice.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles/">Learn more: Distributed Patterns</a></p>

</details>

---

**See also:** [Microservices Principles](../../microservices-distributed/principles.md), [Messaging & Kafka](../../microservices-distributed/messaging.md), [Service Discovery](../../microservices-distributed/service-discovery.md), [Interview Preparation](../index.md)
