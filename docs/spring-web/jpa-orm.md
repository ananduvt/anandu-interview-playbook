# JPA & ORM

**ORM** (Object-Relational Mapping) maps objects ↔ relational tables. **JPA** is the Java spec; **Hibernate** is the most common implementation. Reduces boilerplate, manages transactions and entity lifecycle.

---

## Hibernate: Object-Relational Mapping

### Entity Definition

```java
@Entity  // Marks class as JPA entity (maps to DB table)
@Table(name = "users", uniqueConstraints = {
    @UniqueConstraint(columnNames = "email"),
    @UniqueConstraint(columnNames = {"first_name", "last_name"})
})
public class User {
    
    // Primary key with auto-generation
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // DB auto-increment
    private Long id;
    
    // Column mapping
    @Column(name = "first_name", nullable = false, length = 100)
    private String firstName;
    
    @Column(unique = true)
    private String email;
    
    @Column(columnDefinition = "INT DEFAULT 0")  // Raw SQL definition
    private Integer loginCount;
    
    @Transient  // Not persisted
    private String tempCalculatedField;
    
    // Timestamp
    @Column(updatable = false)  // Set only on insert
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp  // Auto-updated on every change
    private LocalDateTime updatedAt;
}
```

### ID Generation Strategies

```java
public class IdGenerationExamples {
    
    // IDENTITY: DB auto-increment (MySQL, SQL Server)
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // SEQUENCE: Oracle, PostgreSQL native sequence
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
    @SequenceGenerator(name = "user_seq", sequenceName = "seq_users", allocationSize = 1)
    private Long id;
    
    // TABLE: Simulate sequence using a table (database-agnostic)
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "user_table_gen")
    @TableGenerator(name = "user_table_gen", table = "id_generators", 
        pkColumnName = "entity", pkColumnValue = "user", valueColumnName = "id_value")
    private Long id;
    
    // AUTO: Let Hibernate choose based on dialect
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    // UUID: Generate UUID on application side
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String id;
    
    // Manual: Provide ID yourself
    @Id
    private String customId;
}
```

### Relationships

```java
// One-to-Many: User has multiple Orders
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // One user has many orders
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @LazyCollection(LazyCollectionOption.EXTRA)  // Load only when needed
    private Set<Order> orders = new HashSet<>();
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)  // Default for @ManyToOne is EAGER; use LAZY
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
}

// Many-to-Many: Users and Groups
@Entity
public class User {
    @ManyToMany(cascade = CascadeType.PERSIST)
    @JoinTable(
        name = "user_groups",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "group_id")
    )
    private Set<Group> groups = new HashSet<>();
}

@Entity
public class Group {
    @ManyToMany(mappedBy = "groups")
    private Set<User> users = new HashSet<>();
}

// One-to-One
@Entity
public class User {
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}

@Entity
public class UserProfile {
    @OneToOne(mappedBy = "profile")
    private User user;
}
```

### Cascade Types

| Type | Behavior |
|------|----------|
| `PERSIST` | Save parent → save children |
| `MERGE` | Update parent → update children |
| `REMOVE` | Delete parent → delete children |
| `REFRESH` | Reload parent → reload children |
| `DETACH` | Detach parent → detach children |
| `ALL` | All of above |

```java
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items;  // Delete items when removed from list
```

### Persistence Context & Entity Lifecycle

Entity states:
1. **Transient** — Not known to Hibernate; created with `new`
2. **Persistent (Managed)** — Associated with EntityManager; changes tracked
3. **Detached** — Was managed, now disconnected (after session closes)
4. **Removed** — Marked for deletion

```java
@Service
public class UserService {
    private EntityManager em;
    
    public void demonstrateLifecycle() {
        // 1. TRANSIENT
        User user = new User();  // Not in DB, not managed
        
        // 2. PERSISTENT
        em.persist(user);  // Now managed; Hibernate tracks changes
        
        user.setName("John");  // Dirty checked at commit
        
        // 3. DETACHED
        em.detach(user);  // Remove from context
        user.setName("Jane");  // Changes NOT tracked
        
        // 4. MERGE back to PERSISTENT
        User merged = em.merge(user);  // Re-attach; changes sync'd
        
        // 5. REMOVED
        em.remove(merged);  // Mark for deletion
    }
}
```

### Session Lifecycle: persist() vs save() vs merge() vs update()

**Entity State Transitions:**

```
Transient → [persist/save] → Persistent → [detach] → Detached → [merge] → Persistent
```

| Method | Input State | Output State | DB Effect | Use Case |
|--------|------------|--------------|-----------|----------|
| **persist()** | Transient | Persistent | INSERT scheduled (flush) | New entity, guaranteed unique ID handling |
| **save()** | Transient or Detached | Persistent | INSERT or UPDATE (auto-detect) | General save (JPA/Spring Data shorthand) |
| **merge()** | Detached | Persistent (new instance) | UPDATE (or INSERT if new) | Re-attach modified detached entity |
| **update()** | Detached | Unchanged | UPDATE only | Explicit update (rare; use merge instead) |

