# LeetCode 463: Island Perimeter

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Matrix** &nbsp;&nbsp; 🏷️ **Depth-First Search** &nbsp;&nbsp; 🏷️ **Breadth-First Search**

---

## 📝 Problem Description

You are given `row x col` `grid` representing a map where `grid[i][j] = 1` represents land and `grid[i][j] = 0` represents water.

Grid cells are connected horizontally/vertically (not diagonally). The grid is completely surrounded by water, and there is exactly one island (i.e., one or more connected land cells).

The island doesn't have "lakes", meaning the water inside isn't connected to the water around the island. One cell is a square with side length 1. The grid is rectangular, width and height don't exceed 100. Determine the perimeter of the island.

### Examples

**Example 1:**
* **Input:** `grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]]`
* **Output:** `16`
* **Explanation:** The perimeter is the 16 yellow stripes in the image (matching the 16 exposed edges of the connected land blocks).

**Example 2:**
* **Input:** `grid = [[1]]`
* **Output:** `4`

**Example 3:**
* **Input:** `grid = [[1,0]]`
* **Output:** `4`

### Constraints
* $\text{row} == \text{grid.length}$
* $\text{col} == \text{grid}[i].\text{length}$
* $1 \le \text{row}, \text{col} \le 100$
* `grid[i][j]` is `0` or `1`.
* There is exactly one island in `grid`.

---

## 💡 Core Patterns Used

### 1. Land-Neighbor Shared Edge Subtraction Pattern (Linear Scan)
* **Concept:** Computing individual square side bounds up-front and subtracting overlapping adjacent surfaces.
* **Mechanism:** Iterates over each grid cell. Every isolated land block (`grid[r][c] == 1`) starts with 4 perimeter edges. The algorithm then checks only two directions—downward neighbor (`r + 1`) and rightward neighbor (`c + 1`). If a neighbor is also land, it implies a shared edge. Since a shared edge removes a perimeter border for *both* adjacent cells, the total perimeter is reduced by exactly 2.
  $$\text{Perimeter} = (\text{Total Land Cells} \times 4) - (\text{Total Shared Intersections} \times 2)$$

### 2. Boundary Condition Traversal (Exposed Edges Strategy)
* **Concept:** Directly validating out-of-bounds or water intersections for all 4 immediate coordinate directions on every land cell.
* **Mechanism:** For each land cell, it queries up, down, left, and right directions. If an adjacent cell is out of bounds or equals 0, that specific direction counts as an exposed shoreline edge and adds 1 to the perimeter counter.

### 3. Java 8 Coordinate Map Reduction Pipeline
* **Concept:** Processing two-dimensional matrix coordinates via functional index streams.
* **Mechanism:** Converts raw matrix dimensions into flattened integer streams, filtering and mapping coordinate states into terminal summation values without using nested loop wrappers.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal In-Place Neighbor Scan (Optimal $O(R \times C)$ Time & $O(1)$ Space)
The absolute industry standard strategy. It avoids stack overhead completely by sweeping rows and columns, subtracting double-counted inner borders as they appear.

```java
public class IslandPerimeterLinearScan {
    public static int islandPerimeter(int[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) return 0;

        int rows = grid.length;
        int cols = grid[0].length;
        int islands = 0;
        int neighbours = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    islands++; // Track total standalone land cells
                    
                    // Look down to find a shared horizontal boundary edge
                    if (r + 1 < rows && grid[r + 1][c] == 1) {
                        neighbours++;
                    }
                    // Look right to find a shared vertical boundary edge
                    if (c + 1 < cols && grid[r][c + 1] == 1) {
                        neighbours++;
                    }
                }
            }
        }

        // Each shared neighbor removes 2 exposed edges from the total perimeter pool
        return (islands * 4) - (neighbours * 2);
    }
}
```

### 2. Exposed Edges Conditional Verification ($O(R \times C)$ Time & $O(1)$ Space)
This variant iterates over every block and evaluates all 4 compass directions, adding 1 directly to the total count whenever water or a matrix border boundary is reached.

