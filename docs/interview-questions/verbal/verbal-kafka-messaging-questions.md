# Kafka & Messaging Interview Questions

Kafka architecture, consumer groups, reprocessing, delivery guarantees, error handling and choosing a message broker.

---

## 1. Explain Kafka architecture and how it works internally

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Kafka is a distributed, replicated, append-only commit log. Producers write records to <strong>topics</strong>, topics are split into <strong>partitions</strong> spread across <strong>brokers</strong>, and consumers in <strong>consumer groups</strong> read the partitions at their own pace using <strong>offsets</strong>.</p>

<h3>Core Concepts</h3>
<table>
<tr><th>Concept</th><th>Meaning</th></tr>
<tr><td>Broker</td><td>A Kafka server that stores partitions and serves clients</td></tr>
<tr><td>Topic</td><td>Named stream of records</td></tr>
<tr><td>Partition</td><td>Ordered, immutable log. The unit of parallelism and ordering</td></tr>
<tr><td>Offset</td><td>Position of a record within a partition</td></tr>
<tr><td>Leader / follower</td><td>Each partition has one leader for reads and writes and followers that replicate</td></tr>
<tr><td>ISR</td><td>In-sync replicas that are caught up with the leader</td></tr>
<tr><td>Consumer group</td><td>Consumers sharing a topic, where each partition goes to one member of the group</td></tr>
<tr><td>Controller</td><td>Manages metadata and leader election (KRaft in modern Kafka, ZooKeeper in older versions)</td></tr>
</table>

<h3>How It Works Internally</h3>
<ul>
<li>The producer picks a partition by key hash (same key goes to the same partition) or round-robin or sticky for null keys.</li>
<li>The leader appends to the partition log segment on disk (sequential I/O, page cache, zero-copy reads) and followers replicate.</li>
<li>With <code>acks=all</code> the write is acknowledged once the in-sync replicas (at least <code>min.insync.replicas</code>) have it.</li>
<li>Consumers pull, track offsets, and commit them to the internal <code>__consumer_offsets</code> topic.</li>
<li>Records are kept by retention (time or size) or compacted by key, and are not deleted when consumed.</li>
</ul>

<h3>Why It Is Fast</h3>
<p>Sequential disk writes, OS page cache, batching and compression, zero-copy transfer and partition-level parallelism.</p>

<h3>When to Use</h3>
<p>Event streaming, log and metrics pipelines, event-driven microservices and replayable event history. When describing your own project, say which topics, keys and partition counts you chose and why.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 2. How do you know how many consumers are consuming a Kafka topic?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Consumers are tracked per <strong>consumer group</strong>. Use <code>kafka-consumer-groups.sh</code> to list the groups, see their active members, partition assignment, committed offsets and lag.</p>

<h3>Commands</h3>

```bash
# list all groups
kafka-consumer-groups.sh --bootstrap-server broker:9092 --list

# members, assignment and lag for one group
kafka-consumer-groups.sh --bootstrap-server broker:9092 --describe --group orders-service
# columns: TOPIC PARTITION CURRENT-OFFSET LOG-END-OFFSET LAG CONSUMER-ID HOST CLIENT-ID

# active members only
kafka-consumer-groups.sh --bootstrap-server broker:9092 --describe --group orders-service --members --verbose
```

<h3>Things to Know</h3>
<ul>
<li>A topic can have many groups, and each group gets every record independently (pub-sub between groups).</li>
<li>Inside a group, active consumers cannot usefully exceed the partition count. Extra consumers sit idle.</li>
<li><strong>Lag</strong> = log-end offset minus committed offset. Growing lag means consumers cannot keep up.</li>
<li>Programmatic access: <code>AdminClient.describeConsumerGroups()</code> and <code>listConsumerGroupOffsets()</code>.</li>
<li>Monitoring: Kafka UI tools, Prometheus exporters (for example Burrow or kafka-exporter) and alerts on lag.</li>
</ul>

