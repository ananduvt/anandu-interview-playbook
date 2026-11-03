# Algorithms

## Complexity
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!). State **time and space** for every solution.

| Complexity | Typical source |
|------------|----------------|
| O(1) | Constant: array index, hash lookup |
| O(log n) | Binary search, balanced tree operations |
| O(n) | Linear scan |
| O(n log n) | Merge sort, quicksort (avg), heap sort |
| O(n²) | Nested loops, bubble/insertion sort |
| O(n³) | Triple nested loops |
| O(2ⁿ) | Subsets, naive recursion, backtracking |
| O(n!) | Permutations |

**Space:** O(1) = no extra space · O(log n) = recursion depth · O(n) = extra array or hash map · O(n²) = 2D array.

## Sorting
| Algorithm | Time (avg) | Space | Stable |
|-----------|-----------|-------|--------|
| Quicksort | O(n log n) | O(log n) | no |
| Mergesort | O(n log n) | O(n) | yes |
| Heapsort | O(n log n) | O(1) | no |
| Insertion | O(n²) | O(1) | yes |
Java `Arrays.sort` = dual-pivot quicksort (primitives) / Timsort (objects, stable).

## Searching
- **Binary search** — sorted input, O(log n). Variants: first/last occurrence, rotated array, search on answer space.

## Core patterns
| Technique | Use case | Example |
|-----------|----------|---------|
| **Two pointers** | Pairs in sorted arrays, palindrome, dedup, container | Reverse string, merge sorted arrays |
| **Sliding window** | Substring or subarray, fixed/variable window | Longest substring without repeats, max sum of size k |
| **Binary search** | Sorted input, search on answer space | Find target in rotated array |
| **Hash map** | Counting, storing relationships | Group anagrams, find pairs with sum |
| **Heap** | K largest/smallest, min/max tracking | Top K frequent elements |
| **Stack** | LIFO, matching, ordering | Valid parentheses, next greater element |
| **BFS/DFS** | Trees/graphs, connected components; BFS = shortest path (unweighted); topological sort for DAGs | Number of islands, word ladder |
| **Dynamic programming** | Overlapping subproblems + optimal substructure (memoize / tabulate) | Fibonacci, coin change, longest subsequence |
| **Recursion / backtracking** | Subsets, permutations, combinations, constraints | N-Queens, word search |
| **Greedy** | Locally optimal choice; prove correctness | Activity selection, Huffman coding |

## Must-know examples
- **Kadane's algorithm** (max subarray sum) — see [Coding Problems](coding-problems.md).
- Dijkstra (shortest path, weighted), Union-Find (connectivity), Floyd's cycle detection.

## Interview process
Clarify → example → brute force + Big-O → optimize → clean code → test edges → state complexity.

**Practice questions:** [Java Collections and Streams](../interview-questions/verbal/verbal-java-collections-streams-questions.md)
