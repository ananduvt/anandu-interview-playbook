# Spring Caching Interview Questions

Caching fundamentals, Spring Cache abstraction with Redis, and consistency pitfalls.

---

## 1. What is caching and how does it improve performance? What are the main types of caching?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Caching keeps frequently used or expensive-to-compute data in a faster layer so repeated requests skip the slow source (database, remote call, computation). It lowers latency and backend load and increases throughput, at the cost of possible staleness and extra complexity.</p>

<h3>Common Cache Types</h3>
<table>
<tr><th>Type</th><th>What it caches</th><th>Examples</th></tr>
<tr><td>CDN / edge</td><td>Static assets, cacheable API responses near users</td><td>CloudFront, Cloudflare</td></tr>
<tr><td>Web server / reverse proxy / browser</td><td>HTTP responses (<code>Cache-Control</code>, <code>ETag</code>)</td><td>Nginx, Varnish, browser cache</td></tr>
<tr><td>In-memory application cache</td><td>Objects/results in the JVM (local) or a shared store (distributed)</td><td>Caffeine, Redis, Hazelcast</td></tr>
<tr><td>Database cache</td><td>Query results, buffer pool, ORM first/second-level cache</td><td>Buffer pool, Hibernate L2</td></tr>
</table>

<h3>Local vs Distributed</h3>
<ul>
<li><strong>Local</strong> (Caffeine): fastest, no network, but one copy per instance and inconsistent across instances.</li>
<li><strong>Distributed</strong> (Redis): shared, consistent view, survives app restarts; adds network hop and an operational dependency.</li>
</ul>

<h3>Key Concepts</h3>
<ul>
<li>Hit ratio decides the benefit; measure it.</li>
<li>Eviction (LRU/LFU), TTL, and size limits bound memory.</li>
<li>Cache only data that is read often and changes rarely; avoid caching highly personalised or fast-changing data.</li>
<li>Cache stampede: protect with request coalescing, jittered TTLs, or locking.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/caching/">Learn more: Caching</a></p>

</details>

---

## 2. How do you implement Redis caching with Spring Boot?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Add <code>spring-boot-starter-data-redis</code> and <code>spring-boot-starter-cache</code>, set the Redis connection properties, enable caching with <code>@EnableCaching</code>, and annotate methods with <code>@Cacheable</code>, <code>@CachePut</code>, and <code>@CacheEvict</code>. Boot auto-configures a <code>RedisCacheManager</code>.</p>

```yaml
spring:
  data.redis:
    host: localhost
    port: 6379
  cache:
    type: redis
    redis:
      time-to-live: 10m
```

```java
@Configuration
@EnableCaching
class CacheConfig {
    @Bean
    RedisCacheConfiguration cacheConfig() {
        return RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .disableCachingNullValues()
                .serializeValuesWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}

@Service
class ProductService {
    @Cacheable(cacheNames = "products", key = "#id")
    public Product get(long id) { return repo.findById(id).orElseThrow(); }

    @CachePut(cacheNames = "products", key = "#p.id")
    public Product update(Product p) { return repo.save(p); }

    @CacheEvict(cacheNames = "products", key = "#id")
    public void delete(long id) { repo.deleteById(id); }
}
```

<h3>Gotchas</h3>
<ul>
<li>Spring Boot 3.x property prefix is <code>spring.data.redis</code> (it was <code>spring.redis</code> in 2.x).</li>
<li>Cache annotations use AOP proxies: self-invocation and private methods are not cached.</li>
<li>Default serialisation is JDK serialisation; prefer JSON and a TTL so entries cannot live forever.</li>
<li>Use <code>RedisTemplate</code>/<code>StringRedisTemplate</code> directly for data structures, counters, or locks.</li>
<li>Redis outages: decide whether the app falls back to the database (custom <code>CacheErrorHandler</code>) or fails.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/caching/">Learn more: Caching</a></p>

</details>

---

## 3. What are the pitfalls of caching in financial applications?

<details>
<summary>View Answer</summary>

<h3>Direct Answer</h3>
<p>Financial data (balances, limits, rates, ledgers) needs strong correctness, so stale or inconsistent cache entries can cause wrong decisions, overdrafts, or audit failures. Caching must be selective and conservative.</p>

<h3>Main Pitfalls</h3>
<ul>
<li><strong>Stale reads:</strong> a cached balance or limit after a debit lets a second request approve something it should not.</li>
<li><strong>Invalidation in distributed setups:</strong> local caches on several instances diverge; evict messages can be lost or arrive late; race between write and invalidate repopulating old data.</li>
<li><strong>Write-behind data loss:</strong> acknowledged writes in cache only can vanish on a crash.</li>
<li><strong>Dual-write inconsistency:</strong> updating DB and cache separately can partially fail.</li>
<li><strong>Stampede on hot keys</strong> (market open, month-end) overwhelming the DB.</li>
<li><strong>Sensitive data exposure:</strong> cached PII/card data without encryption, access control, or TTL; compliance and audit issues.</li>
<li><strong>Cache poisoning/serialisation changes</strong> across deployments.</li>
</ul>

<h3>Mitigations</h3>
<ul>
<li>Do not cache authoritative balances for decisions; read from the source of truth inside the transaction. Cache reference data (currencies, product catalogue, FX rates with short TTL).</li>
<li>Prefer cache-aside with evict-after-commit (<code>@TransactionalEventListener(AFTER_COMMIT)</code>) and short TTLs as a safety net.</li>
<li>Use versioned keys or version checks (optimistic locking) so stale writes are rejected.</li>
<li>Single shared distributed cache instead of per-node caches where consistency matters.</li>
<li>Monitor hit/miss, staleness, and eviction; keep an audit trail that does not depend on cache.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/distributed-cloud/caching/">Learn more: Caching</a></p>

</details>

---