<h3>Gotcha</h3>
<p>A group with no live members shows state <code>Empty</code>. Its offsets are kept for the offset retention period (7 days by default in recent versions), after which they may be deleted.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 3. How do you reprocess messages in Kafka?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Kafka keeps records until retention expires, so reprocessing means moving the consumer group's offset back. Either reset the group's committed offsets, or seek in code, or replay into a new consumer group.</p>

<h3>Strategies</h3>
<table>
<tr><th>Strategy</th><th>How</th></tr>
<tr><td>Reset group offsets (CLI)</td><td>Stop the group, then run <code>--reset-offsets</code> with <code>--to-earliest</code>, <code>--to-datetime</code>, <code>--shift-by</code> or <code>--to-offset</code>, and add <code>--execute</code></td></tr>
<tr><td>Seek in code</td><td><code>consumer.seek()</code>, <code>seekToBeginning()</code> or <code>offsetsForTimes()</code></td></tr>
<tr><td>New consumer group</td><td>New <code>group.id</code> with <code>auto.offset.reset=earliest</code> replays everything</td></tr>
<tr><td>Replay or retry topic</td><td>Republish failed records from a dead letter topic</td></tr>
</table>

```bash
kafka-consumer-groups.sh --bootstrap-server broker:9092 --group orders-service \
  --topic orders --reset-offsets --to-datetime 2026-01-15T10:00:00.000 --execute
```

```java
// seek in code
consumer.poll(Duration.ZERO);
consumer.seekToBeginning(consumer.assignment());
```

<h3>Gotchas</h3>
<ul>
<li>The group must be inactive to reset offsets. Without <code>--execute</code> it is only a dry run.</li>
<li>Reprocessing causes duplicates in downstream systems, so consumers must be <strong>idempotent</strong>.</li>
<li>You can only replay what retention still holds. Compacted topics keep only the latest value per key.</li>
<li>Replaying at full speed can overload downstream services, so throttle it.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 4. How do you ensure all messages are processed correctly in Kafka?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Combine a durable producer, a consumer that commits offsets only after processing succeeds, and idempotent processing. That gives effectively-once results on top of at-least-once delivery.</p>

<h3>Producer Side (no loss)</h3>
<ul>
<li><code>acks=all</code>, <code>min.insync.replicas=2</code>, replication factor 3.</li>
<li><code>enable.idempotence=true</code> removes duplicates caused by producer retries (the default in Kafka 3.0+ clients).</li>
<li>Handle send failures in callbacks, not fire-and-forget.</li>
</ul>

<h3>Consumer Side (no skipped records)</h3>
<ul>
<li>Disable auto commit and commit after successful processing.</li>
<li>Process, persist the result, then commit. If it crashes before the commit, the record is redelivered (at-least-once).</li>
<li>Make handlers idempotent: unique business key, upsert, or a processed-events table.</li>
</ul>

```java
@KafkaListener(topics = "payments", groupId = "ledger")
public void handle(ConsumerRecord<String, Payment> rec, Acknowledgment ack) {
    if (!processed.exists(rec.value().id())) {   // idempotency check
        ledger.apply(rec.value());
    }
    ack.acknowledge();                           // commit after success (AckMode.MANUAL)
}
```

<h3>Exactly-Once</h3>
<p>Kafka transactions (transactional producer plus <code>isolation.level=read_committed</code>) give exactly-once for Kafka-to-Kafka read-process-write. Side effects in external systems still need idempotency.</p>

<h3>Gotchas</h3>
<ul>
<li>Committing before processing means at-most-once, so records are lost on a crash.</li>
<li>Slow processing exceeding <code>max.poll.interval.ms</code> triggers a rebalance and duplicates.</li>
<li>Route poison messages to a dead letter topic so one bad record does not block a partition.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 5. What happens if a Kafka topic is missed, and how do you handle it?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>It depends on what "missed" means. The usual cases are a consumer that was down, a topic that did not exist or was deleted, or records that failed processing. Kafka's durable log and offsets let you recover in most of them.</p>

