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

## 7. Find the index where the sum of elements on the left equals the sum on the right

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given an int array, return the lowest index where the sum of the elements to its left equals the sum of the elements to its right (the element itself is excluded; an empty side sums to 0). Return -1 if none exists.</p>
<p><strong>Example:</strong> [1, 7, 3, 6, 5, 6] → 3 (left 1+7+3 = 11, right 5+6 = 11)</p>

<h3>Approach 1: Brute Force</h3>

```java
public class PivotBrute {
    static int pivotIndex(int[] a) {
        for (int i = 0; i < a.length; i++) {
            long left = 0, right = 0;
            for (int j = 0; j < i; j++) left += a[j];
            for (int j = i + 1; j < a.length; j++) right += a[j];
            if (left == right) return i;
        }
        return -1;
    }

    public static void main(String[] args) {
        System.out.println(pivotIndex(new int[]{1, 7, 3, 6, 5, 6})); // 3
        System.out.println(pivotIndex(new int[]{1, 2, 3}));          // -1
    }
}
```
<p>Recomputes both sums for every index.</p>

<h3>Approach 2: Prefix Sum Array</h3>

```java
public class PivotPrefix {
    static int pivotIndex(int[] a) {
        int n = a.length;
        long[] prefix = new long[n + 1];            // prefix[i] = sum of a[0..i-1]
        for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + a[i];
        for (int i = 0; i < n; i++) {
            long left = prefix[i];
            long right = prefix[n] - prefix[i + 1];
            if (left == right) return i;
        }
        return -1;
    }

    public static void main(String[] args) {
        System.out.println(pivotIndex(new int[]{1, 7, 3, 6, 5, 6})); // 3
        System.out.println(pivotIndex(new int[]{2, 1, -1}));         // 0
    }
}
```

<h3>Approach 3: Running Sum (Optimal)</h3>

```java
public class PivotRunning {
    static int pivotIndex(int[] a) {
        long total = 0;
        for (int x : a) total += x;
        long left = 0;
        for (int i = 0; i < a.length; i++) {
            // right = total - left - a[i]
            if (left == total - left - a[i]) return i;
            left += a[i];
        }
        return -1;
    }

    public static void main(String[] args) {
        System.out.println(pivotIndex(new int[]{1, 7, 3, 6, 5, 6})); // 3
        System.out.println(pivotIndex(new int[]{}));                 // -1
    }
}
```
<p>Keep only the running left sum and derive the right sum from the total.</p>

<h4>Complexity Comparison</h4>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Brute force</td><td>O(n²)</td><td>O(1)</td><td>Simplest</td><td>Too slow for large input</td></tr>
<tr><td>Prefix array</td><td>O(n)</td><td>O(n)</td><td>Reusable for range-sum queries</td><td>Extra memory</td></tr>
<tr><td>Running sum</td><td>O(n)</td><td>O(1)</td><td>Best of both</td><td>Single-purpose</td></tr>
</table>

<h4>Key Insight: Prefix Sums</h4>
<ul>
<li><code>right = total - left - a[i]</code> removes the need for a second pass or array.</li>
<li>Use <code>long</code> for sums to avoid int overflow; handle negatives and empty arrays.</li>
<li>Clarify with the interviewer: lowest index vs any index, and whether the answer can be at the ends.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>

---

## 8. Generate all combinations of well-formed parentheses for n pairs

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given n pairs of parentheses, generate all valid combinations.</p>
<p><strong>Example:</strong> n = 3 → ["((()))", "(()())", "(())()", "()(())", "()()()"]</p>

<h3>Approach 1: Brute Force (generate all, then validate)</h3>

```java
import java.util.*;

public class ParenBrute {
    static List<String> generate(int n) {
        List<String> result = new ArrayList<>();
        build(new char[2 * n], 0, result);
        return result;
    }

    static void build(char[] cur, int pos, List<String> result) {
        if (pos == cur.length) {
            if (valid(cur)) result.add(new String(cur));
            return;
        }
        cur[pos] = '(';
        build(cur, pos + 1, result);
        cur[pos] = ')';
        build(cur, pos + 1, result);
    }

    static boolean valid(char[] s) {
        int balance = 0;
        for (char c : s) {
            balance += (c == '(') ? 1 : -1;
            if (balance < 0) return false;
        }
        return balance == 0;
    }

    public static void main(String[] args) {
        System.out.println(generate(3));
    }
}
```

