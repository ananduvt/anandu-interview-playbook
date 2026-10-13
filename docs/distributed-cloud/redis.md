# Redis Advanced Patterns

In-memory data store for caching, sessions, pub/sub, and real-time features.

---

## Redis Data Structures

### Strings
```redis
SET key value [EX seconds] [NX|XX]
GET key
MSET key1 value1 key2 value2
MGET key1 key2

INCR counter              # Atomic increment
DECR counter              # Atomic decrement
INCRBY counter 5          # Increment by 5
APPEND key "text"         # Append to string
GETRANGE key 0 5          # Substring
STRLEN key                # String length
```

### Lists (FIFO/LIFO)
```redis
LPUSH list item1 item2    # Push to left (head)
RPUSH list item3          # Push to right (tail)
LPOP list                 # Pop from left
RPOP list                 # Pop from right
LLEN list                 # Length
LRANGE list 0 -1          # Get all items
LINDEX list 0             # Get by index
LTRIM list 0 10           # Keep first 11 items, delete rest (often used)
LINSERT list BEFORE pivot item # Insert before/after pivot
```

**Use Cases:** Job queue, activity feed, messaging

### Sets (Unique Values, No Order)
```redis
SADD myset member1 member2 member3
SREM myset member1
SMEMBERS myset            # All members
SCARD myset               # Count
SISMEMBER myset member1   # Check membership

SINTER set1 set2          # Intersection (common elements)
SUNION set1 set2          # Union (all elements)
SDIFF set1 set2           # Difference (in set1 but not set2)
SRANDMEMBER myset         # Random member
SPOP myset                # Remove and return random
```

**Use Cases:** Unique visitors, followers, tags, deduplication

### Sorted Sets (Ordered by Score)
```redis
ZADD users 100 john 200 jane 150 bob
ZRANGE users 0 -1         # All members in score order
ZREVRANGE users 0 -1      # Reverse order
ZRANGE users 0 -1 WITHSCORES  # Include scores
ZRANK users john          # Rank (0-indexed)
ZREVRANK users john       # Reverse rank
ZCARD users               # Count
ZSCORE users john         # Score of member
ZINCRBY users 10 john     # Increment score
ZCOUNT users 100 200      # Count members in score range
ZRANGEBYSCORE users 100 200  # Members in range
```

**Use Cases:** Leaderboards, priority queues, time-series data, rankings

### Hashes (Key-Value Pairs)
```redis
HSET user:1 name john age 30 email john@example.com
HGET user:1 name          # Get one field
HMGET user:1 name email   # Get multiple fields
HGETALL user:1            # Get all fields
HDEL user:1 age           # Delete field
HEXISTS user:1 name       # Check field exists
HLEN user:1               # Count fields
HKEYS user:1              # All field names
HVALS user:1              # All values
HINCRBY user:1 age 1      # Increment numeric field
```

**Use Cases:** User profiles, session data, object storage

---

## Caching Patterns

### Cache-Aside (Lazy Loading)
Application is responsible for loading cache:
```java
public User getUser(Long id) {
    String cacheKey = "user:" + id;
    User cached = redisTemplate.opsForValue().get(cacheKey);
    
    if (cached != null) {
        return cached;  // Cache hit
    }
    
    // Cache miss → load from DB
    User user = userRepository.findById(id).orElse(null);
    if (user != null) {
        redisTemplate.opsForValue().set(cacheKey, user, Duration.ofHours(1));
    }
    return user;
}
```

**Pros:** Simple, doesn't require cache library  
**Cons:** Cache misses hit DB, inconsistency possible

### Write-Through
Write to cache and DB simultaneously:
```java
public void updateUser(User user) {
    // Write to cache first
    redisTemplate.opsForValue().set("user:" + user.getId(), user);
    // Then write to DB
    userRepository.save(user);
}
```

**Pros:** Cache always consistent with DB  
**Cons:** Slower writes, extra latency

### Write-Behind (Write-Back)
Write to cache immediately, write to DB asynchronously:
```java
public void updateUser(User user) {
    // Write to cache immediately (fast)
    redisTemplate.opsForValue().set("user:" + user.getId(), user);
    
    // Write to DB asynchronously
    asyncService.saveUserToDB(user);
}
```

**Pros:** Fast writes, high throughput  
**Cons:** Risk of data loss if cache crashes before DB sync

---

## Advanced Patterns

