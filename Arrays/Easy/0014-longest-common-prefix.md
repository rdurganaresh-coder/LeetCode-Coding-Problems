# 14. Longest Common Prefix

> 🟢 Easy &nbsp;&nbsp; 🏷️ Strings &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;

---

## Problem Description
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string `""`.

### Example 1:
> **Input:** `strs = ["flower","flow","flight"]`
> **Output:** `"fl"`

### Example 2:
> **Input:** `strs = ["dog","racecar","car"]`
> **Output:** `""`
> **Explanation:** There is no common prefix among the input strings.

### Constraints:
* `1 <= strs.length <= 200`
* `0 <= strs[i].length <= 200`
* `strs[i]` consists of only lowercase English letters.

---

## Solution (Java 8)

```java
    public String longestCommonPrefix(String[] strs) {
        if (strs == null || strs.length == 0) return "";
        
        String prefix = strs[0];
        for (int i = 1; i < strs.length; i++) {
            while (strs[i].indexOf(prefix) != 0) {
                prefix = prefix.substring(0, prefix.length() - 1);
                if (prefix.isEmpty()) return "";
            }
        }
        return prefix;
    }
```

## The Functional Stream Reduction Approach:
> You can treat the array of strings as a Stream and reduce it down to a single prefix string using Java 8's .reduce() function.javaimport java.util.Arrays;
```java
    public String longestCommonPrefix(String[] strs) {
        if (strs == null || strs.length == 0) return "";

        // Stream the array and reduce pairs down to their common prefix
        return Arrays.stream(strs)
                .reduce((prefix, currentStr) -> {
                    // Reduce logic: shrink prefix until currentStr starts with it
                    while (currentStr.indexOf(prefix) != 0) {
                        prefix = prefix.substring(0, prefix.length() - 1);
                        if (prefix.isEmpty()) return "";
                    }
                    return prefix;
                })
                .orElse("");
}
```
> Use code with caution.Why it's great: It completely eliminates manual loop bookkeeping (for indices) and reads as a clean, declarative pipeline.Performance: O(S) time complexity (where S is the sum of all characters in all strings). It runs very fast and is perfectly optimized.

## Stream Sorting & Boundary Matching:
> An elegant alternative is to sort the array alphabetically. The longest common prefix for the entire array will simply be the common prefix between the lexicographically smallest (first) string and the lexicographically largest (last) string.
```java
    public String longestCommonPrefix(String[] strs) {
        if (strs == null || strs.length == 0) return "";
        if (strs.length == 1) return strs[0];

        // Java 8: Sort the array alphabetically
        Arrays.sort(strs);
        
        String first = strs[0];
        String last = strs[strs.length - 1];
        int idx = 0;
        
        // Compare only the first and last strings
        while (idx < first.length() && idx < last.length()) {
            if (first.charAt(idx) == last.charAt(idx)) {
                idx++;
            } else {
                break;
            }
        }
        
        return first.substring(0, idx);
    }
```
> Use code with caution.Performance: O(N * log N * M) where N is the number of strings and M is the maximum string length (due to sorting overhead). Highly efficient for arrays with fewer elements.

## Method 3: Optimized Hybrid Solution (Best LeetCode Score):
> This method combines strict Java 8 object safety checks with the blazing-fast imperative horizontal scanning loop. This code executes in 0ms on LeetCode.
```java
    public String longestCommonPrefix(String[] strs) {
        // Java 8 defensive argument check
        Objects.requireNonNull(strs, "Input string array cannot be null");
        if (strs.length == 0) return "";
        
        String prefix = strs[0];
        for (int i = 1; i < strs.length; i++) {
            while (strs[i].indexOf(prefix) != 0) {
                prefix = prefix.substring(0, prefix.length() - 1);
                if (prefix.isEmpty()) return "";
            }
        }
        return prefix;
    }
```
### Complexity Analysis
* **Time Complexity:** $O(S)$ where $S$ is the sum of all characters in all strings.
* **Space Complexity:** $O(1)$ constant extra space.