<h3>Approach 2: Backtracking with Pruning (Optimal)</h3>

```java
import java.util.*;

public class ParenBacktrack {
    static List<String> generate(int n) {
        List<String> result = new ArrayList<>();
        backtrack(new StringBuilder(), 0, 0, n, result);
        return result;
    }

    static void backtrack(StringBuilder cur, int open, int close, int n, List<String> result) {
        if (cur.length() == 2 * n) {
            result.add(cur.toString());
            return;
        }
        if (open < n) {                       // can still open
            cur.append('(');
            backtrack(cur, open + 1, close, n, result);
            cur.deleteCharAt(cur.length() - 1);
        }
        if (close < open) {                   // can close only if there is an unmatched '('
            cur.append(')');
            backtrack(cur, open, close + 1, n, result);
            cur.deleteCharAt(cur.length() - 1);
        }
    }

    public static void main(String[] args) {
        System.out.println(generate(3));
        System.out.println(generate(1)); // [()]
    }
}
```
<p>Only valid prefixes are ever extended, so no invalid string is built.</p>

<h3>Approach 3: Dynamic Programming (Closure Number)</h3>

```java
import java.util.*;

public class ParenDp {
    static List<String> generate(int n) {
        List<List<String>> dp = new ArrayList<>();
        dp.add(List.of(""));
        for (int i = 1; i <= n; i++) {
            List<String> cur = new ArrayList<>();
            for (int j = 0; j < i; j++) {              // "(" + dp[j] + ")" + dp[i-1-j]
                for (String left : dp.get(j))
                    for (String right : dp.get(i - 1 - j))
                        cur.add("(" + left + ")" + right);
            }
            dp.add(cur);
        }
        return dp.get(n);
    }

    public static void main(String[] args) {
        System.out.println(generate(3));
    }
}
```

<h4>Complexity Comparison</h4>
<p>Let C(n) be the n-th Catalan number, the count of valid results (about 4^n / (n^1.5 · √π)).</p>
<table>
<tr><th>Approach</th><th>Time</th><th>Space (excluding output)</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Brute force</td><td>O(2^(2n) · n)</td><td>O(n)</td><td>Easy to reason about</td><td>Explores many invalid strings</td></tr>
<tr><td>Backtracking</td><td>O(C(n) · n), roughly O(4^n / √n)</td><td>O(n) recursion depth</td><td>Output-sensitive, standard answer</td><td>Recursive</td></tr>
<tr><td>DP</td><td>O(C(n) · n) overall</td><td>O(C(n) · n) stores all smaller results</td><td>No recursion, shows the Catalan structure</td><td>Holds all intermediate lists</td></tr>
</table>

<h4>Key Insight: Pruning Rules</h4>
<ul>
<li>Valid iff <code>open &lt;= n</code> and <code>close &lt;= open</code> at every prefix.</li>
<li>The "O(n!)" sometimes quoted is not accurate; the count of results is the Catalan number, which grows like 4^n / n^1.5.</li>
<li>Undo the choice after the recursive call (backtrack) when reusing one buffer.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>

---

## 9. Check if two strings are anagrams

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Return true if two strings contain the same characters with the same counts.</p>
<p><strong>Example:</strong> "listen", "silent" → true; "rat", "car" → false</p>

<h3>Approach 1: Sort and Compare</h3>

```java
import java.util.Arrays;

public class AnagramSort {
    static boolean isAnagram(String a, String b) {
        if (a.length() != b.length()) return false;
        char[] x = a.toCharArray();
        char[] y = b.toCharArray();
        Arrays.sort(x);
        Arrays.sort(y);
        return Arrays.equals(x, y);
    }

    public static void main(String[] args) {
        System.out.println(isAnagram("listen", "silent")); // true
        System.out.println(isAnagram("rat", "car"));       // false
    }
}
```

<h3>Approach 2: Frequency Array (lowercase a-z)</h3>

