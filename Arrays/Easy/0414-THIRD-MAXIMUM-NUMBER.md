# LeetCode 414: Third Maximum Number

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Sorting

---

## 📝 Problem Description

Given an integer array `nums`, return *the **third distinct maximum** number in this array*. If the third maximum does not exist, return *the **maximum** number*.

### Examples

**Example 1:**
* **Input:** `nums =`
* **Output:** `1`
* **Explanation:** 
  * The first distinct maximum is 3.
  * The second distinct maximum is 2.
  * The third distinct maximum is 1.

**Example 2:**
* **Input:** `nums =`
* **Output:** `2`
* **Explanation:** 
  * The first distinct maximum is 2.
  * The second distinct maximum is 1.
  * The third distinct maximum does not exist (there are only two distinct numbers).
  * According to the problem statement, the maximum number (2) is returned instead.

**Example 3:**
* **Input:** `nums =`
* **Output:** `1`
* **Explanation:** 
  * Note that the third distinct maximum is requested.
  * The first distinct maximum is 3.
  * The second distinct maximum is 2 (both 2s represent the same second distinct maximum).
  * The third distinct maximum is 1.

### Constraints
* $1 \le \text{nums.length} \le 10^4$
* $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$

**Follow-up:** Can you find an $O(n)$ time complexity solution?

---

## 💡 Core Patterns Used

### 1. Finite Slots Elimination Pattern (Three-Pointer Sweep)
* **Concept:** Maintaining a small, ordered tracker window containing the top three maximum unique elements seen so far during a single pass.
* **Mechanism:** Uses three `Long` or `Integer` wrapper tracking pointer variables (`first`, `second`, `third`) initialized to `null`. As the array cursor moves forward, it filters out incoming elements that match any already tracked values. Valid new values are placed into their correct relative positions, dropping lower maximums down a slot.

### 2. TreeSet Bounded Priority Window
* **Concept:** Constraining a self-sorting binary tree container to store a maximum of 3 unique elements.
* **Mechanism:** Iterates over the array while inserting items into a `TreeSet`. If the set expands past 3 elements, the absolute smallest value (`set.first()`) is immediately dropped out of the collection, keeping tree balancing costs to $O(1)$.

### 3. Java 8 Stream Deduplication Pipeline
* **Concept:** Chaining functional unique filters and sorted boxing reductions to select target position boundaries.
* **Mechanism:** Passes data arrays through sequential `.distinct()` and descending comparator streams to cleanly map array profiles to individual results.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal Three-Pointer Variable Tracking (Optimal $O(N)$ Time & $O(1)$ Space)
The definitive benchmark standard. It processes elements in a single forward pass, updates tracking registers cleanly, and avoids heap allocation overhead.

```java
public class ThirdMaxThreePointers {
    public static int thirdMax(int[] nums) {
        // Use boxed Long objects to natively handle arrays containing Integer.MIN_VALUE values
        Long first = null;
        Long second = null;
        Long third = null;

        for (int num : nums) {
            long current = num;
            
            // Deduplication guard: Skip if the element is already recorded in the tracking window
            if ((first != null && current == first) || 
                (second != null && current == second) || 
                (third != null && current == third)) {
                continue;
            }

            // Slide tracking window boundaries downward sequentially based on position mapping
            if (first == null || current > first) {
                third = second;
                second = first;
                first = current;
            } else if (second == null || current > second) {
                third = second;
                second = current;
            } else if (third == null || current > third) {
                third = current;
            }
        }

        // Fallback constraint logic rule validation
        return (third == null) ? first.intValue() : third.intValue();
    }
}
```

### 2. Bounded TreeSet Window Optimization ($O(N)$ Time & $O(1)$ Space)
An exceptionally readable object-oriented solution. It balances processing limits dynamically by capping tree structural expansion to a strict limit of 3 elements.

```java
import java.util.TreeSet;

public class ThirdMaxTreeSet {
    public static int thirdMax(int[] nums) {
        TreeSet<Integer> topThree = new TreeSet<>();
        
        for (int num : nums) {
            topThree.add(num);
            // Capping size ensures internal binary tree search rebalancing takes O(1) constant cycles
            if (topThree.size() > 3) {
                topThree.remove(topThree.first()); // Evict the lowest recorded maximum element
            }
        }
        
        // If 3 distinct values exist, return the minimum element of the top 3; otherwise return the absolute maximum
        return topThree.size() == 3 ? topThree.first() : topThree.last();
    }
}
```

### 3. Java 8 Pure Stream Extraction Pipeline (Declarative Style)
A clean functional approach that uses stream pipelining, sorting transformations, and deduplication boxes to isolate results instantly.

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

public class ThirdMaxStreamPipeline {
    public static int thirdMax(int[] nums) {
        // Collect sorted, distinct wrapped elements into a structural list
        List<Integer> distinctSortedMax = Arrays.stream(nums)
            .boxed()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .collect(Collectors.toList());

        // Select the third element if it exists; otherwise fallback to the maximum element at index 0
        return distinctSortedMax.size() >= 3 ? distinctSortedMax.get(2) : distinctSortedMax.get(0);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Three Pointers** | $O(N)$ | $O(1)$ | Safe if unshared | **Zero (Immune)** | Fastest option by far; code requires handling minor wrapper tracking states carefully. |
| **2. Bounded TreeSet**| $O(N)$ | $O(1)$ (Capped structure) | Safe if unshared | Low | Exceptional balance of clean object architecture and memory restriction bounds. |
| **3. Stream Pipeline** | $O(N \log N)$ | $O(N)$ | Safe | High | High functional clarity, but sorting full deduplicated copies impacts efficiency. |

---

### Detailed Evaluation Breakdown

#### 1. Optimal Three-Pointer Variable Tracking
* **Strengths:** Blazing execution efficiency. Processing arrays via register variable comparisons maximizes L1/L2 cache spatial memory tracking accuracy. It effortlessly complies with the $O(n)$ time execution target.
* **Boundary Case Resilience:** Initializing pointers to a boxed `Long` object wrapper seamlessly safeguards the codebase against crashing when elements explicitly match boundary minimum markers like `Integer.MIN_VALUE`.

#### 2. Bounded TreeSet Window Optimization
* **Strengths:** Highly compact codebase. Since `TreeSet` automatically handles deduplication and keeps elements in sorted order natively, it completely eliminates manual index swapping code blocks.
* **Time Complexity Proof:** Although a full `TreeSet` typically costs $O(\log N)$ per insertion, capping the size to a maximum of 3 elements restricts tree operations to a constant performance cost of $O(\log 3) \rightarrow O(1)$. Hence, total loop execution scales in absolute linear $O(N)$ time.

#### 3. Java 8 Pure Stream Extraction Pipeline
* **Strengths:** Superb code layout transparency. The data filter pipeline explicitly documents the exact sequence of technical specifications (Deduplicate $\rightarrow$ Sort Descending $\rightarrow$ Extract Offset Boundary).
* **Performance Penalties:** Lacks short-circuiting. The underlying engine must sort the *entire* unique array footprint via `Comparator.reverseOrder()`, dragging down performance profiles on vast, unarranged enterprise input sets.
