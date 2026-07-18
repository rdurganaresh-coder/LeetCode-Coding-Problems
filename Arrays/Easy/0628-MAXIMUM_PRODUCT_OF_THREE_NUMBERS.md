# LeetCode 628: Maximum Product of Three Numbers

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Math** &nbsp;&nbsp; 🏷️ **Sorting**

---

## 📝 Problem Description

Given an integer array `nums`, *find three numbers whose product is maximum and return the maximum product*.

### Examples

**Example 1:**
* **Input:** `nums = [1,2,3]`
* **Output:** `6`
* **Explanation:** The only choice is to multiply 1, 2, and 3, yielding $1 \times 2 \times 3 = 6$.

**Example 2:**
* **Input:** `nums = [1,2,3,4]`
* **Output:** `24`
* **Explanation:** Choosing the three largest numbers yields $2 \times 3 \times 4 = 24$.

**Example 3:**
* **Input:** `nums = [-10,-10,5,2]`
* **Output:** `500`
* **Explanation:** Multiplying the two smallest negative numbers and the largest positive number yields $(-10) \times (-10) \times 5 = 500$.

### Constraints
* $3 \le \text{nums.length} \le 10^4$
* $-1000 \le \text{nums}[i] \le 1000$

---

## 💡 Core Patterns Used

### 1. Mathematical Sign-Combination Pattern
* **Concept:** Accounting for the property that multiplying two negative numbers yields a positive product.
* **Mechanism:** When sorting an array ascending, the largest product of three numbers can only ever come from two distinct edge-case combinations:
  1. The product of the three absolute largest numbers: $\text{max}_1 \times \text{max}_2 \times \text{max}_3$ (typically at the tail of the array).
  2. The product of the two most extreme negative numbers (at the head of the array) multiplied by the single absolute largest positive number: $\text{min}_1 \times \text{min}_2 \times \text{max}_1$.
  The final solution is always the absolute maximum of these two paths:
  $$\text{Result} = \max(\text{max}_1 \times \text{max}_2 \times \text{max}_3, \;\text{min}_1 \times \text{min}_2 \times \text{max}_1)$$

### 2. Finite-Slot Tracker Window (Single-Pass Scanner)
* **Concept:** Gathering the top 3 maximums and bottom 2 minimums dynamically in a single pass to achieve linear time optimization.
* **Mechanism:** Maintains 5 primitive reference register scalars simultaneously. As the array cursor moves forward, it conditionally drops elements down fixed tracker tiers, bypassing the overhead of full dataset sorting completely.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Finite-Slot Scan (Optimal $O(N)$ Time & $O(1)$ Space)
The absolute industry gold standard approach. It replaces comparison sorting logic arrays with constant-bound sliding scalar tiers, resulting in lightning-fast processing speeds and zero heap allocations.

```java
public class MaxProductThreeClassic {
    public static int maximumProduct(int[] nums) {
        // Track the top 3 absolute maximum values
        int max1 = Integer.MIN_VALUE, max2 = Integer.MIN_VALUE, max3 = Integer.MIN_VALUE;
        // Track the bottom 2 absolute minimum values (to catch pairs of large negative numbers)
        int min1 = Integer.MAX_VALUE, min2 = Integer.MAX_VALUE;

        for (int num : nums) {
            // 1. Update maximum tracker tiers sequentially
            if (num > max1) {
                max3 = max2;
                max2 = max1;
                max1 = num;
            } else if (num > max2) {
                max3 = max2;
                max2 = num;
            } else if (num > max3) {
                max3 = num;
            }

            // 2. Update minimum tracker tiers sequentially
            if (num < min1) {
                min2 = min1;
                min1 = num;
            } else if (num < min2) {
                min2 = num;
            }
        }

        // Compare the two mathematically viable product combinations
        int productAllPositive = max1 * max2 * max3;
        int productTwoNegative = min1 * min2 * max1;

        return Math.max(productAllPositive, productTwoNegative);
    }
}
```

### 2. Standard Dual-Pivot In-Place Sort ($O(N \log N)$ Time & $O(1)$ Space)
The classic textbook DSA approach. It relies on standard sorting layers to arrange the data bounds before evaluating the extreme edge coordinates.