```java
public class AnagramCount {
    static boolean isAnagram(String a, String b) {
        if (a.length() != b.length()) return false;
        int[] counts = new int[26];
        for (int i = 0; i < a.length(); i++) {
            counts[a.charAt(i) - 'a']++;
            counts[b.charAt(i) - 'a']--;
        }
        for (int c : counts) if (c != 0) return false;
        return true;
    }

    public static void main(String[] args) {
        System.out.println(isAnagram("anagram", "nagaram")); // true
        System.out.println(isAnagram("abc", "abd"));         // false
    }
}
```

<h3>Approach 3: HashMap (any characters, including Unicode)</h3>

```java
import java.util.HashMap;
import java.util.Map;

public class AnagramMap {
    static boolean isAnagram(String a, String b) {
        if (a.length() != b.length()) return false;
        Map<Character, Integer> freq = new HashMap<>();
        for (char c : a.toCharArray()) freq.merge(c, 1, Integer::sum);
        for (char c : b.toCharArray()) {
            Integer count = freq.get(c);
            if (count == null || count == 0) return false;
            freq.put(c, count - 1);
        }
        return true;   // equal lengths + never exceeded a count => all counts are zero
    }

    public static void main(String[] args) {
        System.out.println(isAnagram("Dormitory", "Dirty room")); // false (case and spaces matter)
        System.out.println(isAnagram("aab", "aba"));              // true
    }
}
```

<h4>Complexity Comparison</h4>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Sort</td><td>O(n log n)</td><td>O(n)</td><td>Simple, any chars</td><td>Slower</td></tr>
<tr><td>int[26]</td><td>O(n)</td><td>O(1)</td><td>Fastest, tiny memory</td><td>Only a-z; throws on other chars</td></tr>
<tr><td>HashMap</td><td>O(n)</td><td>O(k), k = distinct chars</td><td>Handles any char</td><td>Boxing overhead</td></tr>
</table>

<h4>Key Insight: Clarify the Rules</h4>
<ul>
<li>Ask about case sensitivity, spaces/punctuation, and the alphabet; normalize first (<code>toLowerCase()</code>, strip non-letters) if needed.</li>
<li>Checking lengths first is a cheap early exit.</li>
<li>For full Unicode beyond the BMP, iterate by code points (<code>String.codePoints()</code>), because a <code>char</code> is only half of a surrogate pair.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings & Primitives</a></p>

</details>

---

## 10. Find the most frequent number in an array

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Return the element that appears most often. If there is a tie, any (or the first to reach the max) is acceptable; state your choice.</p>
<p><strong>Example:</strong> [3, 2, 3] → 3</p>

<h3>Approach 1: HashMap Frequency Count</h3>

```java
import java.util.*;

public class MostFrequentMap {
    static int mostFrequent(int[] nums) {
        Map<Integer, Integer> freq = new HashMap<>();
        int best = nums[0], bestCount = 0;
        for (int n : nums) {
            int c = freq.merge(n, 1, Integer::sum);
            if (c > bestCount) { bestCount = c; best = n; }
        }
        return best;
    }

    public static void main(String[] args) {
        System.out.println(mostFrequent(new int[]{3, 2, 3}));          // 3
        System.out.println(mostFrequent(new int[]{1, 1, 2, 2, 2, 5})); // 2
    }
}
```
<p>Ties resolve to the value that reached the top count first.</p>

<h3>Approach 2: Sort and Scan Runs</h3>

```java
import java.util.Arrays;

public class MostFrequentSort {
    static int mostFrequent(int[] nums) {
        int[] a = nums.clone();                 // do not mutate the caller's array
        Arrays.sort(a);
        int best = a[0], bestRun = 1, run = 1;
        for (int i = 1; i < a.length; i++) {
            run = (a[i] == a[i - 1]) ? run + 1 : 1;
            if (run > bestRun) { bestRun = run; best = a[i]; }
        }
        return best;
    }

    public static void main(String[] args) {
        System.out.println(mostFrequent(new int[]{3, 2, 3})); // 3
        System.out.println(mostFrequent(new int[]{7}));       // 7
    }
}
```

<h3>Approach 3: Boyer-Moore Voting (only when a majority element exists)</h3>

