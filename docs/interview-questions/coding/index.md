# Coding Interview Questions

Programming problems: algorithms, data structures, implementation, optimization.

---

## Question Categories

### Arrays & Strings
- Two pointers, sliding window, prefix sums
- String manipulation, pattern matching

### Linked Lists
- Pointer manipulation, cycle detection, reversal

### Trees & Graphs
- BFS, DFS, binary search trees, tree traversal
- Graph algorithms, shortest path, topological sort

### Dynamic Programming
- Overlapping subproblems, optimal substructure
- Memoization vs tabulation

### Stacks & Queues
- Expression evaluation, monotonic stack
- Queue operations, BFS/topological sort

### Sorting & Searching
- Merge sort, quick sort, binary search
- Counting sort, radix sort

### Math & Logic
- Number theory, combinatorics
- Bit manipulation, probability

### Hash Tables & Sets
- Frequency counting, collision handling
- Finding duplicates, unique elements

---

## How to Approach Coding Questions

### Step 1: Understand
- Read problem carefully
- Identify inputs, outputs, constraints
- Ask clarifying questions (time limit, duplicate handling, negative numbers?)
- Work through examples

### Step 2: Brute Force
- Simplest solution first
- Don't worry about optimization initially
- State time/space complexity

### Step 3: Optimize
- Can we use data structures? (hash, heap, tree)
- Can we reduce passes? (two-pointer, sliding window)
- Trade-offs? (memory for speed, vice versa)

### Step 4: Code
- Write clean, readable code
- Use meaningful variable names
- Add comments for complex logic
- Follow language conventions

### Step 5: Test
- Trace through examples
- Check edge cases: empty, single element, negatives, duplicates, very large
- Verify complexity

### Step 6: Review
- Can this be simplified?
- Are there alternative approaches?
- Any bugs or off-by-one errors?

---

## Complexity Cheat Sheet

### Time Complexity (Best to Worst)
- O(1) - Constant
- O(log n) - Binary search
- O(n) - Linear scan
- O(n log n) - Merge sort, quick sort, heap operations
- O(n²) - Nested loops, bubble sort
- O(n³) - Triple nested loops
- O(2ⁿ) - Exponential (subsets, backtracking)
- O(n!) - Factorial (permutations)

### Space Complexity
- O(1) - No extra space
- O(log n) - Recursion depth
- O(n) - Extra array, hash map
- O(n²) - 2D array

---

## Common Techniques

| Technique | Use Case | Example |
|-----------|----------|---------|
| **Two Pointers** | Sorted array, palindrome, container | Reverse string, merge sorted arrays |
| **Sliding Window** | Substring, subarray, fixed/variable window | Longest substring without repeats |
| **Binary Search** | Sorted, searching, finding position | Find target in rotated array |
| **Hash Map** | Counting, storing relationships | Group anagrams, find pairs with sum |
| **Heap** | K largest/smallest, min/max tracking | Top K frequent elements |
| **Stack** | LIFO, matching, ordering | Valid parentheses, next greater element |
| **DFS/BFS** | Graph, tree, connected components | Number of islands, word ladder |
| **DP** | Overlapping subproblems, optimization | Fibonacci, coin change, longest subsequence |
| **Backtracking** | Combinations, permutations, constraints | N-Queens, word search |
| **Greedy** | Optimal local choice leads to global | Activity selection, huffman coding |

---

## Interview Tips

✅ **Do's**
- Think aloud; explain your reasoning
- Clarify ambiguous requirements
- Start with brute force, then optimize
- Discuss complexity before/after optimizing
- Test on examples and edge cases
- Code neatly; use clear variable names

❌ **Don'ts**
- Jump to coding without understanding
- Over-complicate initially
- Ignore edge cases
- Leave code with bugs
- Forget to state complexity

---

## Example: Two Sum

**Problem:** Given array of integers and target, find two distinct indices whose values sum to target.

**Approach:**
```
Brute Force: 
  O(n²) - Check every pair
  
Optimal:
  O(n) - Use hash map
  Pass 1: Store (value → index)
  Pass 2: For each num, check if (target - num) exists in map
```

**Code:**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    return new int[] {}; // Not found
}
```

**Complexity:** O(n) time, O(n) space

---

## Resources for Practice

- LeetCode, HackerRank, CodeSignal
- Medium to Hard difficulty
- Category-wise practice (Arrays → Linked Lists → Trees, etc.)
- Time yourself (typical: 30-45 minutes per problem)

---

## Problem Collections

### [Coding Problems & Solutions](coding-problems.md)
Real interview problems with explanations:
- Print 1..N without loops (recursion)
- Palindrome checking (recursion vs streams)
- Maximum subarray sum (Kadane's algorithm)
- [More to be added]

### [Stream API Problems](coding-stream-api-problems.md)
Modern functional Java with Stream API:
- Convert to uppercase and sort
- Filter and collect with conditions
- Count occurrences (groupingBy)
- Reduce operations (sum, product, concatenation)
- Combined operations (filter, map, collect)

---

[Additional questions to be added progressively]

**See also:** [Interview Questions Overview](../index.md), [DSA Coding Problems](../../dsa/coding-problems.md)
