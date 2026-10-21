# Coding Problems & Solutions

Common coding interview problems with solutions and explanations.

---

## 1. Print 1..N without a loop (Recursion)

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Print numbers from 1 to N using recursion instead of loops.</p>

<h3>Approach 1: Recursive (Post-order)</h3>

```java
static void printNos(int n) {
    if (n > 0) {
        printNos(n - 1);
        System.out.print(n + " ");
    }
}
// printNos(5);  → Output: 1 2 3 4 5
```

<p><strong>How it works:</strong> Recurse first, then print. This gives ascending order.</p>

<h3>Approach 2: Recursive (Pre-order)</h3>

```java
static void printNosPreOrder(int n) {
    if (n > 0) {
        System.out.print(n + " ");
        printNosPreOrder(n - 1);
    }
}
// printNosPreOrder(5);  → Output: 5 4 3 2 1
```

<p><strong>How it works:</strong> Print first, then recurse. This gives descending order.</p>

<h3>Approach 3: Stream API</h3>

```java
static void printNosStream(int n) {
    IntStream.rangeClosed(1, n).forEach(i -> System.out.print(i + " "));
}
// printNosStream(5);  → Output: 1 2 3 4 5
```

<p><strong>How it works:</strong> Functional approach using streams (no explicit recursion).</p>

<h4>Complexity (All Approaches)</h4>
<p>O(n) time, O(n) space (call stack for recursion)</p>

<h4>Key Insight: Recursion Benefits</h4>
<ul>
<li>Natural for problems like palindromes, tree traversal</li>
<li>Can replace loops for linear processing</li>
<li>Trade-off: Space (call stack) for readability</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Recursion & Algorithms</a></p>

</details>

---

## 2. Check if a string is a palindrome

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Check if a string is a palindrome using recursion or functional approach.</p>

<h3>Recursion Approach</h3>

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

<h4>Approach</h4>
<ul>
<li>Clean: Remove spaces, convert to lowercase</li>
<li>Two pointers: fwd from start, bwd from end</li>
<li>Base case: fwd >= bwd (checked all pairs)</li>
<li>Mismatch: Return false if characters differ</li>
<li>Recurse: Move pointers inward</li>
</ul>

<h4>Complexity</h4>
<p>O(n) time, O(n) space (call stack for recursion)</p>

<h3>Stream API Approach</h3>

```java
public boolean isPalindrome(String text) {
    String t = text.replaceAll("\\s+", "").toLowerCase();
    return IntStream.range(0, t.length() / 2)
        .noneMatch(i -> t.charAt(i) != t.charAt(t.length() - i - 1));
}
```

<h4>Approach</h4>
<ul>
<li>Range: Loop from 0 to length/2</li>
<li>noneMatch: Return true if NO index has mismatching characters</li>
<li>Concise functional style, no explicit recursion</li>
</ul>

<h3>Approach 3: Two-Pointer Iterative</h3>

