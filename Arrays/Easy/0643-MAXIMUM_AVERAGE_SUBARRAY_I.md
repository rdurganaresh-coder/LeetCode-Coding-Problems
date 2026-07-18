# LeetCode 643: Maximum Average Subarray I

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Sliding Window**

---

## 📝 Problem Description

You are given an integer array `nums` consisting of `n` elements, and an integer `k`.

Find a contiguous subarray whose **length is equal to `k`** that has the maximum average value and return *this value*. Any answer with a calculation error less than $10^{-5}$ will be accepted.

### Examples

**Example 1:**
* **Input:** `nums = [1,12,-5,-6,50,3], k = 4`
* **Output:** `12.75000`
* **Explanation:** The maximum average subarray is `[12, -5, -6, 50]` which has a sum of $12 + (-5) + (-6) + 50 = 51$. The average is $51 / 4 = 12.75$.

**Example 2:**
* **Input:** `nums =, k = 1`
* **Output:** `5.00000`

### Constraints
* $n == \text{nums.length}$
* $1 \le k \le n \le 10^5$
* $-10^4 \le \text{nums}[i] \le 10^4$

---

## 💡 Core Patterns Used

### 1. Fixed-Size Sliding Window Pattern
* **Concept:** Maintaining a running sum of a constant number of contiguous elements ($k$) to avoid redundant recalculation loops.
* **Mechanism:** 
  1. **Phase 1 (Initialization):** Sum up the first $k$ elements to establish the baseline window sum.
  2. **Phase 2 (Sliding):** Slide the window across the array one element at a time from index $k$ to $n-1$. To efficiently calculate the next window's sum without a loop, add the newly entered element (`nums[i]`) and subtract the element that just left the back of the window (`nums[i - k]`):
     $$\text{Sum}_{\text{new}} = \text{Sum}_{\text{old}} + \text{nums}[i] - \text{nums}[i - k]$$
  3. Track the maximum window sum observed, and perform a single floating-point division at the absolute end to return the maximum average.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Sliding Window (Optimal $O(N)$ Time & $O(1)$ Space)
The definitive production standard approach. It processes the primitive elements linearly with zero intermediate object allocation overhead.

```java
public class MaxAverageSubarrayClassic {
    public static double findMaxAverage(int[] nums, int k) {
        // Accumulate the initial sum of the first k elements
        int currentWindowSum = 0;
        for (int i = 0; i < k; i++) {
            currentWindowSum += nums[i];
        }

        int maxWindowSum = currentWindowSum;

        // Slide the fixed window across the rest of the array
        for (int i = k; i < nums.length; i++) {
            // Constant-time update: Add incoming element, remove outgoing element
            currentWindowSum += nums[i] - nums[i - k];
            if (currentWindowSum > maxWindowSum) {
                maxWindowSum = currentWindowSum;
            }
        }

        // Perform division only once at the end to minimize floating-point calculation tax
        return (double) maxWindowSum / k;
    }
}
```

### 2. Java 8 Stateful Stream Reducer ($O(N)$ Functional Time)
This variant implements the exact sliding window logic within a declarative stream layout by mutating a localized primitive state array container closure.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class MaxAverageSubarrayStream {
    public static double findMaxAverage(int[] nums, int k) {
        // Index 0: Running window sum | Index 1: Global maximum window sum
        int[] streamState = new int[2];
        
        // Initialize base window sum
        streamState[0] = IntStream.range(0, k).map(i -> nums[i]).sum();
        streamState[1] = streamState[0];

        // Slide the window using an index stream sequence
        IntStream.range(k, nums.length).forEach(i -> {
            streamState[0] += nums[i] - nums[i - k];
            streamState[1] = Math.max(streamState[1], streamState[0]);
        });

        return (double) streamState[1] / k;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Heap Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Sliding Window** | $O(N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Fastest performance profile; traditional procedural code structure. |
| **2. Stateful Stream**| $O(N)$ | $O(1)$ (Bounded wrapper) | Low | Good | High declarative clarity; relies on array closures inside stream lambdas. |

---

### Detailed Evaluation Breakdown

#### 1. Why Naive Calculations Fail
A naive approach would recalculate the sum of $k$ consecutive elements from scratch at every index position. This triggers a nested loop pattern costing $O(N \times k)$ time complexity. Under worst-case constraint boundaries ($N = 10^5, k = 5 \times 10^4$), calculations balloon up to a massive $5 \times 10^9$ operations, instantly triggering a LeetCode **Time Limit Exceeded (TLE)** error. The **Sliding Window Pattern** scales strictly linearly ($O(N)$) by reducing window transitions to simple constant-time addition and subtraction.

#### 2. Integer Precision Preservation Strategy
Notice that both implementations maintain all window accumulations inside primitive `int` variables during array traversal, rather than continually calculating intermediate averages using `double` types. This design provides two key benefits:
1. It avoids compounding rounding errors caused by floating-point arithmetic.
2. It maximizes raw throughput speed, as integer operations are significantly faster at the CPU register layer compared to floating-point units.

#### 3. Hardware Data Locality Advantages
Implementation 1 steps through memory addresses in a sequential forward order. This pattern is perfectly optimized for the CPU's hardware prefetch controller, which automatically copies forthcoming data segments from RAM straight into high-speed L1/L2 data caches, minimizing memory latency.