<h3>Scenarios</h3>
<table>
<tr><th>Situation</th><th>What happens / fix</th></tr>
<tr><td>Consumer was down</td><td>Records stay in the log. It resumes from its committed offset on restart (if within retention)</td></tr>
<tr><td>New group with no offset</td><td>Follows <code>auto.offset.reset</code>: <code>latest</code> skips old data, <code>earliest</code> reads from the start</td></tr>
<tr><td>Offset expired or out of range</td><td>Retention removed the data. The <code>auto.offset.reset</code> policy applies, so the data in the gap is lost to this group</td></tr>
<tr><td>Subscribed topic missing</td><td>Auto-creation or a clear startup error, depending on config. Create topics explicitly in IaC</td></tr>
<tr><td>Processing failed</td><td>Retry, then send to a dead letter topic (DLT) for inspection and replay</td></tr>
<tr><td>Topic deleted</td><td>Restore from a mirror (MirrorMaker 2 or cluster linking) or from the source system</td></tr>
</table>

<h3>Handling Pattern</h3>
<ul>
<li>Retries with backoff, then publish to <code>orders.DLT</code> with the original headers plus error info.</li>
<li>Alert on lag and DLT volume, then fix the cause and replay from the DLT.</li>
<li>Set retention longer than your worst-case outage and recovery time.</li>
</ul>

```java
// Spring Kafka: retry then dead-letter
new DefaultErrorHandler(new DeadLetterPublishingRecoverer(kafkaTemplate),
        new FixedBackOff(1000L, 3));
```

<h3>Gotcha</h3>
<p>Set <code>auto.offset.reset</code> deliberately: <code>latest</code> silently skips history for a new group, and <code>none</code> fails fast.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 6. How do you avoid failures in Kafka?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>You cannot prevent failures, so you design for them at three levels: cluster (replication), producer and consumer (safe configuration), and application (error handling and monitoring).</p>

<h3>Cluster Level</h3>
<ul>
<li>At least 3 brokers, replication factor 3, <code>min.insync.replicas=2</code>, spread across racks or zones.</li>
<li><code>unclean.leader.election.enable=false</code> so an out-of-sync replica never becomes leader.</li>
<li>Sensible retention and enough disk, with disk and broker health monitored.</li>
</ul>

<h3>Producer Level</h3>
<ul>
<li><code>acks=all</code>, idempotent producer, retries with <code>delivery.timeout.ms</code>.</li>
<li>Handle callbacks and failures, and use the outbox pattern when the send must be atomic with a DB change.</li>
</ul>

<h3>Consumer Level</h3>
<ul>
<li>Commit after processing, idempotent handlers, tuned <code>max.poll.records</code> and <code>max.poll.interval.ms</code>.</li>
<li>Retry topics and a dead letter topic, plus static membership or cooperative rebalancing to reduce rebalance churn.</li>
<li>Schema registry with compatibility rules, to avoid deserialization failures.</li>
</ul>

<h3>Operations</h3>
<ul>
<li>Alert on consumer lag, under-replicated partitions, offline partitions and DLT size.</li>
<li>Cross-cluster replication for disaster recovery, and tested runbooks.</li>
</ul>

<h3>Gotchas</h3>
<p>Replication protects against broker loss, not against bad data or accidental deletion. Too many partitions slow down failover and recovery.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 7. How do you implement asynchronous processing with messaging systems like Kafka or MQ?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Put a message broker between the caller and the worker. The caller publishes a message and returns immediately, and one or more consumers process it later. This decouples the services in time, absorbs spikes and lets you scale workers independently.</p>

