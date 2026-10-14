# System Design Interview Questions

Designing large-scale systems: scalability, reliability, performance, and trade-offs.

---

## 1. How do you approach a system design problem?

<details>
<summary>View Answer</summary>

<h3>Step-by-Step Approach</h3>

<ol>
<li><strong>Clarify requirements (functional)</strong>
  <ul>
  <li>What features? Scale? User base?</li>
  </ul>
</li>
<li><strong>Non-functional requirements (NFR)</strong>
  <ul>
  <li>Latency? Throughput? Availability? Consistency?</li>
  </ul>
</li>
<li><strong>Back-of-envelope calculations</strong>
  <ul>
  <li>QPS, bandwidth, storage needed</li>
  </ul>
</li>
<li><strong>High-level architecture</strong>
  <ul>
  <li>Database, cache, queue, load balancer</li>
  </ul>
</li>
<li><strong>Dive into components</strong>
  <ul>
  <li>Database schema, API design, scaling strategy</li>
  </ul>
</li>
<li><strong>Discuss trade-offs</strong>
  <ul>
  <li>Why this choice? Alternatives?</li>
  </ul>
</li>
</ol>

<h3>Recommendation: Start monolithic, scale when needed</h3>

<p>📚 <a href="/anandu-interview-playbook/system-design/worked-examples.md">Learn more: System Design Examples</a></p>

</details>

---

**See also:** [System Design Fundamentals](../../system-design/fundamentals.md), [Building Blocks](../../system-design/building-blocks.md), [Interview Questions Overview](../index.md)