```java
public class MajorityVote {
    // Valid only if some element appears MORE than n/2 times
    static int majority(int[] nums) {
        int candidate = nums[0], count = 0;
        for (int n : nums) {
            if (count == 0) candidate = n;
            count += (n == candidate) ? 1 : -1;
        }
        return candidate;
    }

    public static void main(String[] args) {
        System.out.println(majority(new int[]{3, 2, 3}));             // 3
        System.out.println(majority(new int[]{2, 2, 1, 1, 1, 2, 2})); // 2
    }
}
```

<h4>Complexity Comparison</h4>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>HashMap</td><td>O(n)</td><td>O(k) distinct values</td><td>General, one pass</td><td>Extra memory</td></tr>
<tr><td>Sort + scan</td><td>O(n log n)</td><td>O(n) with the clone (O(1) if sorting in place)</td><td>No map, easy</td><td>Slower</td></tr>
<tr><td>Boyer-Moore</td><td>O(n)</td><td>O(1)</td><td>Optimal memory</td><td>Correct only if a strict majority exists; verify with a second pass otherwise</td></tr>
</table>

<h4>Key Insight: Pick by Constraints</h4>
<ul>
<li>"Most frequent" (mode) in general needs counting; "majority" (more than n/2) allows the O(1) voting trick.</li>
<li>Streams alternative: <code>Arrays.stream(a).boxed().collect(Collectors.groupingBy(x -> x, Collectors.counting()))</code> then take the max entry.</li>
<li>Handle empty input and define tie-breaking explicitly. For a small known value range, a counting array beats a map.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/data-structures/">Learn more: Data Structures</a></p>

</details>

---

## 11. Print all combinations of 1 and 2 that sum to n

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given a positive integer n, print every ordered sequence of 1s and 2s whose sum is n (like climbing n stairs taking 1 or 2 steps).</p>
<p><strong>Example:</strong> n = 4 → [1,1,1,1], [1,1,2], [1,2,1], [2,1,1], [2,2]</p>

<h3>Approach 1: Backtracking</h3>

```java
import java.util.*;

public class StepsBacktrack {
    static List<List<Integer>> combinations(int n) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(n, new ArrayList<>(), result);
        return result;
    }

    static void backtrack(int remaining, List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) {
            result.add(new ArrayList<>(path));      // copy the path
            return;
        }
        for (int step = 1; step <= 2; step++) {
            if (step > remaining) break;
            path.add(step);
            backtrack(remaining - step, path, result);
            path.remove(path.size() - 1);           // undo choice
        }
    }

    public static void main(String[] args) {
        combinations(4).forEach(System.out::println);
    }
}
```

<h3>Approach 2: Dynamic Programming (build lists bottom-up)</h3>

```java
import java.util.*;

public class StepsDp {
    static List<List<Integer>> combinations(int n) {
        List<List<List<Integer>>> dp = new ArrayList<>();
        dp.add(List.of(List.of()));                         // sum 0: one empty combination
        for (int sum = 1; sum <= n; sum++) {
            List<List<Integer>> cur = new ArrayList<>();
            for (int step = 1; step <= 2 && step <= sum; step++) {
                for (List<Integer> prev : dp.get(sum - step)) {
                    List<Integer> next = new ArrayList<>(prev);
                    next.add(step);
                    cur.add(next);
                }
            }
            dp.add(cur);
        }
        return dp.get(n);
    }

    public static void main(String[] args) {
        combinations(4).forEach(System.out::println);
    }
}
```
<p>Order of output differs from backtracking, but the set is the same.</p>

<h3>Approach 3: DP Count Only (when you do not need the lists)</h3>

```java
public class StepsCount {
    static long count(int n) {
        long a = 1, b = 1;                 // ways(0) = 1, ways(1) = 1
        for (int i = 2; i <= n; i++) {
            long c = a + b;
            a = b;
            b = c;
        }
        return b;                          // Fibonacci: ways(n) = ways(n-1) + ways(n-2)
    }

    public static void main(String[] args) {
        System.out.println(count(4));  // 5
        System.out.println(count(10)); // 89
    }
}
```

