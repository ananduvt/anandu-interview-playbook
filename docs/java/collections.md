# Collections Framework

## Hierarchy
- **Collection** → List, Set, Queue. **Map** is separate.
- **List** — ordered, duplicates: `ArrayList`, `LinkedList`.
- **Set** — no duplicates: `HashSet`, `LinkedHashSet`, `TreeSet` (sorted).
- **Queue/Deque** — `ArrayDeque`, `PriorityQueue`.
- **Map** — key→value: `HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap`.

## ArrayList vs LinkedList
| | ArrayList | LinkedList |
|--|-----------|------------|
| Backing | dynamic array | doubly linked nodes |
| Random access | O(1) | O(n) |
| Insert/delete middle | O(n) | O(1) at known node |
| Default | **preferred** most cases | rare (deque/queue) |

`ArrayList` grows by ~1.5× when capacity is exceeded (copy to a new array).

## HashMap internals
- Array of buckets; index = `hash(key) & (n-1)`. Collisions → linked list, **treeify to a red-black tree at 8**
  entries in a bucket (Java 8+).
- **Load factor** default **0.75**; resize (double capacity, rehash) when `size > capacity × loadFactor`.
- `null` key allowed (one); `equals` + `hashCode` contract is essential.

## Thread-safe options
- **`ConcurrentHashMap`** — concurrent reads, bucket/bin-level locking on writes; no null keys/values. Preferred.
- `Collections.synchronizedMap` — one lock for the whole map (coarse).
- `Hashtable`/`Vector` — legacy, fully synchronized.

## Comparable vs Comparator
- **Comparable** — natural ordering, `compareTo` on the class.
- **Comparator** — external/multiple orderings: `Comparator.comparing(...).thenComparing(...).reversed()`.

## fail-fast vs fail-safe
Most collections are **fail-fast** (throw `ConcurrentModificationException`); concurrent collections are **fail-safe**.
