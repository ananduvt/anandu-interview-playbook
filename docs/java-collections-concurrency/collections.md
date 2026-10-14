# Collections Framework


**Collections Hierarchy Diagram**
(Note: Refer to Java documentation for visual collections hierarchy)

* Iterable(I)
* Collection(I)
  * List(I)
    * ArrayList(C)
    * LinkedList(C)
    * Vector(C)
      * Stack(C)

  * Queue(I)
    * PriorityQueue(C)
    * Dque(I)
      * ArrayDque(C)

  * Set(I)
    * HashSet(C)
    * LinkedHashSet(C)

    * SortedSet(I)
      * TreeSet(C)

  * Map(I)
    * HashMap(C)
    * HashTable(C)
    * LinkedHashMap(C)
    *
    * SortedMap(I)
      * TreeMap(C)

**Collections Utility Class**

The `java.util.Collections` class provides static utility methods for common operations on collections:

* **addAll(Collection<? super T> c, T... elements)** - Add all elements to collection
* **sort(List<T> list)** - Sort list in ascending order (natural ordering)
* **sort(List<T> list, Comparator<? super T> c)** - Sort with custom comparator
* **shuffle(List<?> list)** - Randomize order of elements
* **binarySearch(List<? extends Comparable<? super T>> list, T key)** - Search in sorted list
* **reverse(List<?> list)** - Reverse order
* **rotate(List<?> list, int distance)** - Rotate elements
* **fill(List<? super T> list, T obj)** - Fill all positions with same value
* **copy(List<? super T> dest, List<? extends T> src)** - Copy source to destination
* **min(Collection<? extends T> coll)** - Find minimum element
* **max(Collection<? extends T> coll)** - Find maximum element
* **disjoint(Collection<?> c1, Collection<?> c2)** - Check if collections have no common elements
* **frequency(Collection<?> c, Object o)** - Count occurrences of element
* **replaceAll(List<T> list, T oldVal, T newVal)** - Replace all occurrences

**Collections.sort() Algorithm:** Uses merge sort (stable, O(n log n) guaranteed)

## ArrayList