### Distributed Locking
Prevent multiple processes from accessing same resource:
```java
@Component
public class DistributedLock {
    private RedisTemplate<String, String> redisTemplate;
    
    public boolean acquireLock(String key, String value, long timeoutSeconds) {
        // Try to SET only if not exists
        return redisTemplate.opsForValue().setIfAbsent(
            key, value, Duration.ofSeconds(timeoutSeconds)
        );
    }
    
    public void releaseLock(String key, String value) {
        // Only delete if value matches (prevent deleting others' locks)
        String current = redisTemplate.opsForValue().get(key);
        if (value.equals(current)) {
            redisTemplate.delete(key);
        }
    }
}

// Usage
String lockKey = "order:123:processing";
String lockValue = UUID.randomUUID().toString();

if (distributedLock.acquireLock(lockKey, lockValue, 30)) {
    try {
        processOrder(123);
    } finally {
        distributedLock.releaseLock(lockKey, lockValue);
    }
} else {
    // Lock already held by another process
    throw new LockException("Order is being processed");
}
```

### Session Storage
Replace HTTP sessions with Redis:
```yaml
spring:
  session:
    store-type: redis
    timeout: 30m  # Session timeout
```

```java
// Sessions automatically stored in Redis
@PostMapping("/login")
public ResponseEntity<String> login(HttpSession session) {
    session.setAttribute("userId", user.getId());
    session.setAttribute("role", user.getRole());
    return ResponseEntity.ok("Logged in");
}

@GetMapping("/profile")
public ResponseEntity<User> getProfile(HttpSession session) {
    Long userId = (Long) session.getAttribute("userId");
    return userService.findById(userId);
}
```

### Rate Limiting (Sliding Window)
Limit requests using Redis counters:
```java
@Component
public class RateLimiter {
    private RedisTemplate<String, String> redisTemplate;
    
    public boolean isAllowed(String userId, int maxRequests, int windowSeconds) {
        String key = "rate:" + userId;
        long now = System.currentTimeMillis() / 1000;
        String score = String.valueOf(now);
        
        // Add current request
        redisTemplate.opsForZSet().add(key, score, now);
        
        // Remove old requests outside window
        redisTemplate.opsForZSet().removeRangeByScore(key, 0, now - windowSeconds);
        
        // Count requests in window
        Long requestCount = redisTemplate.opsForZSet().size(key);
        
        // Set expiry on key
        redisTemplate.expire(key, Duration.ofSeconds(windowSeconds + 1));
        
        return requestCount <= maxRequests;
    }
}

// Usage
@Component
public class RateLimitingInterceptor implements HandlerInterceptor {
    private RateLimiter rateLimiter;
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String userId = SecurityContextHolder.getContext().getAuthentication().getName();
        
        if (!rateLimiter.isAllowed(userId, 100, 60)) {  // 100 requests per minute
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            return false;
        }
        return true;
    }
}
```

### Leaderboard (Top N)
Store rankings using sorted sets:
```java
@Component
public class LeaderboardService {
    private RedisTemplate<String, Object> redisTemplate;
    
    // Record user score
    public void recordScore(String userId, int score) {
        redisTemplate.opsForZSet().add("leaderboard", userId, score);
    }
    
    // Get top 10 (highest scores first)
    public List<ZSetOperations.TypedTuple<Object>> getTopTen() {
        return redisTemplate.opsForZSet().reverseRangeWithScores("leaderboard", 0, 9);
    }
    
    // Get user rank
    public Long getUserRank(String userId) {
        return redisTemplate.opsForZSet().reverseRank("leaderboard", userId);
    }
}
```

### Pub/Sub (Publish-Subscribe)
Real-time messaging:
```java
// Publisher
@Component
public class OrderEventPublisher {
    private RedisTemplate<String, String> redisTemplate;
    
    public void publishOrderCreated(Order order) {
        String message = objectMapper.writeValueAsString(order);
        redisTemplate.convertAndSend("orders:created", message);
    }
}

// Subscriber
@Component
public class OrderEventSubscriber {
    @Bean
    public MessageListenerAdapter orderListener() {
        return new MessageListenerAdapter(new OrderMessageListener());
    }
    
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.addMessageListener(
            orderListener(),
            new PatternTopic("orders:*")  // Subscribe to orders:created, orders:updated, etc.
        );
        return container;
    }
}

@Component
public class OrderMessageListener implements MessageListener {
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String orderJson = new String(message.getBody());
        Order order = objectMapper.readValue(orderJson, Order.class);
        handleOrderEvent(order);
    }
}
```

---

## Redis Cluster & High Availability