```java
import java.util.Arrays;

public class MaxProductThreeSort {
    public static int maximumProduct(int[] nums) {
        if (nums == null || nums.length < 3) return 0;

        // In-place sort executes via hyper-optimized Dual-Pivot Quicksort
        Arrays.sort(nums);
        int n = nums.length;

        // Choice 1: Product of the three largest numbers at the tail end
        int tailProduct = nums[n - 1] * nums[n - 2] * nums[n - 3];
        // Choice 2: Product of the two lowest values (negatives) and the single absolute largest value
        int headProduct = nums[0] * nums[1] * nums[n - 1];

        return Math.max(tailProduct, headProduct);
    }
}
```

### 3. Java 8 Stateful Stream Accumulator (Pure Functional Style)
This model embeds the slot-tracking state machines inside an isolated custom container class to strictly conform to pure declarative stream reduction pipelines.

```java
import java.util.Arrays;

public class MaxProductThreeStream {
    private static class StreamProductState {
        int max1 = Integer.MIN_VALUE, max2 = Integer.MIN_VALUE, max3 = Integer.MIN_VALUE;
        int min1 = Integer.MAX_VALUE, min2 = Integer.MAX_VALUE;

        public void accept(int num) {
            if (num > max1) { max3 = max2; max2 = max1; max1 = num; }
            else if (num > max2) { max3 = max2; max2 = num; }
            else if (num > max3) { max3 = num; }

            if (num < min1) { min2 = min1; min1 = num; }
            else if (num < min2) { min2 = num; }
        }

        public StreamProductState combine(StreamProductState other) {
            // Combiner fallback logic block to safely handle parallel execution merges
            StreamProductState merged = new StreamProductState();
            int[] candidatesMax = {this.max1, this.max2, this.max3, other.max1, other.max2, other.max3};
            int[] candidatesMin = {this.min1, this.min2, other.min1, other.min2};
            
            for (int m : candidatesMax) merged.accept(m);
            for (int m : candidatesMin) merged.accept(m);
            return merged;
        }
    }

    public static int maximumProduct(int[] nums) {
        if (nums == null || nums.length < 3) return 0;

        StreamProductState state = Arrays.stream(nums)
            .collect(StreamProductState::new, StreamProductState::accept, StreamProductState::combine);

        return Math.max(state.max1 * state.max2 * state.max3, state.min1 * state.min2 * state.max1);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Heap Allocations | Modifies Input Array? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Finite-Slot Scan**| $O(N)$ | $O(1)$ | **Zero (Immune)** | No | Blazing execution speeds; requires managing multiple manual conditional tiers. |
| **2. Standard Sort** | $O(N \log N)$ | $O(1)$ (or $O(\log N)$ internal stack) | **Zero (Immune)** | **Yes (In-Place)** | Extremely short codebase; independent of structural element tracking states. |
| **3. Stateful Stream** | $O(N)$ | $O(1)$ | Low | No | Exceptional object-oriented pipeline encapsulation. |

---

### Detailed Evaluation Breakdown

#### 1. Linear Scaling Performance Benefits
Traditional naive approaches default to full sorting routines ($O(N \log N)$). While completely trivial under small testing boundaries ($N = 10^4$), moving full sorting execution tracks into low-latency stream servers processing arrays with millions of telemetry integers introduces severe latency logs. 

By utilizing the **Finite-Slot Tracker Window**, the algorithm checks each memory block exactly once, compressing execution down to a true linear time performance index ($O(N)$).

#### 2. Hardware Cache Line Optimizations
Implementation 1 processes a plain, unboxed primitive array sequentially. This allows the internal CPU memory unit to activate continuous hardware prefetching, pre-loading adjacent chunks into high-speed L1/L2 data caches. Additionally, since it uses zero helper objects, the method runs with absolutely no Garbage Collection (GC) execution pause constraints.

#### 3. Java 8 Parallelization Dividends
Unlike many sequential state machines, the `StreamProductState` class (Implementation 3) features a fully functional `combine()` block. This allows the pipeline to easily support a `.parallel()` suffix. The framework splits the data pool into separate chunks across available CPU threads, gathers local maximums, and then merges those mini-blocks safely without data corruption or concurrency race conditions.

***

If you want to move on to the next fundamental algorithmic design archetype, or require a customized **JUnit performance verification template script** to analyze raw execution clock metrics, let me know!