```java
public class IslandPerimeterExposedEdges {
    public static int islandPerimeter(int[][] grid) {
        if (grid == null || grid.length == 0) return 0;

        int rows = grid.length;
        int cols = grid[0].length;
        int totalPerimeter = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    // Check top direction boundary
                    if (r == 0 || grid[r - 1][c] == 0) totalPerimeter++;
                    // Check bottom direction boundary
                    if (r == rows - 1 || grid[r + 1][c] == 0) totalPerimeter++;
                    // Check left direction boundary
                    if (c == 0 || grid[r][c - 1] == 0) totalPerimeter++;
                    // Check right direction boundary
                    if (c == cols - 1 || grid[r][c + 1] == 0) totalPerimeter++;
                }
            }
        }
        return totalPerimeter;
    }
}
```

### 3. Java 8 Stream Flattened Index Mapping (Declarative Style)
A clean functional version that uses `IntStream` to convert 2D coordinates into a single linear map pipeline, counting exposed shorelines with zero explicit loop boundaries.

```java
import java.util.stream.IntStream;

public class IslandPerimeterFunctionalStream {
    public static int islandPerimeter(int[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) return 0;

        int rows = grid.length;
        int cols = grid[0].length;

        // Flatten the row coordinates and run calculation mappings over each index cell
        return IntStream.range(0, rows).flatMap(r -> 
            IntStream.range(0, cols).map(c -> {
                if (grid[r][c] == 0) return 0;

                int exposedEdges = 0;
                if (r == 0 || grid[r - 1][c] == 0) exposedEdges++;
                if (r == rows - 1 || grid[r + 1][c] == 0) exposedEdges++;
                if (c == 0 || grid[r][c - 1] == 0) exposedEdges++;
                if (c == cols - 1 || grid[r][c + 1] == 0) exposedEdges++;
                
                return exposedEdges;
            })
        ).sum(); // Terminate by aggregating the mapped integer streams
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Matrix Scale Robustness | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Neighbor Scan** | $O(R \times C)$ | $O(1)$ | Safe if unshared | Excellent | High-speed cache locality; requires checking only 2 direction offsets per loop. |
| **2. Exposed Edges** | $O(R \times C)$ | $O(1)$ | Safe if unshared | Excellent | Intuitive logic layout; evaluates double the number of branching conditions compared to the neighbor scan. |
| **3. Functional Stream**| $O(R \times C)$ | $O(1)$ | Safe | Excellent | High declarative transparency; introduces minor boxing/lambda stack layer initialization overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Optimal In-Place Neighbor Scan
* **Strengths:** Maximum performance optimization. By limiting checks strictly to the *downward* and *rightward* elements, it slashes conditional branch evaluations in half compared to the 4-direction approach. This approach maximizes L1 cache line reading speeds since memory addresses are fetched in a predictable forward sequence.
* **Memory Safety:** It allocates only a few local primitive counters, keeping auxiliary space at absolute zero ($O(1)$) and preventing memory leakage entirely.

#### 2. Exposed Edges Conditional Verification
* **Strengths:** Highly reliable layout that mimics physical edge counts closely. It eliminates the need for any final subtraction calculations since borders are mapped directly on-the-fly.
* **Weaknesses:** Increased branch misprediction penalty risk. Running four separate `if` checking blocks inside the deep inner loop introduces significant branching overhead for the CPU.

#### 3. Java 8 Stream Flattened Index Mapping
* **Strengths:** High expressive composition. It models the matrix canvas as a clean mathematical space and isolates transformation parameters without mutating shared external tracking states.
* **Parallelization Benefit:** This functional style can be safely converted to run concurrently across huge geographical grid sets by appending `.parallel()` to the initial `IntStream`. Because the mapping steps are completely pure and separate, thread segments calculate totals independently before merging them via the terminal `.sum()` step.
