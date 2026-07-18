# LeetCode 566: Reshape the Matrix

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Matrix** &nbsp;&nbsp; 🏷️ **Simulation**

---

## 📝 Problem Description

In MATLAB, there is a handy function called `reshape` which can reshape an `m x n` matrix into a new one with a different size `r x c` keeping its original data.

You are given an `m x n` matrix `mat` and two integers `r` and `c` representing the row number and column number of the wanted reshaped matrix.

The reshaped matrix should be filled with all the elements of the original matrix in the same **row-reading order** as they were.

If the `reshape` operation with given parameters is possible and legal, output the new reshaped matrix; Otherwise, output the original matrix.

### Examples

**Example 1:**
* **Input:** `mat = [,[3,4]], r = 1, c = 4`
* **Output:** `[[1,2,3,4]]`
* **Explanation:** The 2x2 matrix is flattened out row-by-row into a single 1x4 horizontal grid.

**Example 2:**
* **Input:** `mat = [,[3,4]], r = 2, c = 4`
* **Output:** `[,[3,4]]`
* **Explanation:** A 2x2 matrix has 4 elements total. It is mathematically impossible to fill a 2x4 matrix (which requires 8 elements) with only 4 elements. Thus, the original matrix is returned unmodified.

### Constraints
* `m == mat.length`
* `n == mat[i].length`
* $1 \le m, n \le 100$
* $-1000 \le \text{mat}[i][j] \le 1000$
* $1 \le r, c \le 300$

---

## 💡 Core Patterns Used

### 1. Flattened Direct 1D Index Mapping Pattern
* **Concept:** Translating two-dimensional grid coordinates into a singular, virtual 1D array index sequence to eliminate data buffering overhead.
* **Mechanism:** A matrix of size $M \times N$ has exactly $M \times N$ total elements. Any 2D coordinate cell $(r_{\text{old}}, c_{\text{old}})$ maps to a unique 1D linear index position via:
  $$\text{index} = r_{\text{old}} \times N + c_{\text{old}}$$
  Symmetrically, we can project this virtual 1D index straight into our new target matrix dimensions $(r, c)$ using high-speed modulo and division operations without flattening the elements into a physical array first:
  $$r_{\text{new}} = \text{index} / c$$
  $$c_{\text{new}} = \text{index} \pmod c$$

### 2. Stream-Based Index Transformation Pipeline
* **Concept:** Unrolling a sequential range of indices across matrix cells using functional pipelines.
* **Mechanism:** Generates a flat integer sequence representing the total element counts via `IntStream.range()`. It maps each scalar index onto the respective source and target matrix cell locations using lambda coordinate arithmetic.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Direct Index Mapping (Standard DSA Style)
The definitive production standard approach. It validates dimensions up front and reshapes coordinates linearly using primitive variables to maximize cache hit rates.

```java
public class MatrixReshapeClassic {
    public static int[][] matrixReshape(int[][] mat, int r, int c) {
        if (mat == null || mat.length == 0 || mat[0].length == 0) {
            return mat;
        }

        int m = mat.length;
        int n = mat[0].length;

        // Legal Reshape Verification Check: Total element counts must match perfectly
        if (m * n != r * c) {
            return mat;
        }

        int[][] reshapedMatrix = new int[r][c];
        int totalElements = m * n;

        // Map every element through a virtual 1D coordinate system layout
        for (int i = 0; i < totalElements; i++) {
            // Translate the flat index 'i' back to original source coordinates
            int srcRow = i / n;
            int srcCol = i % n;

            // Translate the flat index 'i' to the fresh target coordinates
            int targetRow = i / c;
            int targetCol = i % c;

            reshapedMatrix[targetRow][targetCol] = mat[srcRow][srcCol];
        }

        return reshapedMatrix;
    }
}
```

### 2. Dual-Pointer Tracking Simulation (Procedural Style)
An alternative approach that avoids division and modulo arithmetic entirely inside the inner loops by maintaining explicit row and column write-head counters.

```java
public class MatrixReshapePointers {
    public static int[][] matrixReshape(int[][] mat, int r, int c) {
        int m = mat.length;
        int n = mat[0].length;

        if (m * n != r * c) return mat;

        int[][] result = new int[r][c];
        int targetRow = 0;
        int targetCol = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                result[targetRow][targetCol++] = mat[i][j];
                
                // When column bounds saturate, wrap back to index 0 and advance the row head
                if (targetCol == c) {
                    targetCol = 0;
                    targetRow++;
                }
            }
        }
        return result;
    }
}
```

### 3. Java 8 Stream Coordinate Transformation Pipeline
A clean functional version that relies on an unrolled integer range stream to map matrix data components without nested structural loop boilerplate.

```java
import java.util.stream.IntStream;

public class MatrixReshapeStream {
    public static int[][] matrixReshape(int[][] mat, int r, int c) {
        int m = mat.length;
        int n = mat[0].length;

        if (m * n != r * c) return mat;

        int[][] result = new int[r][c];

        // Process total element spaces using flat primitive integer range mapping streams
        IntStream.range(0, m * n).forEach(i -> {
            result[i / c][i % c] = mat[i / n][i % n];
        });

        return result;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Memory Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Direct Index Map** | $O(M \times N)$ | $O(1)$ (Excluding output) | Exactly 1 matrix array | Excellent | Highly optimized math mapping; uses division/modulo operations inside the loop. |
| **2. Dual-Pointer** | $O(M \times N)$ | $O(1)$ (Excluding output) | Exactly 1 matrix array | Excellent | Peak execution speed; replaces arithmetic formulas with simple conditional index resets. |
| **3. Stream Pipeline** | $O(M \times N)$ | $O(1)$ (Excluding output) | Low | Good | High declarative clarity; introduces minor lambda context initialization overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Dual-Pointer Simulation Optimization Profile
* **Strengths:** Maximum performance champion. For systems processing high-density graphical grids or matrix streams, replacing division (`/`) and remainder (`%`) logic paths inside loops with plain conditional increments (`targetCol == c`) dramatically decreases CPU instruction cycles. Division is historically one of the most expensive arithmetic instructions at the hardware level.

#### 2. Flattened Direct 1D Index Mapping
* **Strengths:** Superb logic transparency. It models the multi-dimensional array coordinates into an unrolled flat memory track, preventing out-of-bounds index tracking bugs completely.
* **Data Locality:** Because rows are evaluated sequentially, it respects the row-major memory format layout of standard JVM matrix implementations, guaranteeing stellar CPU cache prefetching performance.

#### 3. Java 8 Stream Coordinate Transformation Pipeline
* **Strengths:** Highly compact declarative presentation. It handles the full matrix dataset as an immutable functional matrix space, minimizing local tracking state mutations.
* **Concurrence Options:** This functional pipeline can be securely converted to execute concurrently across massive image transformation frames by adding a `.parallel()` suffix. Because each index calculation path maps to distinct, isolated cell addresses, thread chunks execute independently without data collision race hazards.
