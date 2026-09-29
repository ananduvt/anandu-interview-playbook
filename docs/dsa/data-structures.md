# Data Structures

## Big-O of common operations
| Structure | Access | Search | Insert | Delete |
|-----------|--------|--------|--------|--------|
| Array | O(1) | O(n) | O(n) | O(n) |
| Dynamic array (ArrayList) | O(1) | O(n) | O(1)* amortized | O(n) |
| Linked list | O(n) | O(n) | O(1) | O(1) |
| Stack / Queue | O(n) | O(n) | O(1) | O(1) |
| Hash table | — | O(1) avg | O(1) avg | O(1) avg |
| Balanced BST / TreeMap | O(log n) | O(log n) | O(log n) | O(log n) |
| Heap (priority queue) | — | O(n) | O(log n) | O(log n) (min/max O(1) peek) |

## Linear structures
- **Array** — contiguous, O(1) index. Fixed size.
- **Linked list** — nodes with pointers; O(1) insert/delete at known position; no random access.
- **Stack** — LIFO (`ArrayDeque`). Undo, DFS, expression eval.
- **Queue** — FIFO (`ArrayDeque`, `LinkedList`); **Deque** = double-ended; **PriorityQueue** = heap-ordered.

## Hashing
- **HashMap** — buckets via `hashCode()`; O(1) avg; collisions via chaining (treeify at 8 in Java);
  resize when size > capacity × load factor (0.75). See [Collections](../java/collections.md).
- **HashSet** — backed by a HashMap.

## Trees & graphs
- **Binary tree / BST** — ordered; balanced (AVL, Red-Black) keeps O(log n). `TreeMap`/`TreeSet` = Red-Black.
- **Heap** — complete binary tree; min/max at root; `PriorityQueue`.
- **Trie** — prefix tree for strings/autocomplete.
- **Graph** — vertices + edges; adjacency list (sparse) vs matrix (dense); BFS/DFS traversal.

## Choosing
- Need order + range queries → TreeMap. Fast lookup → HashMap. FIFO → Queue. Top-k → Heap. Prefixes → Trie.
