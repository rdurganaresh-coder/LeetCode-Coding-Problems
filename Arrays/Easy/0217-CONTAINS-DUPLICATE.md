# LeetCode 217: Contains Duplicate

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Hash Table &nbsp;&nbsp; 🏷️ Sorting

---

## 📝 Problem Description

Given an integer array `nums`, return `true` if any value appears **at least twice** in the array, and return `false` if every element is distinct.

### Examples

**Example 1:**
* **Input:** `nums = [1, 2, 3, 1]`
* **Output:** `true`
* **Explanation:** The element 1 occurs at index 0 and index 3.

**Example 2:**
* **Input:** `nums = [1, 2, 3, 4]`
* **Output:** `false`
* **Explanation:** All elements are distinct.

**Example 3:**
* **Input:** `nums = [1, 1, 1, 3, 3, 4, 3, 2, 4, 2]`
* **Output:** `true`

### Constraints
* $1 \le \text{nums.length} \le 10^5$
* $-10^9 \le \text{nums}[i] \le 10^9$

---

## 💡 Core Patterns Used

### 1. Hash Set Linear Filter Pattern
* **Concept:** Utilizing the constant-time uniqueness constraint check of hash tables to intercept duplicate elements early.
* **Mechanism:** Iterates over the array while attempting to insert each element into a `HashSet`. Since a set cannot contain duplicate values, `Set.add()` returns `false` if the item is already present, triggering an immediate early return.

### 2. Adjacent Sorting Divergence Pattern
* **Concept:** Sorting values to cluster matching elements into contiguous index locations.
* **Mechanism:** Transforms the array using an $O(N \log N)$ sorting step. Once sorted, a single linear check is run to compare each element with its immediate neighbor (`nums[i] == nums[i-1]`).

### 3. Java 8 Stream Distinct Cardinality Matching
* **Concept:** Comparing data pipeline volume before and after unique filter reductions.
* **Mechanism:** Counts the absolute length of the initial stream pipeline and compares it directly against the length of a filtered `.distinct()` track.

---

## 💻 Java & Java 8 Implementations

### 1. Hash Set Single-Pass (Optimal $O(N)$ Time & $O(N)$ Space)
The standard enterprise approach. It runs in linear time and short-circuits out of the method the moment the first duplicate is detected.

```java
import java.util.HashSet;
import java.util.Set;

public class ContainsDuplicateHashSet {
    public static boolean containsDuplicate(int[] nums) {
        if (nums == null || nums.length <= 1) return false;

        Set<Integer> visited = new HashSet<>(nums.length);
        for (int num : nums) {
            // Set.add() returns false if the item already exists in the set
            if (!visited.add(num)) {
                return true;
            }
        }
        return false;
    }
}
```

### 2. In-Place Sorting ($O(N \log N)$ Time & $O(1)$ Space)
An excellent approach when working under highly restrictive memory limits. It avoids creating additional objects on the heap by modifying the input array array in-place.

```java
import java.util.Arrays;

public class ContainsDuplicateSorting {
    public static boolean containsDuplicate(int[] nums) {
        if (nums == null || nums.length <= 1) return false;

        // Uses Dual-Pivot Quicksort, which has an O(N log N) runtime performance profile
        Arrays.sort(nums);

        // Scan for matching consecutive adjacent values
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] == nums[i - 1]) {
                return true;
            }
        }
        return false;
    }
}
```

### 3. Java 8 Stream Cardinality Match (Declarative Style)
A clean, elegant single-line expression that abstracts loop structures away completely by evaluating stream collection limits.

```java
import java.util.Arrays;

public class ContainsDuplicateStreamCount {
    public static boolean containsDuplicate(int[] nums) {
        if (nums == null || nums.length <= 1) return false;

        // If the number of distinct elements is less than total elements, a duplicate exists
        return Arrays.stream(nums).distinct().count() < nums.length;
    }
}
```

### 4. Java 8 Parallel Short-Circuiting Box Match
An optimization for massive data pipelines. This approach uses parallel evaluation streams along with a custom state container to allow for fast, concurrent early termination.

```java
import java.util.Arrays;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class ContainsDuplicateParallelStream {
    public static boolean containsDuplicate(int[] nums) {
        if (nums == null || nums.length <= 1) return false;

        Set<Integer> visited = ConcurrentHashMap.newKeySet();
        
        // parallel() splits the workload across available CPU cores safely
        return Arrays.stream(nums)
                     .parallel()
                     .boxed()
                     .anyMatch(num -> !visited.add(num));
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Short-Circuits Early? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Hash Set Pass** | $O(N)$ | $O(N)$ | Safe if unshared | **Yes** | Optimal average performance, but allocates object wrappers for primitive integers. |
| **2. In-Place Sort** | $O(N \log N)$ | $O(1)$ (or $O(\log N)$ internal stack) | Safe if unshared | No (must complete sort) | Modifies the input dataset and adds a time penalty, but guarantees constant memory safety. |
| **3. Stream Count** | $O(N)$ | $O(N)$ | Safe | No | Highly expressive and clean code, but forces a full traversal of the array. |
| **4. Parallel Stream**| $O(N)$ | $O(N)$ | **Yes (Thread-Safe)** | **Yes** | Scales exceptionally well on multi-core architectures for large inputs, but introduces thread scheduling overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Hash Set Single-Pass
* **Strengths:** Excellent real-world performance. In scenarios where duplicates occur near the beginning of a large array, the loop exits immediately, minimizing runtime overhead.
* **Weaknesses:** Internal boxing. Passing a primitive `int` to a `HashSet<Integer>` requires automatic boxing into an `Integer` object, which can stress the Garbage Collector if the input array grows into millions of entries.

#### 2. In-Place Sorting Approach
* **Strengths:** Zero heap allocations. Essential for embedded systems, high-frequency low-latency execution graphs, or environments where heap footprints must remain completely stationary.
* **Side Effects:** Destructive mutation. Modifying the structural ordering of the input array can cause unexpected side effects if downstream dependencies rely on the original sequence.

#### 3. Java 8 Stream Cardinality Match
* **Strengths:** High declarative clarity. It tells the reader *what* the code is accomplishing rather than detailing the manual loop tracking steps.
* **Performance Impact:** Lacks early exit functionality. The underlying collection logic must run completely across the entire dataset to establish the definitive `.count()` metadata value, resulting in unnecessary computations if a duplicate exists at the first two indices.

#### 4. Java 8 Parallel Short-Circuiting Box Match
* **Strengths:** Blends functional elegance with high-concurrency throughput optimizations. Using `.anyMatch()` reintroduces crucial short-circuit behavior to stream pipelines.
* **Thread Overhead:** For small arrays ($N \le 10^3$), spawning and coordinating data splits across a `ForkJoinPool` is less efficient than running a simple single-threaded loop. This approach is best reserved for massive data feeds.