```java
public boolean isPalindromeIterative(String text) {
    String t = text.replaceAll("\\s+", "").toLowerCase();
    int left = 0, right = t.length() - 1;
    
    while (left < right) {
        if (t.charAt(left) != t.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

<p><strong>How it works:</strong> Two pointers move inward, no recursion or streams.</p>

<h3>Approach 4: StringBuilder Reverse</h3>

```java
public boolean isPalindromeReverse(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    String reversed = new StringBuilder(clean).reverse().toString();
    return clean.equals(reversed);
}
```

<p><strong>How it works:</strong> Reverse the string and compare. Simple but uses extra space.</p>

<h4>Complexity Comparison</h4>
<ul>
<li><strong>Recursive:</strong> O(n) time, O(n) space (call stack)</li>
<li><strong>Stream:</strong> O(n) time, O(1) space</li>
<li><strong>Two-pointer:</strong> O(n) time, O(1) space</li>
<li><strong>StringBuilder:</strong> O(n) time, O(n) space (new string)</li>
</ul>

<h4>Key Insight: Functional Approaches</h4>
<ul>
<li>Streams, lambdas reduce loops and improve readability</li>
<li>More declarative (what, not how)</li>
<li>Can be harder to debug but often more concise</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings</a></p>

</details>

---

## 3. Find maximum sum of a contiguous subarray (Kadane's Algorithm)

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Find maximum sum of a contiguous subarray.</p>

<p><strong>Example:</strong> [-2, 1, -3, 4, -1, 2, 1, -5, 4] → 6 (subarray: [4, -1, 2, 1])</p>

<h3>Approach 1: Kadane's Algorithm (Optimal)</h3>

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

<p><strong>Why it works:</strong> If cumulative sum becomes negative, start fresh. Dynamic programming: optimal substructure.</p>

<h3>Approach 2: Brute Force (All Subarrays)</h3>

```java
public static int maxSubArrayBruteForce(int[] nums) {
    int maxSum = Integer.MIN_VALUE;
    
    for (int i = 0; i < nums.length; i++) {
        int sum = 0;
        for (int j = i; j < nums.length; j++) {
            sum += nums[j];
            maxSum = Math.max(maxSum, sum);
        }
    }
    
    return maxSum;
}
```

<p><strong>How it works:</strong> Try all possible subarrays and track maximum. Simple but inefficient.</p>

<h3>Approach 3: Divide and Conquer</h3>

```java
public static int maxSubArrayDC(int[] nums, int left, int right) {
    if (left == right) return nums[left];
    
    int mid = left + (right - left) / 2;
    int leftMax = maxSubArrayDC(nums, left, mid);
    int rightMax = maxSubArrayDC(nums, mid + 1, right);
    int crossMax = maxCrossingSum(nums, left, mid, right);
    
    return Math.max(crossMax, Math.max(leftMax, rightMax));
}

private static int maxCrossingSum(int[] nums, int left, int mid, int right) {
    int leftSum = Integer.MIN_VALUE, sum = 0;
    for (int i = mid; i >= left; i--) {
        sum += nums[i];
        leftSum = Math.max(leftSum, sum);
    }
    
    int rightSum = Integer.MIN_VALUE;
    sum = 0;
    for (int i = mid + 1; i <= right; i++) {
        sum += nums[i];
        rightSum = Math.max(rightSum, sum);
    }
    
    return leftSum + rightSum;
}
```

<p><strong>How it works:</strong> Divide array in half, find max in left, right, and crossing subarrays.</p>

<h4>Complexity Comparison</h4>
<ul>
<li><strong>Kadane's:</strong> O(n) time, O(1) space ⭐ Best</li>
<li><strong>Brute Force:</strong> O(n²) time, O(1) space</li>
<li><strong>Divide & Conquer:</strong> O(n log n) time, O(log n) space (recursion stack)</li>
</ul>

<h4>Key Insight: Algorithm Efficiency</h4>
<ul>
<li>Kadane's algorithm: O(n) instead of brute force O(n²)</li>
<li>Key optimization: Can we reduce passes? Can we use state efficiently?</li>
<li>Dynamic programming: Optimal substructure - best solution built from smaller subproblems</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Dynamic Programming</a></p>

</details>

---

## 4. Remove duplicates from a list

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>How would you remove duplicates from a list while maintaining order?</p>

<h3>Approach 1: Stream + HashSet Filter</h3>

```java
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 3);
Set<Integer> seen = new HashSet<>();
List<Integer> result = list.stream().filter(seen::add).collect(Collectors.toList());
// Result: [1, 2, 3]
```

<p><strong>Key insight:</strong> <code>seen.add()</code> returns false if element already exists, so <code>filter()</code> keeps only new elements.</p>

<h3>Approach 2: LinkedHashSet (Maintains Order)</h3>

```java
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 3);
List<Integer> result = new ArrayList<>(new LinkedHashSet<>(list));
// Result: [1, 2, 3]
```

<p><strong>How it works:</strong> LinkedHashSet maintains insertion order while removing duplicates. Simplest approach.</p>

<h3>Approach 3: Stream + toSet (No Order Guarantee)</h3>

```java
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 3);
Set<Integer> result = list.stream().collect(Collectors.toSet());
// Result: {1, 2, 3} - order not guaranteed
```

<p><strong>How it works:</strong> Use Collectors.toSet() if order doesn't matter (faster).</p>

<h3>Approach 4: Manual Iteration with Set</h3>

```java
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 3);
Set<Integer> seen = new HashSet<>();
List<Integer> result = new ArrayList<>();

