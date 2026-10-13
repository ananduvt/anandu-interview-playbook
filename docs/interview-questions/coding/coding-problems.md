# Coding Problems & Solutions

Common coding interview problems with solutions and explanations.

---

## Print 1..N without a loop (Recursion)

**Problem:** Print numbers from 1 to N using recursion instead of loops.

```java
static void printNos(int n) {
    if (n > 0) {
        printNos(n - 1);
        System.out.print(n + " ");
    }
}
// printNos(100);  → Output: 1 2 3 ... 100
```

**Approach:**
- Base case: n ≤ 0 (stop recursion)
- Recursive case: Call printNos(n-1), then print n
- Result: Prints after recursive call (post-order), giving ascending order

**Complexity:** O(n) time, O(n) space (call stack)

---

## Palindrome Check without Loops

**Problem:** Check if a string is a palindrome using recursion or functional approach.

### Recursion Approach

```java
public boolean isPalindromeRecursive(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    return recursivePalindrome(clean, 0, clean.length() - 1);
}

private boolean recursivePalindrome(String s, int fwd, int bwd) {
    if (fwd >= bwd) return true;  // Base case: reached middle
    if (s.charAt(fwd) != s.charAt(bwd)) return false;  // Mismatch
    return recursivePalindrome(s, fwd + 1, bwd - 1);  // Recurse
}
```

**Approach:**
- Clean: Remove spaces, convert to lowercase
- Two pointers: fwd from start, bwd from end
- Base case: fwd >= bwd (checked all pairs)
- Mismatch: Return false if characters differ
- Recurse: Move pointers inward

**Complexity:** O(n) time, O(n) space (call stack for recursion)

### Stream API Approach

```java
public boolean isPalindrome(String text) {
    String t = text.replaceAll("\\s+", "").toLowerCase();
    return IntStream.range(0, t.length() / 2)
        .noneMatch(i -> t.charAt(i) != t.charAt(t.length() - i - 1));
}
```

**Approach:**
- Range: Loop from 0 to length/2
- noneMatch: Return true if NO index has mismatching characters
- Concise functional style, no explicit recursion

**Complexity:** O(n) time, O(1) space (no recursion)

---

## Max Subarray Sum (Kadane's Algorithm)

**Problem:** Find maximum sum of a contiguous subarray.

**Example:** [-2, 1, -3, 4, -1, 2, 1, -5, 4] → 6 (subarray: [4, -1, 2, 1])

```java
public static int maxSubArray(int[] nums) {
    int maxSum = Integer.MIN_VALUE;
    int curr = 0;
    
    for (int n : nums) {
        curr += n;                           // Include current element
        maxSum = Math.max(maxSum, curr);    // Track max so far
        if (curr < 0) curr = 0;            // Reset if cumulative sum negative
    }
    
    return maxSum;
}
```

**Approach:**
1. **Track two values:**
   - `maxSum`: Best sum seen so far
   - `curr`: Current cumulative sum
2. **For each element:**
   - Add to current sum
   - Update maxSum if current > maxSum
   - Reset current to 0 if negative (no point carrying negative forward)
3. **Return maxSum**

**Why it works:**
- If cumulative sum becomes negative, it's better to start fresh with the next element
- At each step, we know the best sum ending at current position
- Dynamic programming: optimal substructure

**Complexity:** O(n) time, O(1) space

---

## Key Insights

### Recursion Benefits
- Natural for problems like palindromes, tree traversal
- Can replace loops for linear processing
- Trade-off: Space (call stack) for readability

### Functional Approaches
- Streams, lambdas reduce loops
- More declarative (what, not how)
- Can be harder to debug

### Algorithm Efficiency
- Kadane's algorithm: O(n) instead of brute force O(n²)
- Look for: Can we reduce passes? Can we use state efficiently?

---

## Backlog

See [DSA Backlog](../../dsa/coding-problems.md) for additional problems:
- Sort students by mark + rank assignment
- List ↔ Array ↔ Stream conversions
- Integer ↔ String conversions
- Bitwise operations & tricks

---

[More problems to be added progressively]

**See also:** [Coding Questions Overview](index.md), [DSA Algorithms](../../dsa/algorithms.md), [Backlog](../../backlog.md)
