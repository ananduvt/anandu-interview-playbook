# Algorithms

## Complexity
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!). State **time and space** for every solution.

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
- **Two pointers** — pairs in sorted arrays, palindrome, dedup.
- **Sliding window** — longest substring, max sum of size k.
- **Recursion / backtracking** — subsets, permutations, combinations, N-queens.
- **BFS/DFS** — trees/graphs; BFS = shortest path (unweighted); topological sort for DAGs.
- **Dynamic programming** — overlapping subproblems + optimal substructure (memoize / tabulate).
- **Greedy** — locally optimal choice (intervals, Huffman); prove correctness.

## Must-know examples
- **Kadane's algorithm** (max subarray sum) — see [Coding Problems](coding-problems.md).
- Dijkstra (shortest path, weighted), Union-Find (connectivity), Floyd's cycle detection.

## Interview process
Clarify → example → brute force + Big-O → optimize → clean code → test edges → state complexity.
