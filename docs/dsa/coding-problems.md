# Coding Problems

## Print 1..N without a loop (recursion)
```java
static void printNos(int n) {
    if (n > 0) {
        printNos(n - 1);
        System.out.print(n + " ");
    }
}
// printNos(100);
```

## Palindrome without a loop
**Recursion:**
```java
public boolean isPalindromeRecursive(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    return recursivePalindrome(clean, 0, clean.length() - 1);
}
private boolean recursivePalindrome(String s, int fwd, int bwd) {
    if (fwd >= bwd) return true;
    if (s.charAt(fwd) != s.charAt(bwd)) return false;
    return recursivePalindrome(s, fwd + 1, bwd - 1);
}
```
**Stream API:**
```java
public boolean isPalindrome(String text) {
    String t = text.replaceAll("\\s+", "").toLowerCase();
    return IntStream.range(0, t.length() / 2)
        .noneMatch(i -> t.charAt(i) != t.charAt(t.length() - i - 1));
}
```

## Max subarray sum (Kadane's algorithm)
```java
public static int maxSubArray(int[] nums) {
    int maxSum = Integer.MIN_VALUE, curr = 0;
    for (int n : nums) {
        curr += n;
        maxSum = Math.max(maxSum, curr);
        if (curr < 0) curr = 0;
    }
    return maxSum;
}
// [-2,1,-3,4,-1,2,1,-5,4] -> 6  (subarray [4,-1,2,1])
```
O(n) time, O(1) space — an iterative DP.

## More to add
See [Backlog](../backlog.md): sort students by mark + rank, conversions, bitwise tricks.