<h3>Patterns</h3>
<table>
<tr><th>Pattern</th><th>Description</th><th>Typical broker</th></tr>
<tr><td>Work queue</td><td>Many workers share the load, each message handled once</td><td>RabbitMQ, SQS, Kafka consumer group</td></tr>
<tr><td>Publish-subscribe</td><td>Every subscriber gets each message</td><td>Kafka topics, RabbitMQ fanout/topic exchange</td></tr>
<tr><td>Request-reply</td><td>Async request with a reply queue plus correlation ID</td><td>JMS, RabbitMQ</td></tr>
<tr><td>Event streaming</td><td>Replayable log of events</td><td>Kafka</td></tr>
</table>

<h3>Example (HTTP 202 plus a worker)</h3>

```java
@PostMapping("/reports")
public ResponseEntity<Void> create(@RequestBody ReportRequest r) {
    String id = reports.createPending(r);
    kafkaTemplate.send("report-requests", id, r);       // publish, do not wait for the work
    return ResponseEntity.accepted().header("Location", "/reports/" + id).build();
}

@KafkaListener(topics = "report-requests", groupId = "report-workers")
public void generate(ReportRequest r) { reports.complete(r); }
```

<h3>Design Points</h3>
<ul>
<li>Idempotent consumers, because delivery is at-least-once.</li>
<li>Give the client a status endpoint, a callback (webhook) or a notification for the result.</li>
<li>Retry and dead letter handling, and backpressure through consumer concurrency and lag-based scaling.</li>
<li>Version and validate message schemas (Avro, JSON Schema, Protobuf with a schema registry).</li>
</ul>

<h3>Gotchas</h3>
<p>Async makes the system eventually consistent and harder to debug, so propagate correlation IDs. Use <code>@Async</code> or an in-process executor only when losing work on a crash is acceptable.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 8. How do you ensure message ordering and delivery reliability in distributed messaging systems?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Ordering in Kafka is guaranteed only <strong>within a partition</strong>, so route related messages by key to the same partition. Reliability comes from the delivery semantics you choose, plus idempotent handling.</p>

<h3>Ordering</h3>
<ul>
<li>Use a key such as <code>orderId</code>, so the same key always maps to the same partition, and one consumer in the group processes it in order.</li>
<li>No global order across partitions. Using one partition gives a total order but no parallelism.</li>
<li>Producer retries can reorder messages unless <code>enable.idempotence=true</code> (which keeps order with up to 5 in-flight requests) or <code>max.in.flight.requests.per.connection=1</code>.</li>
<li>Changing the partition count changes key-to-partition mapping, which breaks ordering for existing keys.</li>
<li>Within a consumer, process each partition sequentially. Handing records to a thread pool breaks ordering.</li>
</ul>

<h3>Delivery Semantics</h3>
<table>
<tr><th>Semantics</th><th>How</th><th>Risk</th></tr>
<tr><td>At-most-once</td><td>Commit before processing</td><td>Message loss</td></tr>
<tr><td>At-least-once</td><td>Commit after processing</td><td>Duplicates, so use idempotent handlers</td></tr>
<tr><td>Exactly-once</td><td>Idempotent producer plus transactions, <code>read_committed</code></td><td>Throughput and complexity. Scoped to Kafka</td></tr>
</table>

<h3>Other Brokers</h3>
<p>RabbitMQ keeps order per queue, but competing consumers and redelivery can break it. Use acknowledgements, publisher confirms and durable queues, and use a single active consumer or consistent-hash exchange when order matters.</p>

<h3>Gotcha</h3>
<p>Retry topics can break ordering, since a retried message arrives after later ones. Block the partition on failure if strict order matters, or design events to tolerate reordering (version numbers, last-write-wins).</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 9. How do you handle errors in distributed messaging systems?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Classify the error first. Retry transient errors with backoff, send permanent ones (poison messages) to a dead letter queue or topic, and protect downstream dependencies with a circuit breaker. Never let one bad message block the whole queue.</p>