<h4>Complexity Comparison</h4>
<p>The number of combinations is Fibonacci(n+1), roughly 1.618^n, so listing them is exponential regardless of method.</p>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Backtracking</td><td>O(F(n) · n) to produce all sequences</td><td>O(n) recursion plus output</td><td>Prints lazily, minimal memory</td><td>Recursion depth n</td></tr>
<tr><td>DP lists</td><td>O(F(n) · n)</td><td>O(F(n) · n) stores all prefixes</td><td>No recursion</td><td>Memory heavy</td></tr>
<tr><td>DP count</td><td>O(n)</td><td>O(1)</td><td>Fast for "how many"</td><td>Gives only the number</td></tr>
</table>

<h4>Key Insight: Enumerate vs Count</h4>
<ul>
<li>If asked only for the count, never enumerate: it is Fibonacci. Use <code>long</code> or <code>BigInteger</code> as n grows (long overflows after about n = 90).</li>
<li>Recursion without memo for counting is O(2^n); memoization or the iterative form makes it O(n).</li>
<li>Remember to copy the path before adding it to the results, otherwise all entries end up empty.</li>
<li>Variant: unordered combinations (1+2 same as 2+1) is a different, coin-change style problem.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>

---

## 12. Reverse a string without using predefined methods

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Reverse a string without <code>StringBuilder.reverse()</code>, <code>Collections.reverse()</code> or similar helpers.</p>
<p><strong>Example:</strong> "hello" → "olleh"</p>

<h3>Approach 1: Two-Pointer Swap on a char Array</h3>

```java
public class ReverseSwap {
    static String reverse(String s) {
        if (s == null) return null;
        char[] c = s.toCharArray();
        for (int i = 0, j = c.length - 1; i < j; i++, j--) {
            char tmp = c[i];
            c[i] = c[j];
            c[j] = tmp;
        }
        return new String(c);
    }

    public static void main(String[] args) {
        System.out.println(reverse("hello")); // olleh
        System.out.println(reverse(""));      // (empty)
    }
}
```

<h3>Approach 2: Build from the End</h3>

```java
public class ReverseLoop {
    static String reverse(String s) {
        char[] result = new char[s.length()];
        for (int i = 0; i < s.length(); i++) {
            result[i] = s.charAt(s.length() - 1 - i);
        }
        return new String(result);
    }

    public static void main(String[] args) {
        System.out.println(reverse("interview")); // weivretni
    }
}
```
<p>Avoid <code>result += ch</code> in a loop: strings are immutable, so that is O(n²).</p>

<h3>Approach 3: Recursion</h3>

```java
public class ReverseRecursive {
    static String reverse(String s) {
        if (s.length() <= 1) return s;
        return reverse(s.substring(1)) + s.charAt(0);
    }

    public static void main(String[] args) {
        System.out.println(reverse("abc")); // cba
    }
}
```
<p>Elegant but costly: <code>substring</code> and concatenation copy, and deep recursion risks <code>StackOverflowError</code> for long strings.</p>

<h4>Complexity Comparison</h4>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Two-pointer swap</td><td>O(n)</td><td>O(n) char copy</td><td>Standard, n/2 swaps</td><td>None significant</td></tr>
<tr><td>Build from end</td><td>O(n)</td><td>O(n)</td><td>Very readable</td><td>Slightly more reads</td></tr>
<tr><td>Recursion</td><td>O(n²) due to copying</td><td>O(n²) transient, O(n) stack depth</td><td>Shows recursion</td><td>Slow, stack overflow risk</td></tr>
</table>

<h4>Key Insight: Strings Are Immutable</h4>
<ul>
<li>Java strings cannot be reversed in place; any solution needs a new char array or builder (O(n) extra space). Only a mutable <code>char[]</code> can be reversed in place.</li>
<li><code>length()</code>, <code>charAt()</code> and <code>toCharArray()</code> are normally acceptable "basic" methods; confirm with the interviewer.</li>
<li>Char-based reversal breaks surrogate pairs (emoji) and combining characters; <code>StringBuilder.reverse()</code> handles surrogate pairs, not grapheme clusters.</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/java-core/strings-primitives/">Learn more: Strings & Primitives</a></p>

</details>

---

## 13. Sort students by marks and assign ranks (including ties)

<details>
<summary>View Solution</summary>

<h3>Problem</h3>
<p>Given students with marks, sort by marks descending and assign ranks. Equal marks share a rank. Show standard competition ranking (1, 2, 2, 4) and mention dense ranking (1, 2, 2, 3).</p>
<p><strong>Example:</strong> A=90, B=85, C=85, D=70 → A:1, B:2, C:2, D:4</p>

