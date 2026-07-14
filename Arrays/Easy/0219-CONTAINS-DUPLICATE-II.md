# LeetCode 219: Contains Duplicate II

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Hash Table &nbsp;&nbsp; 🏷️ Sliding Window

---

## 📝 Problem Description

Given an integer array `nums` and an integer `k`, return `true` *if there are two **distinct indices** `i` and `j` in the array such that `nums[i] == nums[j]` and `abs(i - j) <= k`*.

### Examples

**Example 1:**
* **Input:** `nums =, k = 3`
* **Output:** `true`
* **Explanation:** The element 1 appears at index 0 and index 3. The absolute difference between their indices is $|0 - 3| = 3$, which is $\le k$.

**Example 2:**
* **Input:** `nums =, k = 1`
* **Output:** `true`
* **Explanation:** The element 1 appears at index 2 and index 3. The absolute difference $|2 - 3| = 1$, which is $\le k$.

**Example 3:**
* **Input:** `nums =, k = 2`
* **Output:** `false`
* **Explanation:** While duplicates exist, the closest matching elements (e.g., 1 at index 0 and index 3) have an index difference of 3, which is greater than $k = 2$.

### Constraints
* $1 \le \text{nums.length} \le 10^5$
* $-10^9 \le \text{nums}[i] \le 10^9$
* $0 \le k \le 10^5$

---

## 💡 Core Patterns Used

### 1. Sliding Window Set Pattern
* **Concept:** Constraining a Hash Set search boundary to a fixed maximum window length $k$.
* **Mechanism:** Maintains a standard `HashSet` that stores at most $k$ elements. As the index loop cursor $i$ advances, it tries to add `nums[i]`. If it fails, a duplicate exists within the window. If the loop index passes the window length ($i \ge k$), the oldest element `nums[i - k]` is dropped out of the set to keep the search area bounded.

### 2. Map-Based Last-Seen Coordinate Indexing
* **Concept:** Tracking the exact index locations of previously evaluated numbers.
* **Mechanism:** Maps elements to their most recent array index (`HashMap<Integer, Integer>`). When checking a value, if it is already present in the map, the code compares the current index with the stored historical index. If the difference satisfies $\le k$, it returns early; otherwise, it updates the map with the fresh index position.

---

## 💻 Java & Java 8 Implementations

### 1. Sliding Window HashSet (Optimal $O(N)$ Time & $O(\min(N, k))$ Space)
The absolute gold standard approach for memory efficiency. It dynamically limits the tracking set's memory allocations to a maximum capacity of $k$.

```java
import java.util.HashSet;
import java.util.Set;

public class ContainsDuplicateIISlidingWindow {
    public static boolean containsNearbyDuplicate(int[] nums, int k) {
        if (nums == null || nums.length <= 1 || k <= 0) return false;

        Set<Integer> windowSet = new HashSet<>();
        
        for (int i = 0; i < nums.length; i++) {
            // If addition fails, a matching duplicate exists within the dynamic k-window range
            if (!windowSet.add(nums[i])) {
                return true;
            }
            
            // Maintain sliding window boundaries by removing the stale element at the tail
            if (i >= k) {
                windowSet.remove(nums[i - k]);
            }
        }
        return false;
    }
}
```

### 2. Coordinate Tracking Map ($O(N)$ Time & $O(N)$ Space)
An intuitive, tracking-focused approach that records the exact location where each number was last spotted in the array stream.

```java
import java.util.HashMap;
import java.util.Map;

public class ContainsDuplicateIILastSeenMap {
    public static boolean containsNearbyDuplicate(int[] nums, int k) {
        if (nums == null || nums.length <= 1 || k <= 0) return false;

        Map<Integer, Integer> lastSeenIndices = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int currentNum = nums[i];
            
            // Check if the number has a valid past index inside the allowed k distance limit
            if (lastSeenIndices.containsKey(currentNum)) {
                if (i - lastSeenIndices.get(currentNum) <= k) {
                    return true;
                }
            }
            
            // Always overwrite with the most recent index coordinates to minimize distance checks
            lastSeenIndices.put(currentNum, i);
        }
        return false;
    }
}
```

### 3. Java 8 Stateful Stream Index Scan ($O(N)$ Functional Time)
This approach leverages an `IntStream` to iterate over positions, using a localized state dictionary map inside a short-circuit predicate flow.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.stream.IntStream;

public class ContainsDuplicateIIStreamMap {
    public static boolean containsNearbyDuplicate(int[] nums, int k) {
        if (nums == null || nums.length <= 1 || k <= 0) return false;

        Map<Integer, Integer> lastSeen = new HashMap<>();

        // anyMatch short-circuits instantly upon locating the first match
        return IntStream.range(0, nums.length)
                .anyMatch(i -> {
                    Integer previousIndex = lastSeen.put(nums[i], i);
                    return previousIndex != null && (i - previousIndex <= k);
                });
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Memory Footprint Strategy | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Sliding Window** | $O(N)$ | $O(\min(N, k))$ | Safe if unshared | Upper-bounded at size $k$ | Restricts map size efficiently; ideal for large arrays with small $k$ parameters. |
| **2. Last-Seen Map** | $O(N)$ | $O(N)$ | Safe if unshared | Extends up to full size $N$ | Highly granular coordinate metadata preservation, but has a larger heap layout. |
| **3. Stream Scan** | $O(N)$ | $O(N)$ | Safe | Extends up to full size $N$ | Clean declarative short-circuit pipeline; introduces minor autoboxing overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Sliding Window HashSet Approach
* **Strengths:** Superb space-saving profiles. If $k = 5$ and the input size $N = 10^5$, the memory collection structure holds at most 5 elements at any given point during traversal. This keeps memory consumption constant ($O(1)$ relative to massive variations in array scales).
* **Weaknesses:** Requires continuous cleanup operations, invoking `.remove()` calculations on every index iteration once the window boundary fills up.

#### 2. Coordinate Tracking Map
* **Strengths:** Highly resilient tracking properties. By continually updating the coordinates using `lastSeenIndices.put(currentNum, i)`, it guarantees that calculations always evaluate the absolute shortest relative gap for any repeating number sequence.
* **Limitations:** Unbounded memory growth. Because elements are never cleaned out or removed from the dictionary map, memory usage scales lineally ($O(N)$) with the unique element count of the array.

#### 3. Java 8 Stateful Stream Index Scan
* **Strengths:** Implements modern, expressive code syntax. Utilizing `Map.put()`'s return property (which natively returns the *previous* value associated with the target key before substitution) results in an incredibly concise, elegant layout block.
* **Performance Impact:** Autoboxing tax. Translating array search elements into map keys repeatedly prompts primitive `int` to `Integer` boxing wrapper allocations inside the application's memory pool.
