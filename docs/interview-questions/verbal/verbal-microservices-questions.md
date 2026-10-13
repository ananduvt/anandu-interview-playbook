# Microservices & Distributed Systems Interview Questions

Architecture patterns, scalability, resilience, messaging, and distributed computing principles.

---

## Question Categories

### Microservices Architecture
- Monolith vs microservices
- Service boundaries, domain-driven design
- Decomposition strategies
- Deployment, versioning

### Service Communication
- Synchronous: REST, gRPC
- Asynchronous: messaging, event-driven
- API contracts, versioning
- Service discovery, load balancing

### Data Management
- Database per service pattern
- Distributed transactions, saga pattern
- CQRS, event sourcing
- Data consistency, eventual consistency

### Resilience & Fault Tolerance
- Circuit breaker, retry, timeout
- Bulkhead pattern, rate limiting
- Health checks, graceful degradation
- Monitoring, alerting

### Messaging & Event-Driven
- Message queues (RabbitMQ, Kafka)
- Event streams, topics, partitions
- Message ordering, delivery guarantees
- Consumer groups, offsets

### Distributed Tracing & Monitoring
- Correlation IDs, distributed logging
- APM tools, metrics
- Request tracing, latency analysis

---

## Interview Questions (With Answers)

<details>
<summary>What is an offset in Kafka?</summary>

**Answer:** An offset is a unique sequential identifier (integer) for each message within a Kafka partition. Consumers track their offset to know which messages they've processed. Offsets enable exactly-once processing semantics and allow consumers to resume from where they left off after failures.

📚 [Learn more: Messaging (Kafka)](../../microservices-distributed/messaging.md)
</details>

<details>
<summary>How do multiple consumers read from a topic with only one producer?</summary>

**Answer:** Multiple consumers can belong to the same consumer group, each reading different partitions. Kafka distributes partitions across consumers in a group. Each partition goes to one consumer, enabling parallel processing. Different consumer groups can read the same topic independently from their own offsets.

📚 [Learn more: Messaging (Kafka)](../../microservices-distributed/messaging.md)
</details>

<details>
<summary>How do you ensure that a message has been delivered successfully in Kafka?</summary>

**Answer:** Use producer acknowledgments (`acks=all`), consumer commit modes (auto-commit vs manual), and idempotent producers. For exactly-once semantics: use transactions or idempotent producer + consumer with offset commits. Monitor lag, replication factor, and use dead-letter topics for failed messages.

📚 [Learn more: Messaging (Kafka)](../../microservices-distributed/messaging.md)
</details>

---

## Sample Topics (Questions to be added)

- When should you use microservices vs monolith?
- How do you define service boundaries?
- What's the difference between synchronous and asynchronous communication?
- Explain the saga pattern for distributed transactions
- How does the circuit breaker pattern work?
- What's eventual consistency and when is it acceptable?
- Explain the database per service pattern and its challenges
- How does message ordering work in Kafka?
- What's a consumer group and how do offsets work?
- How do you handle distributed transactions?
- Explain CQRS and event sourcing
- What's service discovery and why do you need it?
- How do you monitor and trace requests across services?
- Explain the bulkhead pattern

---

[Content to be added progressively]

**See also:** [Microservices Principles](../../microservices-distributed/principles.md), [Service Discovery](../../microservices-distributed/service-discovery.md), [Messaging](../../microservices-distributed/messaging.md), [Interview Questions Overview](../index.md)