<h3>Approach 1: Sort, Then Scan</h3>

```java
import java.util.*;

public class RankSort {
    record Student(String name, int marks) {}

    static void printRanks(List<Student> students) {
        List<Student> sorted = new ArrayList<>(students);
        sorted.sort(Comparator.comparingInt(Student::marks).reversed()
                              .thenComparing(Student::name));   // stable, deterministic tie order
        int rank = 0;
        for (int i = 0; i < sorted.size(); i++) {
            if (i == 0 || sorted.get(i).marks() != sorted.get(i - 1).marks()) {
                rank = i + 1;                                   // jump past tied positions
            }
            System.out.println(sorted.get(i).name() + " " + sorted.get(i).marks() + " rank " + rank);
        }
    }

    public static void main(String[] args) {
        printRanks(List.of(new Student("A", 90), new Student("B", 85),
                           new Student("C", 85), new Student("D", 70)));
    }
}
```
<p>Uses a record (Java 16+).</p>

<h3>Approach 2: Group by Marks with a TreeMap</h3>

```java
import java.util.*;

public class RankTreeMap {
    record Student(String name, int marks) {}

    static Map<String, Integer> ranks(List<Student> students) {
        TreeMap<Integer, List<String>> byMarks = new TreeMap<>(Comparator.reverseOrder());
        for (Student s : students) {
            byMarks.computeIfAbsent(s.marks(), k -> new ArrayList<>()).add(s.name());
        }
        Map<String, Integer> result = new LinkedHashMap<>();
        int position = 1;                                       // competition rank = 1 + students ahead
        for (List<String> group : byMarks.values()) {
            for (String name : group) result.put(name, position);
            position += group.size();
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println(ranks(List.of(new Student("A", 90), new Student("B", 85),
                                         new Student("C", 85), new Student("D", 70))));
        // {A=1, B=2, C=2, D=4}
    }
}
```
<p>For dense ranking, increment by 1 per group instead of by <code>group.size()</code>.</p>

<h3>Approach 3: Streams</h3>

```java
import java.util.*;
import java.util.stream.*;

public class RankStreams {
    record Student(String name, int marks) {}

    public static void main(String[] args) {
        List<Student> students = List.of(new Student("A", 90), new Student("B", 85),
                                         new Student("C", 85), new Student("D", 70));

        // rank = 1 + number of students with strictly higher marks
        students.stream()
                .sorted(Comparator.comparingInt(Student::marks).reversed())
                .forEach(s -> {
                    long higher = students.stream().filter(o -> o.marks() > s.marks()).count();
                    System.out.println(s.name() + " rank " + (higher + 1));
                });
    }
}
```
<p>Short and obviously correct, but quadratic.</p>

<h4>Complexity Comparison</h4>
<table>
<tr><th>Approach</th><th>Time</th><th>Space</th><th>Pros</th><th>Cons</th></tr>
<tr><td>Sort + scan</td><td>O(n log n)</td><td>O(n)</td><td>Simple, single pass after sort</td><td>Needs a copy to keep input order</td></tr>
<tr><td>TreeMap grouping</td><td>O(n log k), k = distinct marks</td><td>O(n)</td><td>Easy dense vs competition ranking</td><td>More code</td></tr>
<tr><td>Streams count</td><td>O(n²)</td><td>O(n)</td><td>Very short</td><td>Does not scale</td></tr>
</table>

<h4>Key Insight: Ties and Ranking Styles</h4>
<ul>
<li>Competition ranking (1, 2, 2, 4) = 1 + count of students strictly ahead. Dense ranking (1, 2, 2, 3) increments per distinct score. Ask which one is wanted.</li>
<li>Add a secondary key (name) so output is deterministic. Compare ints with <code>Integer.compare</code> or <code>comparingInt</code>, never by subtraction (overflow).</li>
<li>For marks in a small range (0 to 100), counting sort gives O(n + range).</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/dsa/algorithms/">Learn more: Algorithms</a></p>

</details>

---

**See also:** [Coding Questions Overview](index.md), [DSA Algorithms](../../dsa/algorithms.md), [DSA Coding Problems](../../dsa/coding-problems.md)
