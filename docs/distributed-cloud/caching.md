# Caching

## Why cache
Reduce latency and load on databases/downstreams by keeping frequently-read data close.

## Patterns
- **Cache-aside (lazy)** — app checks cache; on miss, loads from DB and populates. Most common.
- **Read-through** — cache library loads from DB on miss.
- **Write-through** — write to cache + DB synchronously (consistent, slower writes).
- **Write-behind** — write to cache, async flush to DB (fast, risk of loss).

## Eviction policies
**LRU** (least recently used), LFU, FIFO, **TTL** (time-based). Bound size to avoid memory pressure.

## Local vs distributed
| | Local (in-JVM: EhCache, Caffeine) | Distributed (Redis, Memcached) |
|--|-----------------------------------|-------------------------------|
| Speed | fastest (no network) | network hop |
| Scope | per instance (coherence issue) | shared across instances |
| Capacity | heap-bound | large, independent |
| Survives restart | no | yes |

## Invalidation ("one of the two hard things")
TTL expiry, explicit eviction on write, or an explicit refresh endpoint. Beware **stale data** and
**cache stampede / thundering herd** on expiry → mitigate with TTL jitter, locks, or request coalescing.

## Redis
In-memory data store; strings/hashes/lists/sets/sorted-sets; TTL; pub/sub; used as distributed cache,
session store, rate limiter, leaderboard.

## Spring Cache Abstraction

Spring provides unified caching annotations over multiple providers (Caffeine, EhCache, Redis).

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Choose provider -->
<!-- EhCache 3 -->
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>

<!-- OR Caffeine (lightweight, no config) -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>

<!-- OR Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### Enable Caching

```java
@SpringBootApplication
@EnableCaching  // Activate caching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Core Annotations

```java
@Service
public class UserService {
    
    // @Cacheable: Return cached value if exists; otherwise execute & cache result
    @Cacheable(value = "users", key = "#id")
    public User getUser(Long id) {
        return userRepository.findById(id).orElse(null);  // DB hit only on miss
    }
    
    // @CachePut: Always execute & update cache (write-through)
    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        return userRepository.save(user);  // DB updated, cache refreshed
    }
    
    // @CacheEvict: Remove from cache (invalidation)
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);  // DB deleted, cache invalidated
    }
    
    // @CacheEvict with allEntries: Clear entire cache
    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        // Invalidate all users cache
    }
    
    // Conditional caching (cache only if condition true)
    @Cacheable(value = "users", key = "#id", condition = "#id > 0")
    public User getUserIfValid(Long id) {
        return userRepository.findById(id).orElse(null);
    }
    
    // Unless: Cache all except when condition true
    @Cacheable(value = "users", key = "#id", unless = "#result == null")
    public User getUserNonNull(Long id) {
        return userRepository.findById(id).orElse(null);  // Don't cache null
    }
}
```

### Configuration

#### EhCache Configuration

```xml
<!-- ehcache.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.ehcache.org/v3">
    <cache alias="users">
        <key-type>java.lang.Long</key-type>
        <value-type>com.example.User</value-type>
        <expiry>
            <tti unit="minutes">30</tti>  <!-- Time To Idle: evict if unused for 30 min -->
        </expiry>
        <resources>
            <heap unit="entries">10000</heap>  <!-- Max 10k entries in heap -->
        </resources>
    </cache>
    
    <cache alias="orders">
        <key-type>java.lang.Long</key-type>
        <value-type>com.example.Order</value-type>
        <expiry>
            <ttl unit="hours">1</ttl>  <!-- Time To Live: expire after 1 hour -->
        </expiry>
        <resources>
            <heap unit="entries">5000</heap>
        </resources>
    </cache>
</config>
```

```yaml
# application.yml
spring:
  cache:
    type: ehcache  # Use EhCache provider
    ehcache:
      config: classpath:ehcache.xml