for (Integer num : list) {
    if (seen.add(num)) {
        result.add(num);
    }
}
// Result: [1, 2, 3]
```

<p><strong>How it works:</strong> Explicit loop, no streams. More verbose but easier to debug.</p>

<h4>Complexity (All Approaches)</h4>
<ul>
<li><strong>Time:</strong> O(n) for all approaches</li>
<li><strong>Space:</strong> O(n) for all approaches (storing unique elements)</li>
</ul>

<h4>When to Use</h4>
<ul>
<li><strong>LinkedHashSet:</strong> Simplest, maintains insertion order ⭐ Recommended</li>
<li><strong>Stream + filter:</strong> Functional style, maintains order</li>
<li><strong>Manual iteration:</strong> Most explicit, easiest to debug</li>
<li><strong>Collectors.toSet():</strong> Only if order doesn't matter</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-collections-concurrency/collections/">Learn more: Collections Framework</a></p>

</details>

---

## 5. Find duplicate number from a linked list

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Detect if a linked list contains a duplicate value.</p>

<h3>Approach</h3>
<p>Use Floyd's cycle detection (tortoise & hare) if duplicate implies cycle, or use HashSet to track visited values. <strong>Time: O(n), Space: O(1) for cycle detection or O(n) for HashSet</strong></p>

<h3>HashSet Approach</h3>

```java
Set<Integer> seen = new HashSet<>();
while (head != null) {
    if (!seen.add(head.val)) return head.val;
    head = head.next;
}
```

<h3>Two-pointer Approach (Cycle Detection)</h3>

```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true;  // Cycle detected
}
return false;
```

<p>📚 <a href="/anandu-interview-playbook/dsa/data-structures/">Learn more: Data Structures</a></p>

</details>

---

## 6. Trace the output of a given Java program

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Predict the output of a given Java code snippet.</p>

<h3>Approach</h3>
<p>Trace execution step-by-step:</p>
<ol>
<li>Variable initialization and scope</li>
<li>Method calls and return values</li>
<li>Control flow (loops, conditionals)</li>
<li>Side effects (mutations, I/O)</li>
<li>Type coercion and operator precedence</li>
</ol>

<h3>Example</h3>

```java
int x = 5;
int y = ++x + x++ + x;  // ?
System.out.println(x + " " + y);
```

<h3>Trace</h3>
<ul>
<li><code>++x</code>: x becomes 6 (pre-increment), used value is 6</li>
<li><code>x++</code>: x becomes 7 (post-increment), used value is 6</li>
<li><code>x</code>: x is 7</li>
<li><code>y = 6 + 6 + 7 = 19</code></li>
<li><strong>Output:</strong> <code>7 19</code></li>
</ul>

<h4>Tips</h4>
<ul>
<li>Pay attention to ++/-- (pre vs post-increment)</li>
<li>Watch for operator precedence</li>
<li>Test with actual compilation if unsure</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/basics/">Learn more: Java Basics</a></p>

</details>

---

**See also:** [Coding Questions Overview](index.md), [DSA Algorithms](../../dsa/algorithms.md), [DSA Coding Problems](../../dsa/coding-problems.md)