**Detailed Comparison:**

```java
@Transactional
@Service
public class UserService {
    private EntityManager em;
    private UserRepository userRepo;  // Spring Data
    
    // ✅ persist() — new entity, guaranteed INSERT
    public void persistNewUser() {
        User user = new User("John", "john@example.com");  // Transient
        em.persist(user);  // Now Persistent; INSERT queued
        
        // At transaction commit:
        // 1. Hibernate generates ID
        // 2. INSERT executes
        // 3. ID is set on user object
        
        System.out.println(user.getId());  // ✅ ID available after flush/commit
    }
    
    // ✅ save() — auto-detect persist vs update
    public void saveUser() {
        User user = new User("Jane", "jane@example.com");  // Transient
        User saved = userRepo.save(user);  // Returns managed instance
        
        // Spring Data's save():
        // - If ID is null → persist() → INSERT
        // - If ID exists → merge() → UPDATE
        
        System.out.println(saved.getId());  // ✅ ID available immediately
    }
    
    // ⚠️ merge() — re-attach detached entity (creates new managed copy)
    public void mergeDetachedUser(Long userId) {
        // Get and detach
        User user = em.find(User.class, userId);  // Persistent
        em.detach(user);  // Now Detached
        
        user.setName("Updated Name");  // ❌ Changes NOT tracked!
        
        // Re-attach
        User managed = em.merge(user);  // Returns NEW managed instance
        managed.setEmail("new@example.com");  // ✅ Changes tracked
        
        // user != managed (different objects)
        // UPDATE executes at commit with managed's state
    }
    
    // ❌ update() — raw update, requires existing entity
    public void updateExistingUser(User detachedUser) {
        // Rarely used; merge() is preferred
        em.getTransaction().begin();
        em.update(detachedUser);  // UPDATE only (assumes entity exists)
        em.getTransaction().commit();
    }
}
```

**Key Insight: Dirty Checking vs Explicit Operations**

```java
@Transactional  // Critical: provides persistence context
public void demonstrateDirtychecking() {
    User user = userRepo.findById(1L).get();  // Persistent
    
    user.setName("New Name");
    
    // ❌ Don't need to call save()!
    // ✅ Dirty checking happens automatically:
    // 1. At transaction commit, Hibernate snapshots entity state
    // 2. Detects differences from original
    // 3. Generates UPDATE SQL
    // 4. Executes and commits
    
    // userRepo.save(user);  // ← Not needed for managed entities
}

// Without @Transactional: Changes are NOT tracked!
public void noTransactionNoUpdate() {
    User user = userRepo.findById(1L).get();
    user.setName("This won't persist");
    // No @Transactional = no transaction = no dirty check
    // Changes silently lost!
}
```

**Best Practices:**

```java
// ✅ DO: Use Spring Data save() for simplicity
User saved = userRepository.save(newUser);

// ✅ DO: Use @Transactional for automatic updates
@Transactional
public void updateUser(Long id, String newName) {
    User user = userRepository.findById(id).orElseThrow();
    user.setName(newName);  // Dirty check handles update
}

// ✅ DO: Always use merge() to re-attach detached entities
User managed = entityManager.merge(detachedUser);
managed.setEmail("new@example.com");

// ❌ DON'T: Use em.update() directly
// ❌ DON'T: Forget @Transactional (changes won't persist)
// ❌ DON'T: Assume detached entities are still managed
```

### Lazy Loading & N+1 Problem

```java
// ❌ N+1 problem
@Repository
public class UserRepository extends JpaRepository<User, Long> {
    public List<User> findAllWithOrders() {
        List<User> users = findAll();  // 1 query
        for (User user : users) {
            System.out.println(user.getOrders());  // N queries (one per user)
        }
        return users;
    }
}

// ✅ Fix 1: JOIN FETCH
@Repository
public class UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders")
    public List<User> findAllWithOrdersJoinFetch();
}

// ✅ Fix 2: Entity Graph
@Repository
public class UserRepository extends JpaRepository<User, Long> {
    @EntityGraph(attributePaths = "orders")
    @Query("SELECT u FROM User u")
    public List<User> findAllWithOrdersGraph();
}

// ✅ Fix 3: Batch loading
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)  // Load 10 users' orders in single query
    private Set<Order> orders;
}
```

### Dirty Checking & Flush