```

#### Caffeine Configuration (No XML Needed)

```yaml
# application.yml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=10m  # Config as string
```

```java
// Or programmatically
@Configuration
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("users", "orders");
        cacheManager.setCaffeine(
            Caffeine.newBuilder()
                .maximumSize(10000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()  // Enable statistics
        );
        return cacheManager;
    }
}
```

#### Redis Configuration

```yaml
# application.yml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minutes in milliseconds
      key-prefix: "cache:"
      use-key-prefix: true
  
  redis:
    host: localhost
    port: 6379
```

### Advanced: Custom Key Generation

```java
// By default: key="#id" uses method parameter name
@Cacheable(value = "users", key = "#id")
public User getUser(Long id) { ... }

// Custom key with multiple params
@Cacheable(value = "users", key = "#userId + ':' + #includeOrders")
public User getUser(Long userId, boolean includeOrders) { ... }

// SpEL expressions
@Cacheable(value = "users", key = "T(java.lang.String).format('user:%d', #id)")
public User getUser(Long id) { ... }

// Custom key generator class
@Configuration
public class CacheConfig {
    @Bean
    public KeyGenerator customKeyGenerator() {
        return (target, method, params) -> {
            String methodName = method.getName();
            return methodName + ":" + Arrays.toString(params);
        };
    }
}

@Cacheable(value = "users", keyGenerator = "customKeyGenerator")
public User getUser(Long id) { ... }
```

### Monitoring & Statistics

```java
@Component
public class CacheStats {
    
    @Autowired
    private CacheManager cacheManager;
    
    @Scheduled(fixedRate = 60000)  // Every minute
    public void printCacheStats() {
        for (String cacheName : cacheManager.getCacheNames()) {
            Cache cache = cacheManager.getCache(cacheName);
            
            // Get native cache implementation (Caffeine)
            if (cache.getNativeCache() instanceof com.github.benmanes.caffeine.cache.Cache) {
                com.github.benmanes.caffeine.cache.Cache<?> caffeineCache = 
                    (com.github.benmanes.caffeine.cache.Cache<?>) cache.getNativeCache();
                
                System.out.println("Cache: " + cacheName);
                System.out.println("  Size: " + caffeineCache.estimatedSize());
                System.out.println("  Stats: " + caffeineCache.stats());
            }
        }
    }
}
```

---

## EhCache: Local In-JVM Cache

**EhCache** — Popular, production-grade local caching library for Java.

### POM Dependency

```xml
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.10.8</version>
</dependency>
```

### Programmatic Configuration

```java
@Configuration
public class EhCacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CacheConfiguration<Long, User> userCacheConfig = CacheConfigurationBuilder
            .newCacheConfigurationBuilder(Long.class, User.class,
                ResourcePoolsBuilder.heap(10000))  // 10k entries
            .withExpiry(ExpiryPolicyBuilder.timeToIdleExpiration(
                Duration.ofMinutes(30)))  // Evict if unused for 30 min
            .withEvictionPriority(EvictionPriority.FIFO)
            .build();
        
        CacheConfiguration<Long, Order> orderCacheConfig = CacheConfigurationBuilder
            .newCacheConfigurationBuilder(Long.class, Order.class,
                ResourcePoolsBuilder.heap(5000))
            .withExpiry(ExpiryPolicyBuilder.timeToLiveExpiration(
                Duration.ofHours(1)))  // Expire after 1 hour
            .build();
        
        CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()
            .withCache("users", userCacheConfig)
            .withCache("orders", orderCacheConfig)
            .build(true);  // init on creation
        
        return new org.springframework.cache.ehcache.EhCacheCacheManager(cacheManager);
    }
}
```

### Direct API Usage (Without Spring)

```java
// Create cache programmatically
CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()
    .withCache("users",
        CacheConfigurationBuilder
            .newCacheConfigurationBuilder(Long.class, String.class,
                ResourcePoolsBuilder.heap(100))
            .build())
    .build(true);

Cache<Long, String> cache = cacheManager.getCache("users", Long.class, String.class);

// Put
cache.put(1L, "John");

// Get
String user = cache.get(1L);

