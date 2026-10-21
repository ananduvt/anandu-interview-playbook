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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles.md">Learn more: Microservices Principles</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging.md">Learn more: Messaging & Events</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles.md">Learn more: Microservices Patterns</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/spring-web/spring-boot-circuit-breaker.md">Learn more: Circuit Breaker Patterns</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/system-design/fundamentals.md">Learn more: System Design Fundamentals</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles.md">Learn more: Microservices Patterns</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging.md">Learn more: Kafka & Messaging</a></p>

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

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/principles.md">Learn more: Distributed Patterns</a></p>

</details>

---

## Additional Topics

- How do you define service boundaries?
- What's service discovery and why do you need it?
- How do you monitor and trace requests across services?
- Explain the bulkhead pattern
- When should you use RabbitMQ vs Kafka?
- How do you handle network failures in distributed systems?

---

**See also:** [Microservices Principles](../../microservices-distributed/principles.md), [Messaging & Kafka](../../microservices-distributed/messaging.md), [Service Discovery](../../microservices-distributed/service-discovery.md), [Interview Preparation](../index.md)
