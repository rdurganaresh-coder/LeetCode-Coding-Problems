# LeetCode 598: Range Addition II

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Math** &nbsp;&nbsp; 🏷️ **Simulation**

---

## 📝 Problem Description

You are given an `m x n` matrix `M` initialized with all `0`s and an array of operations `ops`, where `ops[i] = [ai, bi]` means `M[x][y]` should be incremented by one for all `0 <= x < ai` and `0 <= y < bi`.

Count and return *the number of maximum integers in the matrix after performing all the operations*.

### Examples

**Example 1:**
* **Input:** `m = 3, n = 3, ops = [[2,2],[3,3]]`
* **Output:** `4`
* **Explanation:** 
  * The initial matrix is all 0s.
  * After `[2,2]`, the top-left 2x2 grid values become 1.
  * After `[3,3]`, the entire 3x3 grid values increment by 1.
  The maximum integer in the matrix is 2, and there are exactly 4 cells holding this value (the top-left 2x2 section).

**Example 2:**
* **Input:** `m = 3, n = 3, ops = [[2,2],[3,3],[3,3],[3,3],[2,2],[3,3],[3,3],[3,3],[2,2],[3,3],[3,3],[3,3]]`
* **Output:** `4`

**Example 3:**
* **Input:** `m = 3, n = 3, ops = []`
* **Output:** `9`
* **Explanation:** With no operations performed, all 9 cells remain 0. Since 0 is the maximum integer, the total count is $3 \times 3 = 9$.

### Constraints
* $1 \le m, n \le 4 \times 10^4$
* $0 \le \text{ops.length} \le 10^4$
* $\text{ops}[i].\text{length} == 2$
* $1 \le a_i \le m$
* $1 \le b_i \le n$

---

## 💡 Core Patterns Used

### 1. Matrix Intersection Overlap Pattern (Math Minimization)
* **Concept:** Avoiding an explicit $O(M \times N \times \text{ops.length})$ grid manipulation loop by tracking only the boundaries of the overlapping sections.
* **Mechanism:** Every operation starts at coordinate `(0, 0)` and increments a sub-grid of width $a_i$ and height $b_i$. Because every operation consistently targets the top-left corner, the cells that get incremented the most are always inside the intersection of *all* operations. Finding the size of this region simplifies to tracking the minimum row boundary ($a_{\text{min}}$) and minimum column boundary ($b_{\text{min}}$) across the entire operation list.
  $$\text{Maximum Elements Area} = \min(a_1, a_2, \dots, a_k) \times \min(b_1, b_2, \dots, b_k)$$

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Iterative Intersection (Standard DSA Style)
The definitive production standard approach. It loops through the operations list linearly using basic registers, resulting in maximum speed and zero heap allocations.

```java
public class RangeAdditionClassic {
    public static int maxCount(int m, int n, int[][] ops) {
        // If there are no operations, the entire matrix stays at the maximum value (0)
        if (ops == null || ops.length == 0) {
            return m * n;
        }

        int minRow = m;
        int minCol = n;

        // Trace the smallest bounding box across all incoming operations
        for (int[] op : ops) {
            if (op[0] < minRow) {
                minRow = op[0];
            }
            if (op[1] < minCol) {
                minCol = op[1];
            }
        }

        // The intersection area determines the count of maximum values
        return minRow * minCol;
    }
}
```

### 2. Java 8 Declarative Stream Reduction Pipeline
A clean functional version that uses stream operators to extract the minimum row and column dimensions.

```java
import java.util.Arrays;

public class RangeAdditionStream {
    public static int maxCount(int m, int n, int[][] ops) {
        if (ops == null || ops.length == 0) {
            return m * n;
        }

        // Map row coordinates (index 0) and find the lowest value
        int minRow = Arrays.stream(ops)
                           .mapToInt(op -> op[0])
                           .min()
                           .orElse(m);

        // Map column coordinates (index 1) and find the lowest value
        int minCol = Arrays.stream(ops)
                           .mapToInt(op -> op[1])
                           .min()
                           .orElse(n);

        return minRow * minCol;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Heap Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Iterative Scan** | $O(K)$ | $O(1)$ | **Zero (Immune)** | Excellent | Fastest option by far; utilizes raw memory locality. |
| **2. Stream Pipeline** | $O(K)$ | $O(1)$ | Low | Good | High declarative clarity; introduces minor pipeline setup overhead. |

*Where **$K$** is the number of operations inside the `ops` matrix array ($0 \le K \le 10^4$).*

---

### Detailed Evaluation Breakdown

#### 1. Why a Naive Simulation Fails
A brute-force simulation that instantiates a physical matrix of size $4 \times 10^4$ by $4 \times 10^4$ and fills it in will allocate a massive array grid containing up to $1.6 \times 10^9$ cells. This immediately breaks constraint limits, throwing an `OutOfMemoryError` or causing severe processing delays. By using the **Matrix Intersection Overlap Pattern**, we bypass grid creation entirely, slashing time complexity from an unmanageable $O(M \times N \times K)$ down to a lean linear scan of the operations list ($O(K)$).

#### 2. Hardware-Level Speed Benefits (Iterative Simulation)
* **Zero GC Overhead:** Completely avoiding stream allocations or container boxes inside the loops ensures zero objects are requested on the heap. This prevents Garbage Collection (GC) pauses entirely.
* **Cache Line Locality:** Scanning a single contiguous primitive `int[][]` matrix sequentially allows the internal CPU hardware controllers to seamlessly pull forthcoming memory cells into the high-speed L1/L2 cache before execution cycles look them up, minimizing latency.

#### 3. Java 8 Stream Pipeline Nuances
* **Strengths:** High expressive composition. It models the problem requirements into clear pipeline steps (Extract minimum row limit $\rightarrow$ Extract minimum column limit $\rightarrow$ Multiply dimensions), avoiding manual local tracking states entirely.
* **Performance Impact:** Since `mapToInt()` processes primitives directly, it completely avoids autoboxing costs. However, it scans the `ops` array twice—once for rows and once for columns. While minor for $10^4$ entries, it runs slightly slower than the single-pass iterative loop.