### Master-Slave Replication
Data replicated from master to slaves (read-only):
```yaml
# Master
port: 6379

# Slave (read-only)
port: 6380
slaveof 127.0.0.1 6379  # Replicate from master
```

### Sentinel (Automatic Failover)
Monitors master, promotes slave if master fails:
```yaml
# sentinel.conf
sentinel monitor mymaster 127.0.0.1 6379 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
```

Spring Boot Config:
```yaml
spring:
  redis:
    sentinel:
      master: mymaster
      nodes: 127.0.0.1:26379, 127.0.0.1:26380, 127.0.0.1:26381
    timeout: 2000
```

### Redis Cluster (Sharding)
Data split across multiple nodes:
- **Partitioning**: Hash slot (0-16383) determines node
- **Replication**: Each slot has master + slaves
- **Failover**: Automatic if master fails

```
Cluster setup:
redis-server --port 7000 --cluster-enabled yes
redis-server --port 7001 --cluster-enabled yes
...
redis-cli --cluster create 127.0.0.1:7000 ... --cluster-replicas 1
```

---

## Persistence & Backups

### RDB (Redis Database)
Snapshot of data at point-in-time:
```conf
save 900 1      # Save if 1 change in 900 seconds
save 300 10     # Save if 10 changes in 300 seconds
save 60 10000   # Save if 10000 changes in 60 seconds
```

**Pros:** Compact file, fast recovery  
**Cons:** Data loss between snapshots

### AOF (Append-Only File)
Log every write command:
```conf
appendonly yes
appendfsync everysec  # Fsync every second (balance speed/safety)
```

**Pros:** Minimal data loss, can replay commands  
**Cons:** Larger file size, slower performance

### Hybrid Approach
Use both RDB + AOF for best safety/performance.

---

## Monitoring & Debugging

### INFO Command
```bash
redis-cli INFO server    # Server info
redis-cli INFO stats     # Statistics (commands, keys, clients)
redis-cli INFO memory    # Memory usage
redis-cli INFO clients   # Connected clients
```

### SLOWLOG
Track slow commands:
```bash
redis-cli SLOWLOG GET 10      # Last 10 slow commands
redis-cli SLOWLOG LEN         # Count of slow commands
redis-cli SLOWLOG RESET       # Clear slowlog
```

### Monitor
Real-time command trace:
```bash
redis-cli MONITOR             # Shows every command in real-time
```

### Memory Analysis
```bash
redis-cli --memkeys           # Memory per key
redis-cli OBJECT ENCODING key # Encoding of value
redis-cli DEBUG OBJECT key    # Memory info
```

---

## Best Practices

✅ **Do's**
- Set TTL on cache keys (don't store forever)
- Use appropriate data structure (don't store sets as strings)
- Monitor memory usage (Redis is in-memory)
- Use Redis Cluster for high availability
- Implement circuit breaker (if Redis down, gracefully degrade)
- Version cache keys when schema changes
- Use connection pooling

❌ **Don'ts**
- Don't store sensitive data unencrypted
- Don't rely on Redis for durable storage (use DB)
- Don't use Redis without TTL (memory leaks)
- Don't make Redis calls in loops (use pipeline)
- Don't block on operations (use async)
- Don't ignore Redis AUTH in production
- Don't use Redis for large objects without compression

---

## Spring Boot Integration

### Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>io.lettuce</groupId>  <!-- or jedis -->
    <artifactId>lettuce-core</artifactId>
</dependency>
```

### Configuration
```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: your-password
    timeout: 2000
    lettuce:
      pool:
        max-active: 20
        max-idle: 10
        min-idle: 5
```

### Basic Operations
```java
@Service
public class CacheService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // String operations
    public void set(String key, Object value, long timeout) {
        redisTemplate.opsForValue().set(key, value, Duration.ofSeconds(timeout));
    }
    
    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    
    // Set operations
    public void addToSet(String key, Object... members) {
        redisTemplate.opsForSet().add(key, members);
    }
    
    public Set<Object> getSet(String key) {
        return redisTemplate.opsForSet().members(key);
    }
    
    // Hash operations
    public void putHash(String key, String hashKey, Object value) {
        redisTemplate.opsForHash().put(key, hashKey, value);
    }
    
    public Map<Object, Object> getHash(String key) {
        return redisTemplate.opsForHash().getAll(key);
    }
}
```

---

**See also:** [Caching](caching.md), [AWS ElastiCache](aws.md), [Spring Data Redis](../spring-web/spring-boot-advanced.md)