```java
@Transactional
public void demonstrateDirtyChecking() {
    User user = userRepo.findById(1L).orElse(null);
    user.setName("New Name");
    
    // No explicit save() call!
    // At @Transactional commit:
    // 1. Dirty checking: Hibernate compares object state to snapshot
    // 2. Flush: Auto-generates UPDATE SQL
    // 3. Commit: DB commits
}

// Manual flush
@Transactional
public void manualFlush() {
    User user = userRepo.findById(1L).orElse(null);
    user.setName("John");
    
    em.flush();  // Write to DB immediately (still in transaction)
    
    // Later...
    user.setEmail("john@example.com");
    // flush() again on commit
}
```

### HQL (Hibernate Query Language)

```java
@Repository
public class UserRepository extends JpaRepository<User, Long> {
    
    // Simple query
    @Query("FROM User u WHERE u.email = ?1")
    User findByEmail(String email);
    
    // With named parameter
    @Query("FROM User u WHERE u.email = :email AND u.age > :minAge")
    List<User> findAdultsByEmail(@Param("email") String email, @Param("minAge") int age);
    
    // Join
    @Query("SELECT u FROM User u JOIN u.orders o WHERE o.amount > :amount")
    List<User> findUsersWithOrdersOver(@Param("amount") BigDecimal amount);
    
    // Aggregation
    @Query("SELECT COUNT(u) FROM User u WHERE u.age > :age")
    long countAdults(@Param("age") int age);
    
    // Update (dangerous!)
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.loginCount = u.loginCount + 1 WHERE u.id = :id")
    void incrementLoginCount(@Param("id") Long id);
}
```

---

## MyBatis: SQL Mapper

**MyBatis** — Write SQL manually; full control over queries. Good for complex/legacy schemas.

### Setup

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

### Mapper Interface & XML

```java
@Mapper
public interface UserMapper {
    User selectById(Long id);
    List<User> selectAll();
    int insert(User user);
    int update(User user);
    int deleteById(Long id);
}
```

```xml
<!-- UserMapper.xml -->
<mapper namespace="com.example.mapper.UserMapper">
    
    <resultMap id="userResultMap" type="User">
        <id column="id" property="id"/>
        <result column="first_name" property="firstName"/>
        <result column="email" property="email"/>
        <result column="created_at" property="createdAt"/>
    </resultMap>
    
    <!-- SELECT -->
    <select id="selectById" parameterType="long" resultMap="userResultMap">
        SELECT id, first_name, email, created_at
        FROM users
        WHERE id = #{id}
    </select>
    
    <!-- SELECT multiple with dynamic where -->
    <select id="selectAll" resultMap="userResultMap">
        SELECT id, first_name, email, created_at
        FROM users
        <where>
            <if test="firstName != null">
                AND first_name LIKE CONCAT('%', #{firstName}, '%')
            </if>
            <if test="email != null">
                AND email = #{email}
            </if>
        </where>
        ORDER BY id
    </select>
    
    <!-- INSERT -->
    <insert id="insert" parameterType="User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO users (first_name, email, created_at)
        VALUES (#{firstName}, #{email}, NOW())
    </insert>
    
    <!-- UPSERT (MySQL) -->
    <insert id="upsert" parameterType="User">
        INSERT INTO users (id, first_name, email)
        VALUES (#{id}, #{firstName}, #{email})
        ON DUPLICATE KEY UPDATE
            first_name = #{firstName},
            email = #{email}
    </insert>
    
    <!-- UPDATE -->
    <update id="update" parameterType="User">
        UPDATE users
        <set>
            <if test="firstName != null">first_name = #{firstName},</if>
            <if test="email != null">email = #{email},</if>
            updated_at = NOW()
        </set>
        WHERE id = #{id}
    </update>
    
    <!-- DELETE -->
    <delete id="deleteById" parameterType="long">
        DELETE FROM users WHERE id = #{id}
    </delete>
</mapper>
```

### Annotation-Based Mapper

```java
@Mapper
public interface UserMapper {
    
    @Select("SELECT * FROM users WHERE id = #{id}")
    @Results({
        @Result(property = "id", column = "id", id = true),
        @Result(property = "firstName", column = "first_name"),
        @Result(property = "createdAt", column = "created_at")
    })
    User selectById(Long id);
    
    @Select("SELECT * FROM users WHERE email = #{email}")
    User selectByEmail(String email);
    
    @Insert("INSERT INTO users (first_name, email) VALUES (#{firstName}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(User user);
    
    @Update("UPDATE users SET first_name = #{firstName} WHERE id = #{id}")
    int update(User user);
    
    @Delete("DELETE FROM users WHERE id = #{id}")
    int deleteById(Long id);
}
```

### Complex Queries with Joins

