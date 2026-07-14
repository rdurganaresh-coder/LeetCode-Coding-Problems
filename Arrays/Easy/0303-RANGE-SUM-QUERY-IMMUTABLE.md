# LeetCode 303: Range Sum Query - Immutable

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Design &nbsp;&nbsp; 🏷️ Prefix Sum

---

## 📝 Problem Description

Given an integer array `nums`, handle multiple queries of the following type:

Calculate the sum of the elements of `nums` between indices `left` and `right` inclusive where `left <= right`.

Implement the `NumArray` class:
* `NumArray(int[] nums)` Initializes the object with the integer array `nums`.
* `int sumRange(int left, int right)` Returns the sum of the elements of `nums` between indices `left` and `right` inclusive (i.e. `nums[left] + nums[left + 1] + ... + nums[right]`).

### Examples

**Example 1:**
```java
NumArray numArray = new NumArray([-2, 0, 3, -5, 2, -1]);
numArray.sumRange(0, 2); // return (-2) + 0 + 3 = 1
numArray.sumRange(2, 5); // return 3 + (-5) + 2 + (-1) = -1
numArray.sumRange(0, 5); // return (-2) + 0 + 3 + (-5) + 2 + (-1) = -3
```

### Constraints
* $1 \le \text{nums.length} \le 10^4$
* $-10^5 \le \text{nums}[i] \le 10^5$
* $0 \le \text{left} \le \text{right} < \text{nums.length}$
* At most $10^4$ calls will be made to `sumRange`.

---

## 💡 Core Patterns Used

### 1. Prefix Sum Accumulation Pattern
* **Concept:** Precomputing running cumulative additions to turn iterative, linear range queries into instantaneous mathematical subtractions.
* **Mechanism:** Spawns an internal lookup cache array `prefixSums` of size $N + 1$, where `prefixSums[i]` stores the sum of all elements from index $0$ to $i-1$. 
* **The Formula:** The inclusive range sum between `left` and `right` can be resolved instantly in $O(1)$ constant time using the formula:
  $$\text{Sum}(left, right) = \text{prefixSums}[right + 1] - \text{prefixSums}[left]$$

### 2. Primitive Stream Parallel Reduction
* **Concept:** Building tracking caches using stream reduction operations to isolate state mutations securely.
* **Mechanism:** Uses an `IntStream` along with functional mapping operations to securely populate index lookup tables, removing the need for manual pointer adjustments.

---

## 💻 Java & Java 8 Implementations

### 1. Optimized Prefix Sum Cache (Optimal $O(1)$ Query Time)
The gold standard implementation pattern for high-frequency trading logs or database range scanning operations. It requires a slightly longer setup time to ensure that query lookups take zero loop iterations.

```java
public class NumArrayPrefixSum {
    private final int[] prefixSums;

    // O(N) Initialization Time Complexity
    public NumArrayPrefixSum(int[] nums) {
        if (nums == null || nums.length == 0) {
            this.prefixSums = new int[1];
        } else {
            // Allocate N + 1 slots to handle boundary checks cleanly without offset adjustments
            this.prefixSums = new int[nums.length + 1];
            for (int i = 0; i < nums.length; i++) {
                this.prefixSums[i + 1] = this.prefixSums[i] + nums[i];
            }
        }
    }
    
    // O(1) Query Time Complexity
    public int sumRange(int left, int right) {
        // Direct lookups prevent execution looping overhead entirely
        return prefixSums[right + 1] - prefixSums[left];
    }
}
```

### 2. Java 8 Stream Prefix Generator Array
This functional approach uses primitive stream mapping structures to initialize the array and store the running totals across tracking blocks.

```java
import java.util.Arrays;

public class NumArrayStreamInit {
    private final int[] prefixSums;

    // O(N) Initialization Time Complexity
    public NumArrayStreamInit(int[] nums) {
        this.prefixSums = new int[nums.length + 1];
        
        // Use an array wrapper closure to mutate local state within the stream
        int[] runningTotal = { 0 };
        Arrays.stream(nums).forEach(num -> {
            runningTotal[0] += num;
            // Capture intermediate totals continuously into sequential memory slots
            this.prefixSums[runningTotal[0] + 1] = runningTotal[0];
        });
    }

    // O(1) Query Time Complexity
    public int sumRange(int left, int right) {
        return prefixSums[right + 1] - prefixSums[left];
    }
}
```

### 3. On-The-Fly Java 8 Stream Scanner (Brute-Force Baseline)
This approach completely avoids allocating extra memory at startup by calculating range totals on-the-fly during each call. It serves as an informative baseline for systems with low query traffic.

```java
import java.util.stream.IntStream;

public class NumArrayBruteStream {
    private final int[] sourceNums;

    // O(1) Initialization Time Complexity
    public NumArrayBruteStream(int[] nums) {
        this.sourceNums = nums; // Store direct structural array reference
    }

    // O(N) Query Time Complexity
    public int sumRange(int left, int right) {
        // Iterates through the range dynamically on every call
        return IntStream.rangeClosed(left, right)
                        .map(i -> sourceNums[i])
                        .sum();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Initialization Complexity | Query Complexity (`sumRange`) | Space Complexity (Auxiliary) | Ideal System Use-Case Profile | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Prefix Sum**| $O(N)$ | $O(1)$ | $O(N)$ cache allocation | **High-Read / Heavy Query Feeds** | Optimal performance; requires a memory layout up front to maximize lookup speed. |
| **2. Stream Initialization**| $O(N)$ | $O(1)$ | $O(N)$ cache allocation | **High-Read Pipelines** | Clean, declarative startup logic; minor pipeline initialization overhead. |
| **3. Brute-Force Stream**| $O(1)$ | $O(N)$ | $O(1)$ | **Write-Heavy / Rare Lookups** | Zero initialization memory costs, but degrades under frequent queries. |

---

### Detailed Evaluation Breakdown

#### 1. Optimized Prefix Sum Cache
* **Strengths:** Blazing execution efficiency during read queries. By pre-allocating an extra memory layout, calculating sub-segment values bypasses loops entirely, taking full advantage of L1/L2 cache speeds.
* **Memory Optimization:** Initializing the lookup index with an extra element (`N + 1`) ensures that a lookup starting at index `0` works seamlessly, avoiding extra boundary check conditions.

#### 2. Java 8 Stream Prefix Generator Array
* **Strengths:** Implements modern, expressive code formatting. It replaces procedural tracking variables with isolated data layers.
* **Stream Limitation:** The loop mutation closure array (`runningTotal[0]`) depends on a sequential, ordered stream. Appending a `.parallel()` modifier during initialization will introduce race hazards and corrupt the lookup table.

#### 3. On-The-Fly Java 8 Stream Scanner
* **Strengths:** Immediate, constant-time initialization ($O(1)$) with zero auxiliary memory footprints. It prevents memory overhead when processing data arrays that are rarely queried.
* **Performance Impact:** Highly sub-optimal under standard LeetCode constraint conditions. Executing $10^4$ lookups across an array of length $10^4$ balloons worst-case runtime computations up to $10^8$ operations, introducing significant query latency.