// Remove
cache.remove(1L);

// Close
cacheManager.close();
```

### TTL vs TTI

| Type | Behavior |
|------|----------|
| **TTL (Time-To-Live)** | Expires after set duration, regardless of access |
| **TTI (Time-To-Idle)** | Expires if not accessed within set duration |

```xml
<!-- TTL: Entry expires 1 hour after creation -->
<expiry>
    <ttl unit="hours">1</ttl>
</expiry>

<!-- TTI: Entry expires if not accessed for 30 minutes -->
<expiry>
    <tti unit="minutes">30</tti>
</expiry>
```

### Eviction Policies

| Policy | Eviction |
|--------|-----------|
| **FIFO** | First-In-First-Out |
| **LRU** | Least Recently Used |
| **LFU** | Least Frequently Used |

```xml
<cache>
    <resources>
        <heap unit="entries">10000</heap>
    </resources>
    <eviction-adviser>
        <!-- Custom eviction logic -->
    </eviction-adviser>
</cache>
```

### Three-Tier Caching (Heap, Offheap, Disk)

```xml
<cache>
    <resources>
        <heap unit="entries">1000</heap>      <!-- Fast, in JVM heap -->
        <offheap unit="MB">100</offheap>      <!-- Slower, native memory -->
        <disk unit="MB">1000</disk>           <!-- Persistent, slow -->
    </resources>
</cache>
```

Data flows: Heap → Offheap → Disk as size increases.

---

## Cache Invalidation Strategies

### 1. TTL/TTI Expiry (Time-based)

Automatically remove after fixed duration or inactivity:
```java
@Cacheable(value = "users", key = "#id")
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}
// Cache expires after 10 minutes (config)
```

**Pros:** Simple, no code needed  
**Cons:** May serve stale data, or lose fresh data

### 2. Explicit Invalidation (Event-based)

Manually evict on known changes:
```java
@CacheEvict(value = "users", key = "#id")
public void updateUser(User user) {
    userRepository.save(user);
}

@CacheEvict(value = "users", allEntries = true)
public void bulkUpdate() { ... }
```

**Pros:** Always fresh  
**Cons:** Must remember to evict

### 3. Refresh Endpoint

Admin endpoint to clear cache:
```java
@PostMapping("/admin/cache/clear")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<Void> clearCache() {
    cacheManager.getCacheNames()
        .forEach(name -> cacheManager.getCache(name).clear());
    return ResponseEntity.noContent().build();
}
```

### 4. Cache Stampede Mitigation

When many threads miss cache simultaneously, prevent dogpiling:

```java
// Approach 1: Probabilistic TTL jitter
@Cacheable(value = "users", key = "#id")
public User getUser(Long id) {
    return userRepository.findById(id).orElse(null);
}
// Add jitter in config: TTL ± random(0-10%)

// Approach 2: Lock-based loading
@Service
public class UserService {
    private final Map<Long, Boolean> locks = new ConcurrentHashMap<>();
    
    public User getUser(Long id) {
        User cached = getFromCache(id);
        if (cached != null) return cached;
        
        // Only one thread loads; others wait
        synchronized(locks.computeIfAbsent(id, k -> new Object())) {
            cached = getFromCache(id);
            if (cached == null) {
                cached = userRepository.findById(id).orElse(null);
                if (cached != null) putInCache(id, cached);
            }
        }
        return cached;
    }
}
```

---

## Local vs Distributed Cache Comparison

| Aspect | Local (EhCache, Caffeine) | Distributed (Redis) |
|--------|---------------------------|-------------------|
| **Latency** | Microseconds (in-JVM) | Milliseconds (network) |
| **Scope** | Per instance | Shared across all instances |
| **Consistency** | Can be stale | More consistent (single source) |
| **Memory** | Bounded by JVM heap | Separate Redis server |
| **Survives restart** | No | Yes |
| **Serialization** | Not needed | Needed (JSON/binary) |
| **Best for** | High-traffic local ops | Multi-instance, consistency-critical |