[https://www.baeldung.com/java-list-capacity-array-size](https://www.baeldung.com/java-list-capacity-array-size)

When an ArrayList is created in Java, it is backed by an array. Initially, this array has a default capacity, often 10. When elements are added to the ArrayList, they are stored in this underlying array. If the ArrayList grows beyond its current capacity, a new, larger array is created, typically with a size that is 1.5 or 2 times the original size. The elements from the old array are then copied to the new array, and the old array is discarded, a process known as resizing. This ensures that ArrayList can accommodate a dynamic number of elements.

The memory occupied by an ArrayList includes the object header, the size of the internal array, and the memory used by the elements themselves. The header typically takes 12 bytes, and the array size depends on the number of elements and the data type. For example, an ArrayList of integers with 10 elements would require 40 bytes for the elements themselves (4 bytes per integer). When the ArrayList is resized, the memory usage increases to accommodate the new capacity.

ArrayList stores elements contiguously in memory, which allows for efficient access using indices. However, frequent resizing can lead to performance overhead due to the copying of elements. To mitigate this, it is often beneficial to initialize an ArrayList with an appropriate initial capacity if the number of elements is known in advance.

## Hashmap

[https://anmolsehgal.medium.com/java-hashmap-internal-implementation-21597e1efec3](https://anmolsehgal.medium.com/java-hashmap-internal-implementation-21597e1efec3)
[https://www.geeksforgeeks.org/internal-working-of-hashmap-java/](https://www.geeksforgeeks.org/internal-working-of-hashmap-java/)

In Java hashing, several key concepts govern the performance and behavior of hash-based data structures like HashMap. These concepts include capacity, threshold, rehashing, and collision.

**Capacity**:
The initial size of the internal array (bucket array) in a HashMap. The default capacity is typically 16, but it can be specified during HashMap instantiation.

**Threshold**:
A value that triggers the rehashing process. It's calculated as the product of the capacity and the load factor (default 0.75). When the number of entries in the HashMap exceeds the threshold, the HashMap resizes. For example, with a capacity of 16 and a load factor of 0.75, the threshold is 12.

**Rehashing**:
The process of resizing the internal array of a HashMap when the number of entries exceeds the threshold. It involves creating a new array with double the capacity, recalculating the hash codes for existing entries, and redistributing them into the new array. This ensures efficient retrieval times as the HashMap grows.

**Collision**:
Occurs when two or more keys produce the same hash code, resulting in them mapping to the same index in the internal array. HashMap handles collisions using techniques like separate chaining (using linked lists or trees) to store multiple entries at the same index.

### Load Factor

[https://www.baeldung.com/java-hashmap-load-factor#:\~:text=The%20load%20factor%20is%20the,code%20of%20already%20stored%20entries](https://www.baeldung.com/java-hashmap-load-factor#:~:text=The%20load%20factor%20is%20the,code%20of%20already%20stored%20entries).

In Java, the load factor is a crucial parameter associated with hash-based data structures like HashMap and HashSet. It determines when the data structure's internal capacity should be increased (rehashing). The load factor is a number between 0 and 1 (inclusive), with a default value of 0.75 in Java.

It represents the ratio of the number of elements stored in the hash table to the total number of buckets (capacity). When the load factor is reached, the capacity of the hash table is doubled, and all existing elements are rehashed into the new buckets. This process helps maintain the efficiency of the hash table's operations, such as insertion, deletion, and retrieval.

A lower load factor means that the hash table will be resized more frequently, leading to less collision and faster access times but also higher memory consumption. Conversely, a higher load factor means that the hash table will be resized less frequently, saving memory but potentially increasing collision and slowing down operations. The default value of 0.75 is generally considered a good balance between time and space efficiency.

**List**

* The List is an indexed sequence. - maintain order
* List allows duplicate elements
* Elements by their position can be accessed
* Multiple null elements can be stored.
* Implementations are ArrayList, LinkedList, Vector, Stack

**Set**

* The Set is an non-indexed sequence. - does not maintain order
* Set doesn’t allow duplicate elements.
* Position access to elements is not allowed.
* Null element can store only once.
* Implementations are HashSet, LinkedHashSet.

**HashMap - Key Characteristics**

| Property | Detail |
|----------|--------|
| **Hashing** | Uses hash codes to map keys to bucket positions |
| **Collisions** | Handled via chaining (linked list) or tree (since Java 8) |
| **Bucket Conversion** | When chain length > 8, converts to Red-Black tree for O(log n) |
| **Thread Safety** | Not synchronized (use ConcurrentHashMap or Collections.synchronizedMap) |
| **Null Keys** | Allows one null key |
| **Null Values** | Allows multiple null values |
| **Iteration Order** | No guaranteed order (insertion order: LinkedHashMap; sorted: TreeMap) |
| **Performance** | O(1) average case, O(n) worst case (hash collision) |

**HashMap vs ConcurrentHashMap vs Hashtable:**
- **HashMap:** Fast, not thread-safe, preferred for single-threaded
- **ConcurrentHashMap:** Thread-safe with segment-level locking, best for multi-threaded
- **Hashtable:** Legacy, fully synchronized (slow), avoid in modern code

## ConcurrentHashMap: Thread-Safe HashMap

**ConcurrentHashMap** provides thread-safe operations without locking the entire map. Uses segment-level locking (or bucket-level in Java 8+).

### Internal Structure (Java 8+)

```
ConcurrentHashMap segments the map into buckets:

HashMap (single lock - blocks entire map):
[===LOCK===]
[Bucket 0] [Bucket 1] [Bucket 2] [Bucket 3] [Bucket 4]

ConcurrentHashMap (locks per bucket - allows concurrent access):
[LOCK A] [LOCK B] [LOCK C] [LOCK D] [LOCK E]
Bucket 0  Bucket 1  Bucket 2  Bucket 3  Bucket 4

Multiple threads can read/write different buckets simultaneously!
```

### Basic Usage

```java
// Create ConcurrentHashMap
Map<String, Integer> map = new ConcurrentHashMap<>(16, 0.75f, 4);
// Capacity: 16, Load Factor: 0.75, Concurrency Level: 4

map.put("key1", 100);
map.put("key2", 200);

Integer value = map.get("key1");  // 100

map.remove("key2");
map.replace("key1", 150);

// Iteration is safe (doesn't require external synchronization)
for (String key : map.keySet()) {
    System.out.println(key + " = " + map.get(key));
}
```

### Atomic Operations

```java
// putIfAbsent: Set only if key doesn't exist
map.putIfAbsent("counter", 0);  // Returns 0

// replace: Update if value matches
map.replace("key1", 100, 200);  // Update 100 to 200 if key1 == 100

// getOrDefault: Get with fallback
Integer count = map.getOrDefault("missing", -1);  // Returns -1

// computeIfPresent: Conditional update
map.computeIfPresent("key1", (k, v) -> v + 10);  // Add 10 to existing

// computeIfAbsent: Conditional insert
map.computeIfAbsent("key2", k -> 999);  // Set to 999 if absent
```

### Practical: Thread-Safe Counter

```java
// ❌ Bad: Not thread-safe
Map<String, Integer> badCounter = new HashMap<>();
badCounter.put("count", 0);

// Multiple threads increment (race condition!)
for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        Integer val = badCounter.get("count");
        badCounter.put("count", val + 1);  // NOT atomic!
    }).start();
}

// Final value may be < 1000 due to race condition

// ✅ Good: ConcurrentHashMap
Map<String, Integer> safeCounter = new ConcurrentHashMap<>();
safeCounter.put("count", 0);

for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        safeCounter.compute("count", (k, v) -> (v == null ? 0 : v) + 1);
    }).start();
}

// Final value will be exactly 1000 (atomic operation)
```

### Performance Comparison

```java
// Benchmark: 4 threads, 10k operations each

// HashMap + synchronized (serialize all access)
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
// Time: ~500ms (bottleneck: single lock)

// ConcurrentHashMap (parallel access)
Map<String, Integer> concMap = new ConcurrentHashMap<>();
// Time: ~100ms (4× faster - less contention)

// Under high contention:
// - HashMap: All threads wait for same lock (serial)
// - ConcurrentHashMap: Threads access different buckets (parallel)
```

### ConcurrentHashMap vs Synchronized HashMap

| Aspect | SynchronizedMap | ConcurrentHashMap |
|--------|-----------------|-------------------|
| **Locking** | Single lock on entire map | Segment/bucket-level locks |
| **Concurrency** | Only 1 thread at a time | Multiple threads in parallel |
| **Performance** | Slower under contention | Fast under contention |
| **Iteration** | Requires external sync | Safe (weakly consistent) |
| **null keys/values** | Allows | ❌ Rejects |
| **Best for** | Rare writes, frequent reads (low contention) | Frequent concurrent access |

### Iteration (Weakly Consistent)

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Safe to iterate while others modify
// (May see partial updates, but won't throw ConcurrentModificationException)
new Thread(() -> {
    map.put("key1", 1);
    map.put("key2", 2);
    map.put("key3", 3);
}).start();

Thread.sleep(100);

for (String key : map.keySet()) {
    System.out.println(key);  // May see 1, 2, or 3 keys (not synchronized)
}
```

### Example: Cache with ConcurrentHashMap

```java
public class Cache<K, V> {
    private final Map<K, V> data = new ConcurrentHashMap<>();
    private final Map<K, Long> timestamps = new ConcurrentHashMap<>();
    private final long ttlMillis;
    
    public Cache(long ttlMillis) {
        this.ttlMillis = ttlMillis;
    }
    
    public void put(K key, V value) {
        data.put(key, value);
        timestamps.put(key, System.currentTimeMillis());
    }
    
    public V get(K key) {
        Long timestamp = timestamps.get(key);
        if (timestamp == null) return null;
        
        // Check if expired
        if (System.currentTimeMillis() - timestamp > ttlMillis) {
            data.remove(key);
            timestamps.remove(key);
            return null;
        }
        
        return data.get(key);
    }
    
    public int size() {
        return data.size();
    }
}

// Usage: Multiple threads can safely access concurrently
Cache<String, User> userCache = new Cache<>(3600000);  // 1 hour TTL

for (int i = 0; i < 100; i++) {
    final int userId = i;
    new Thread(() -> {
        userCache.put("user:" + userId, new User(userId, "User" + userId));
        User user = userCache.get("user:" + userId);
    }).start();
}
```

**List vs Set - Comparison**

| Feature | List | Set |
|---------|------|-----|
| **Order** | ✅ Maintains insertion order | ❌ No order guarantee (except LinkedHashSet, TreeSet) |
| **Duplicates** | ✅ Allows duplicates | ❌ No duplicates |
| **Indexing** | ✅ Access by index (get, set) | ❌ No index-based access |
| **Null** | ✅ Multiple nulls allowed | ⚠️ One null (only HashSet/LinkedHashSet) |
| **Common Impls** | ArrayList, LinkedList, Vector | HashSet, LinkedHashSet, TreeSet |
| **Use Case** | Maintaining sequence | Unique values, membership checking |
| **Performance** | ArrayList: O(1) access, O(n) insert | HashSet: O(1) average, O(n) worst |

**When to use:**
- **List:** You need sequence order or indexed access (users in order, student records)
- **Set:** You need uniqueness and fast lookup (unique emails, tags, categories)

Stack and queue

Hash Collision
Binary tree - use cases
Heap DS

clob , blob
Hashing
tokenization