<h3>Error Types</h3>
<table>
<tr><th>Type</th><th>Examples</th><th>Handling</th></tr>
<tr><td>Transient</td><td>Timeout, DB deadlock, downstream 503</td><td>Retry with exponential backoff and jitter</td></tr>
<tr><td>Permanent</td><td>Deserialization error, validation failure, bug</td><td>No retry, dead letter immediately</td></tr>
<tr><td>Downstream outage</td><td>Dependency down for minutes</td><td>Circuit breaker, pause the consumer, resume later</td></tr>
</table>

<h3>Patterns</h3>
<ul>
<li><strong>Retry topics:</strong> non-blocking retries via topics such as <code>orders-retry-1s</code>, <code>orders-retry-10s</code>, so the main partition keeps moving (Spring Kafka <code>@RetryableTopic</code>).</li>
<li><strong>Dead letter queue/topic:</strong> keep the payload, headers, error and stack trace. Alert and provide a replay tool.</li>
<li><strong>Circuit breaker or pause:</strong> pause the listener container when a dependency is down instead of burning retries.</li>
<li><strong>Idempotency:</strong> so replays and redeliveries are safe.</li>
<li><strong>Handle deserialization errors</strong> with <code>ErrorHandlingDeserializer</code>, otherwise the consumer loops on the same bad record.</li>
</ul>

```java
@RetryableTopic(attempts = "4", backoff = @Backoff(delay = 1000, multiplier = 2),
                dltStrategy = DltStrategy.FAIL_ON_ERROR)
@KafkaListener(topics = "orders")
public void on(Order o) { service.process(o); }

@DltHandler
public void dead(Order o) { alerts.notify(o); }
```

<h3>Gotchas</h3>
<ul>
<li>Non-blocking retries give up strict ordering.</li>
<li>Monitor DLT growth and set a policy for who fixes and replays messages.</li>
<li>In RabbitMQ use a dead letter exchange plus TTL queues for delayed retries.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---

## 10. When should you use RabbitMQ vs Kafka?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Use <strong>RabbitMQ</strong> as a smart broker for task queues and flexible routing, where each message is consumed and removed. Use <strong>Kafka</strong> as a distributed log for high-throughput event streaming, replay and many independent consumers.</p>

<h3>Comparison</h3>
<table>
<tr><th>Aspect</th><th>RabbitMQ</th><th>Kafka</th></tr>
<tr><td>Model</td><td>Broker pushes messages to consumers, queues and exchanges</td><td>Consumers pull from a partitioned log</td></tr>
<tr><td>Retention</td><td>Removed after ack</td><td>Kept by time or size, replayable</td></tr>
<tr><td>Routing</td><td>Rich (direct, topic, fanout, headers)</td><td>Topic and partition key only</td></tr>
<tr><td>Ordering</td><td>Per queue (weakened by competing consumers)</td><td>Per partition</td></tr>
<tr><td>Throughput</td><td>Tens of thousands per second</td><td>Very high, millions per second at scale</td></tr>
<tr><td>Per-message features</td><td>Priority, TTL, delayed messages, per-message ack</td><td>Not built in</td></tr>
<tr><td>Scaling consumers</td><td>Add consumers to a queue</td><td>Limited by partition count</td></tr>
</table>

<h3>Choose RabbitMQ When</h3>
<ul>
<li>Background jobs, work distribution, request-reply, complex routing, priorities or delays.</li>
<li>Moderate volume and low latency per message.</li>
</ul>

<h3>Choose Kafka When</h3>
<ul>
<li>Event streaming, event sourcing, log or metrics pipelines, stream processing (Kafka Streams, Flink).</li>
<li>Many consumers need the same data, or you need replay and long retention.</li>
</ul>

<h3>Gotchas</h3>
<p>Kafka is operationally heavier. RabbitMQ also offers streams (since 3.9) that add log-like behavior. Cloud options (SQS/SNS, Pub/Sub) are often simpler when you do not need Kafka features.</p>

<p>📚 <a href="/anandu-interview-playbook/microservices-distributed/messaging/">Learn more: Messaging & Kafka</a></p>

</details>

---
