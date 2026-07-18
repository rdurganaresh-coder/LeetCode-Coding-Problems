# LeetCode 561: Array Partition

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Greedy** &nbsp;&nbsp; 🏷️ **Sorting** &nbsp;&nbsp; 🏷️ **Counting Sort**

---

## 📝 Problem Description

Given an integer array `nums` of `2n` integers, group these integers into `n` pairs `(a1, b1), (a2, b2), ..., (an, bn)` such that the sum of `min(ai, bi)` for all `i` is **maximized**. Return *the maximized sum*.

### Examples

**Example 1:**
* **Input:** `nums = [1,4,3,2]`
* **Output:** `4`
* **Explanation:** All possible pairings (ignoring the ordering of elements) are:
  1. `(1, 4), (2, 3) -> min(1, 4) + min(2, 3) = 1 + 2 = 3`
  2. `(1, 3), (2, 4) -> min(1, 3) + min(2, 4) = 1 + 2 = 3`
  3. `(1, 2), (3, 4) -> min(1, 2) + min(3, 4) = 1 + 3 = 4`
  The maximum possible sum is 4.

**Example 2:**
* **Input:** `nums = [6,2,6,5,1,2]`
* **Output:** `9`
* **Explanation:** The optimal pairing is `(1, 2), (2, 5), (6, 6)`. `min(1, 2) + min(2, 5) + min(6, 6) = 1 + 2 + 6 = 9`.

### Constraints
* $1 \le n \le 10^4$
* `nums.length == 2 * n`
* $-10^4 \le \text{nums}[i] \le 10^4$

---

## 💡 Core Patterns Used

### 1. Greedy Pair-Matching Sorting Pattern
* **Concept:** Minimizing the "loss" of value when taking the minimum of two paired items. 
* **Mechanism:** To maximize the sum of minimums, we should pair elements that are closest to each other in value. Sorting the array ascending aligns them automatically. Once sorted, for every adjacent pair `(nums[i], nums[i+1])`, `min(nums[i], nums[i+1])` will always evaluate cleanly to the left element `nums[i]`. Thus, the problem reduces to summing up elements at every even index ($0, 2, 4, \dots, 2n-2$).

### 2. Frequency Bucket Counting (Counting Sort Pattern)
* **Concept:** Avoiding $O(N \log N)$ sorting routines by plotting bounded data limits into fixed-size counting arrays.
* **Mechanism:** Since the constraints guarantee numbers fall strictly between $-10^4$ and $10^4$, we can create a tiny bucket cache array of size $20001$. We offset inputs by adding $10^4$ to map negative numbers to non-negative indices. Iterating over buckets while maintaining a boolean toggle allows us to pick up every alternate sorted value in linear time without formal array reordering.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Counting Sort (Optimal $O(N + \text{Range})$ Time & $O(\text{Range})$ Space)
The absolute performance benchmark champion. It bypasses classic comparison-based sorting entirely, resolving elements via quick direct-address index frequency scans.

```java
public class ArrayPartitionCountingSort {
    public static int arrayPairSum(int[] nums) {
        // Constraints: -10000 <= nums[i] <= 10000 -> Range span size is 20001
        int offset = 10000;
        int[] frequencyBuckets = new int[20001];
        
        // Populate element frequency count registers
        for (int num : nums) {
            frequencyBuckets[num + offset]++;
        }
        
        int maximizedSum = 0;
        boolean selectCurrentElement = true; // Toggle to select every alternate sorted element
        
        // Traverse through the sorted bucket landscape linearly
        for (int i = 0; i < frequencyBuckets.length; i++) {
            while (frequencyBuckets[i] > 0) {
                if (selectCurrentElement) {
                    maximizedSum += (i - offset); // Reverse index offset mapping shift
                }
                selectCurrentElement = !selectCurrentElement; // Alternate picking flag
                frequencyBuckets[i]--;
            }
        }
        
        return maximizedSum;
    }
}
```

### 2. Standard Dual-Pivot In-Place Sort (Optimal $O(N \log N)$ Time & $O(1)$ Space)
The standard industry standard approach. It sorts the primitive values in place using a standard API call, stepping forward via twin index steps.

```java
import java.util.Arrays;

public class ArrayPartitionStandardSort {
    public static int arrayPairSum(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        // In-place sort executes via hyper-optimized Dual-Pivot Quicksort
        Arrays.sort(nums);
        
        int maximizedSum = 0;
        // Increment index pointer by exactly 2 steps to hit even elements exclusively
        for (int i = 0; i < nums.length; i += 2) {
            maximizedSum += nums[i];
        }
        
        return maximizedSum;
    }
}
```

### 3. Java 8 Stream Filter Pipeline ($O(N \log N)$ Time)
A clean functional version that uses stream transformations to sort the primitives and collect the even positional values.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class ArrayPartitionStream {
    public static int arrayPairSum(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        // Sort primitive stream array data components cleanly
        int[] sortedNums = Arrays.stream(nums).sorted().toArray();

        // Stream across indices, filtering for even numbers to aggregate values
        return IntStream.range(0, sortedNums.length)
                .filter(index -> index % 2 == 0)
                .map(index -> sortedNums[index])
                .sum();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Allocations | Max Data Limit Capacity | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Counting Sort** | $O(N + \text{Range})$ | $O(\text{Range})$ | **Zero (Immune)** | Limited (Fails if range $> 10^7$) | Blazing execution speeds; relies strictly on tight element parameter bounds. |
| **2. Standard Sort** | $O(N \log N)$ | $O(1)$ (or $O(\log N)$ internal stack) | **Zero (Immune)** | **Unlimited** | Reliable textbook logic; independent of high data value boundaries. |
| **3. Stream Filter** | $O(N \log N)$ | $O(N)$ | Low | **Unlimited** | Outstanding code transparency; introduces array cloning memory footprints. |

---

### Detailed Evaluation Breakdown

#### 1. High-Performance Counting Sort Optimization
* **Strengths:** Maximum performance optimization. Since $N$ and the range are both tightly bounded, converting search paths from comparison loops ($O(N \log N)$) into a flat linear index sweep ($O(N)$) bypasses CPU scheduling latencies completely. 
* **Memory Safety:** It allocates the small fixed-size buffer exactly once upon activation, creating zero heap footprint changes during processing, completely eliminating garbage collection pressure.

#### 2. Standard Dual-Pivot In-Place Sort
* **Strengths:** Exceptional architectural elasticity. It safely handles values of any arbitrary range or size without risking index array overflows since structural tracking depends exclusively on array size limits ($N$).
* **Side Effects:** Destructive mutation. Modifying the structural sequencing of the input array parameter can introduce bugs downstream if sibling microservices expect elements to match original database log tracks.

#### 3. Java 8 Stream Filter Pipeline
* **Strengths:** High expressive composition. It models the problem requirements into clear pipeline steps (Sort values $\rightarrow$ Filter even indices $\rightarrow$ Aggregate total sum), avoiding manual local tracking states entirely.
* **Performance Penalties:** Slower execution speed. Calling `.toArray()` duplicates the full data payload onto the heap, adding initialization overhead compared to raw register loops.