```xml
<mapper namespace="com.example.mapper.UserMapper">
    
    <resultMap id="userWithOrdersMap" type="User">
        <id column="user_id" property="id"/>
        <result column="first_name" property="firstName"/>
        <collection property="orders" ofType="Order">
            <id column="order_id" property="id"/>
            <result column="amount" property="amount"/>
            <result column="order_date" property="createdAt"/>
        </collection>
    </resultMap>
    
    <select id="selectUserWithOrders" parameterType="long" resultMap="userWithOrdersMap">
        SELECT 
            u.id AS user_id,
            u.first_name,
            o.id AS order_id,
            o.amount,
            o.created_at AS order_date
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE u.id = #{id}
    </select>
</mapper>
```

### Dynamic SQL

```xml
<select id="searchUsers" parameterType="map" resultType="User">
    SELECT * FROM users
    <where>
        <if test="firstName != null">
            AND first_name LIKE CONCAT('%', #{firstName}, '%')
        </if>
        <if test="minAge != null">
            AND age >= #{minAge}
        </if>
        <if test="status != null">
            AND status = #{status}
        </if>
    </where>
    <choose>
        <when test="sortBy == 'name'">
            ORDER BY first_name
        </when>
        <when test="sortBy == 'created'">
            ORDER BY created_at DESC
        </when>
        <otherwise>
            ORDER BY id
        </otherwise>
    </choose>
    LIMIT #{limit} OFFSET #{offset}
</select>

<!-- Batch insert -->
<insert id="batchInsert" parameterType="list">
    INSERT INTO users (first_name, email) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.firstName}, #{user.email})
    </foreach>
</insert>
```

---

## JPA vs MyBatis Comparison

| Aspect | JPA/Hibernate | MyBatis |
|--------|---------------|---------|
| **Style** | ORM (object-centric) | SQL mapper (SQL-centric) |
| **Boilerplate** | Less; auto CRUD | More; write SQL for each query |
| **Control** | Less control over SQL | Full SQL control |
| **Relationships** | Built-in (@OneToMany, etc.) | Manual joins in XML/SQL |
| **Performance** | Good; caching, lazy loading | Very fast; pure SQL, no overhead |
| **Best for** | Standard CRUD, clean domains | Complex queries, legacy schemas |
| **Learning Curve** | Moderate (concepts: lazy, cascade, context) | Low (just SQL + mappings) |

### When to Use Each

**Use JPA/Hibernate for:**
- Domain-model-driven design
- Standard CRUD operations
- OOP-heavy codebase
- Need for automatic relationships

**Use MyBatis for:**
- Complex/tuned queries
- Legacy databases with odd schemas
- Bulk operations
- Team comfortable with SQL

---

## Spring Data JPA

Convenience layer over Hibernate with derived queries and custom methods.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived query (method name → SQL)
    User findByEmail(String email);
    List<User> findByFirstNameContainsIgnCase(String firstName);
    List<User> findByAgeGreaterThan(int age);
    
    // Custom JPQL
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    User findByEmailCustom(String email);
    
    // Native SQL
    @Query(value = "SELECT * FROM users WHERE age > ?1", nativeQuery = true)
    List<User> findAdultNative(int age);
    
    // Pageable
    Page<User> findByAgeGreaterThan(int age, Pageable pageable);
    
    // Sorting
    List<User> findAll(Sort.by("firstName"));
}
```

---

## Maintaining Entity IDs

### Best Practices

```java
@Entity
public class User {
    // ✅ DO: Use Long/UUID for primary keys
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // ❌ DON'T: Use String PKs without UUID
    // ❌ DON'T: Use business logic as PK (email, SSN, etc.)
    
    // Consider: Business key as unique index
    @Column(unique = true, nullable = false)
    private String email;  // Natural key, indexed
}

// Composite key (when necessary)
@Embeddable
public class OrderItemKey implements Serializable {
    @Column(name = "order_id")
    private Long orderId;
    
    @Column(name = "product_id")
    private Long productId;
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemKey id;
    
    @ManyToOne
    @MapsId("orderId")
    private Order order;
}
```

### ID Assignment Patterns

```java
// Strategy 1: Database-generated (IDENTITY) — recommended for most cases
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// Strategy 2: UUID — good for distributed systems, no central DB sequence
@Id
@GeneratedValue(generator = "uuid")
@GenericGenerator(name = "uuid", strategy = "uuid2")
private String id;

// Strategy 3: Custom ID generation
@Id
@GeneratedValue(strategy = GenerationType.AUTO, generator = "custom_id")
private String id;

// Strategy 4: Manual assignment (for imports/migrations)
@Id
private Long id;
// Set manually before persist
user.setId(1L);
em.persist(user);
```

### Handling ID After Insert

```java
@Transactional
public void createUserAndGetId() {
    User user = new User("John", "john@example.com");
    
    userRepository.save(user);  // ID generated and assigned
    
    System.out.println(user.getId());  // ID now available!
    
    // ID is available immediately after save() for IDENTITY strategy
}
```
