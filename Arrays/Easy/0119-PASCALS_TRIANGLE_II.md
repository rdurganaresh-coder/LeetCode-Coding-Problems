# LeetCode 119: Pascal's Triangle II (Java & Java 8 Implementations)
> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Dynamic Programming &nbsp;&nbsp;
---
This document provides optimized solutions for returning the 0-indexed $k^{th}$ row of Pascal's Triangle using standard Java and Java 8 functional styles, followed by a comparative performance evaluation.

---

## Part 1: Implementations

### 1. In-Place Array Optimization (Memory Efficient)
Instead of building the entire triangle, this approach updates a single array from right to left to avoid overwriting values needed for the current iteration.

```java
import java.util.ArrayList;
import java.util.List;

public class PascalRowInPlace {
    public static List<Integer> getRow(int rowIndex) {
        List<Integer> row = new ArrayList<>(rowIndex + 1);
        // Initialize the list with 1 followed by 0s
        row.add(1);
        for (int i = 1; i <= rowIndex; i++) {
            row.add(0);
        }

        // Update elements in reverse to use values from the previous iteration
        for (int i = 1; i <= rowIndex; i++) {
            for (int j = i; j > 0; j--) {
                row.set(j, row.get(j) + row.get(j - 1));
            }
        }
        return row;
    }
}
```

### 2. Linear Time Combinatorics (Optimal $O(N)$ Time)
This approach uses the binomial coefficient formula $C(n, k) = C(n, k-1) \times \frac{n-k+1}{k}$ to compute each element in the row directly in $O(1)$ time per element.

```java
import java.util.ArrayList;
import java.util.List;

public class PascalRowCombinatorics {
    public static List<Integer> getRow(int rowIndex) {
        List<Integer> row = new ArrayList<>();
        long val = 1; // Use long to prevent intermediate integer overflow
        
        for (int j = 0; j <= rowIndex; j++) {
            row.add((int) val);
            val = val * (rowIndex - j) / (j + 1);
        }
        return row;
    }
}
```

### 3. Java 8 Streams with `Stream.iterate` (Functional Row Traversal)
This functional pipeline simulates the row-by-row evolution, keeping track of only the previous row in flight and mapping index ranges dynamically.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class PascalRowStreamIterate {
    public static List<Integer> getRow(int rowIndex) {
        if (rowIndex < 0) return Arrays.asList();

        return Stream.iterate(Arrays.asList(1), prevRow -> {
                int nextRowSize = prevRow.size() + 1;
                return IntStream.range(0, nextRowSize)
                    .mapToObj(j -> (j == 0 || j == nextRowSize - 1) 
                              ? 1 
                              : prevRow.get(j - 1) + prevRow.get(j))
                    .collect(Collectors.toList());
            })
            .skip(rowIndex) // Advance straight to the target index row
            .findFirst()
            .orElse(Arrays.asList(1));
    }
}
```

### 4. Java 8 Functional Array Reduction
This approach uses an array-based loop converted into an `IntStream` reduce operation. It avoids creating list wrappers until the final row state is locked in.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class PascalRowStreamReduce {
    public static List<Integer> getRow(int rowIndex) {
        int[] row = new int[rowIndex + 1];
        row[0] = 1;

        // Use sequential reduction to mutate the primitive array state safely
        IntStream.rangeClosed(1, rowIndex).forEach(i -> {
            for (int j = i; j > 0; j--) {
                row[j] += row[j - 1];
            }
        });

        return Arrays.stream(row).boxed().collect(Collectors.toList());
    }
}
```

---

## Part 2: Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Safe `rowIndex` Limit | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. In-Place Array** | $O(N^2)$ | $O(1)$ | Safe if unshared | ~33 | High mutation reliance |
| **2. Combinatorics** | $O(N)$ | $O(1)$ | Safe | ~61 | Fastest approach but fails on overflow after row 61 |
| **3. `Stream.iterate`**| $O(N^2)$ | $O(N)$ | Safe (Immutable) | ~33 | Heavy GC memory churn (generates all previous rows) |
| **4. Stream Reduce** | $O(N^2)$ | $O(N)$ | Safe if unshared | ~33 | Boxed integer overhead at the tail end |

---

### Detailed Evaluation Breakdown

#### 1. In-Place Array Optimization
* **Strengths:** Excellent memory profile. It allocates exactly one list of size $k+1$ and mutates elements directly.
* **Weaknesses:** Nested iterations result in $O(N^2)$ time complexity, making it less optimal than combinatorics for higher row numbers.

#### 2. Linear Time Combinatorics
* **Strengths:** Highly optimized. It executes in linear time $O(N)$ without needing to compute or store properties from any preceding rows.
* **Constraints:** Intermediate calculation limits. The operation `val * (rowIndex - j)` overflows a standard primitive 32-bit `int` at row 30. Upgrading to a 64-bit `long` extends the limit to row 61. Beyond row 61, `java.math.BigInteger` must be integrated.

#### 3. Java 8 `Stream.iterate`
* **Strengths:** Declarative and highly expressive. Completely side-effect free.
* **Weaknesses:** Defeats the space optimization goal of Pascal's Triangle II. It creates new lists for *every intermediate row* up to $k$, dropping old references immediately. This causes heavy garbage collection overhead.

#### 4. Java 8 Functional Array Reduction
* **Strengths:** Combines primitive execution speeds with standard stream outputs. Keeps mutations hidden inside a localized primitive array pipeline.
* **Performance Impact:** Faster than `Stream.iterate` because it modifies a primitive array under the hood instead of generating new structural object references at every intermediate layer.
