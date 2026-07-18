# LeetCode 661: Image Smoother

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Matrix** &nbsp;&nbsp; 🏷️ **Simulation**

---

## 📝 Problem Description

An **image smoother** is a filter of the size `3 x 3` that can be applied to each cell of an image by rounding down the average of the cell and its 8 surrounding cells (i.e., the average of the 9 cells in the blue smoother). If one or more of the surrounding cells of a cell are not present, we do not consider it in the average (i.e., the average of the cells that are present).

Given an `m x n` integer matrix `img` representing the grayscale of an image, return *the matrix after applying the smoother on each cell of it*.

### Examples

**Example 1:**
* **Input:** `img = [,,]`
* **Output:** `[,,]`
* **Explanation:** 
  * For the corner cells, [0,2], [2,0], [2,2]: Each has 4 surrounding cells. Sum = 1+0+0+0 = 1. Average = 1 / 4 = 0.
  * For the edge cells, [1,0], [1,2], [2,1]: Each has 6 surrounding cells. Sum = 1+0+0+0+0+0 = 1. Average = 1 / 6 = 0.
  * For the central cell: It has 9 surrounding cells. Sum = 1+0+0+0+0+0+0+0+0 = 1. Average = 1 / 9 = 0.

**Example 2:**
* **Input:** `img = [,,]`
* **Output:** `[,,]`
* **Explanation:** Applying the 3x3 sliding filter bounds dynamically across all cell coordinates and rounding down the averages generates the smoothed output matrix.

### Constraints
* `m == img.length`
* `n == img[i].length`
* $1 \le m, n \le 200$
* $0 \le \text{img}[i][j] \le 255$

---

## 💡 Core Standard DSA Patterns

### 1. 2D Coordinate Offset Traversal Pattern (8-Directional Array)
* **Concept:** Modeling spatial neighbor lookups using constant index transformation step arrays.
* **Mechanism:** To look up a cell and its 8 immediate surrounding neighbors efficiently without hardcoding separate conditional statements, we define explicit bounds tracking loops or a pre-calculated 2D offset matrix loop ranging from `r - 1` to `r + 1` and `c - 1` to `c + 1`.

### 2. Matrix Boundary Validation Guard
* **Concept:** Intercepting coordinate sweeps dynamically to prevent pointer execution violations over jagged or edge boundaries.
* **Mechanism:** During neighborhood cell aggregation, each calculated row pointer `nr` and column pointer `nc` is filtered through a standard boundary safety check ($0 \le nr < m$ and $0 \le nc < n$). Invalid out-of-bound tracks are safely skipped without processing.

---

## 💻 Standard DSA Implementations

### 1. Standard Simulation with Buffer Allocation ($O(M \times N)$ Time & $O(M \times N)$ Space)
The standard and highly readable production architecture. It allocates a distinct result canvas matrix up front to avoid mutating cell values while the filter is still reading neighbor values.

```java
public class ImageSmootherStandard {
    public static int[][] imageSmoother(int[][] img) {
        if (img == null || img.length == 0 || img[0].length == 0) {
            return new int[0][0];
        }

        int m = img.length;
        int n = img[0].length;
        int[][] smoothImg = new int[m][n];

        // Sweep through every coordinate cell in the image matrix
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                
                int totalSum = 0;
                int validCellCount = 0;

                // 3x3 Sliding Window Filter bounds traversal
                for (int nr = r - 1; nr <= r + 1; nr++) {
                    for (int nc = c - 1; nc <= c + 1; nc++) {
                        
                        // Boundary Guard: Only aggregate if neighbor is safely inside grid limits
                        if (nr >= 0 && nr < m && nc >= 0 && nc < n) {
                            totalSum += img[nr][nc];
                            validCellCount++;
                        }
                    }
                }

                // Integer division automatically rounds down towards zero
                smoothImg[r][c] = totalSum / validCellCount;
            }
        }

        return smoothImg;
    }
}
```

### 2. Bitmask In-Place Optimization ($O(M \times N)$ Time & $O(1)$ Space)
An advanced variant for memory-restricted systems. Since raw pixel grayscale values are strictly bound by $0 \le \text{img}[i][j] \le 255$, they only consume the lowest 8 bits of a standard 32-bit integer. This leaves the higher bits available to cache the newly computed average value in place, achieving constant auxiliary space complexity.

```java
public class ImageSmootherInPlace {
    public static int[][] imageSmoother(int[][] img) {
        if (img == null || img.length == 0 || img[0].length == 0) {
            return img;
        }

        int m = img.length;
        int n = img[0].length;

        // Phase 1: Compute averages and embed them within the higher 8 bits
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                int totalSum = 0;
                int validCellCount = 0;

                for (int nr = r - 1; nr <= r + 1; nr++) {
                    for (int nc = c - 1; nc <= c + 1; nc++) {
                        if (nr >= 0 && nr < m && nc >= 0 && nc < n) {
                            // Extract only the original value using a bitwise mask (lowest 8 bits)
                            totalSum += (img[nr][nc] & 0xFF);
                            validCellCount++;
                        }
                    }
                }

                int average = totalSum / validCellCount;
                // Shift the average value left by 8 bits and bitwise-OR it to store it alongside the original value
                img[r][c] |= (average << 8);
            }
        }

        // Phase 2: Clear out the original values, shifting the calculated averages down into place
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                img[r][c] >>= 8;
            }
        }

        return img;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Object Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Standard Buffer** | $O(M \times N)$ | $O(M \times N)$ | Exactly 1 matrix array | Excellent | Clear separation of logic; requires a temporary memory layout copy. |
| **2. Bitmask In-Place**| $O(M \times N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Maximum performance with zero memory allocation; destructive to original values. |

---

### Detailed Evaluation Breakdown

#### 1. Hardware-Level Speed Benefits (In-Place Bitmasking)
* **Zero GC Overhead:** Completely avoiding Java 8 streams, boxed wrapper classes (`Integer`), or new array allocations means zero memory allocations are requested on the heap during processing. This prevents Garbage Collection execution pauses entirely, ensuring predictable runtime signatures.
* **Cache Line Locality:** Scanning a single contiguous primitive array sequentially allows the internal CPU hardware controllers to seamlessly pull forthcoming memory cells into the high-speed L1/L2 cache before execution cycles look them up, minimizing latency.

#### 2. The Power of Bit Manipulation in 2D Grids
In standard image filtering pipelines, updating a cell's value directly while a filter is sliding across the matrix will corrupt subsequent neighbor reads. Normally, this forces an $O(M \times N)$ auxiliary memory allocation to store results safely. 

However, by leveraging the **Bitmask In-Place Optimization**, we utilize the vacant upper 24 bits of standard 32-bit integer structures to store the new states. This allows the algorithm to run completely in-place with a static constant memory footprint ($O(1)$), optimizing performance for embedded device graphics architectures.
