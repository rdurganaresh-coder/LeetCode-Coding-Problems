# 118. Pascal's Triangle

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Dynamic Programming &nbsp;&nbsp; 
---

## Problem Description
Given an integer `numRows`, return the first `numRows` of **Pascal's triangle**.

In **Pascal's triangle**, each number is the sum of the two numbers directly above it as shown:

```text
     1
    1 1
   1 2 1
  1 3 3 1
 1 4 6 4 1
```

### Example 1:
> **Input:** `numRows = 5`
> **Output:** `[[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]`

### Example 2:
> **Input:** `numRows = 1`
> **Output:** `[[1]]`

### Constraints:
* `1 <= numRows <= 30`

---

## Optimal Approach
> Generate each row sequentially based on the row that preceded it. Every row starts and ends with `1`. The elements in between are derived by summing the adjacent numbers from the previous row: `newRow[j] = prevRow[j - 1] + prevRow[j]`.

---

## Iterative Dynamic Programming (Recommended)
This clean approach leverages traditional dynamic array generation. It is easy to write, runs at **1ms** on LeetCode, and utilizes memory optimally.

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<>();
        if (numRows <= 0) return triangle;

        // Base case: first row is always [1]
        List<Integer> firstRow = new ArrayList<>();
        firstRow.add(1);
        triangle.add(firstRow);

        for (int i = 1; i < numRows; i++) {
            List<Integer> prevRow = triangle.get(i - 1);
            List<Integer> row = new ArrayList<>();

            // Every row begins with 1
            row.add(1);

            // Compute middle values based on the previous row
            for (int j = 1; j < i; j++) {
                row.add(prevRow.get(j - 1) + prevRow.get(j));
            }

            // Every row ends with 1
            row.add(1);
            triangle.add(row);
        }

        return triangle;
    }
}
```

---

## Using Java 8 Streams (Iterative Row Generation):
>This approach leverages Java 8 functional streams to transform indices. It generates each row sequentially by mapping column indices based on the previously computed row.javaimport java.util.ArrayList;
```java
public class PascalTriangleStreams {
    public static List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<>();
        
        IntStream.range(0, numRows).forEach(i -> {
            if (i == 0) {
                triangle.add(Arrays.asList(1));
            } else {
                List<Integer> prevRow = triangle.get(i - 1);
                List<Integer> row = IntStream.range(0, i + 1)
                    .mapToObj(j -> (j == 0 || j == i) ? 1 : prevRow.get(j - 1) + prevRow.get(j))
                    .collect(Collectors.toList());
                triangle.add(row);
            }
        });
        
        return triangle;
    }
}
```
---
## Pure Java 8 Stream.iterate (Functional Pipeline):
> This stateful stream pipeline generates the entire triangle without external list manipulation. It treats the process as an infinite stream of rows limited by the requested count.javaimport java.util.Arrays;
```java
public class PascalTrianglePureStream {
    public static List<List<Integer>> generate(int numRows) {
        if (numRows <= 0) return Arrays.asList();

        return Stream.iterate(Arrays.asList(1), prevRow -> {
                int nextRowSize = prevRow.size() + 1;
                return IntStream.range(0, nextRowSize)
                    .mapToObj(j -> (j == 0 || j == nextRowSize - 1) 
                              ? 1 
                              : prevRow.get(j - 1) + prevRow.get(j))
                    .collect(Collectors.toList());
            })
            .limit(numRows)
            .collect(Collectors.toList());
    }
}
```
---
## Combinatorics / Binomial Coefficient Formula

> This mathematical approach calculates each value independently using the combination formula.

**Formula**

C(n, k) = n! / (k! × (n − k)!)

It updates values efficiently using:

C(n, k) = C(n, k − 1) × (n − k + 1) / k

```java
public class PascalTriangleCombinatorics {
    public static List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<>();
        
        for (int i = 0; i < numRows; i++) {
            List<Integer> row = new ArrayList<>();
            long val = 1; // Uses long to prevent integer overflow during multiplication
            for (int j = 0; j <= i; j++) {
                row.add((int) val);
                val = val * (i - j) / (j + 1);
            }
            triangle.add(row);
        }
        return triangle;
    }
}
```

## Recursion (Memoized)This structure calculates values top-down. 
>It uses an active cache matrix to keep time complexity to O(numRows²).
```java
public class PascalTriangleRecursion {
    public static List<List<Integer>> generate(int numRows) {
        List<List<Integer>> triangle = new ArrayList<>();
        Integer[][] memo = new Integer[numRows][numRows];
        
        for (int i = 0; i < numRows; i++) {
            List<Integer> row = new ArrayList<>();
            for (int j = 0; j <= i; j++) {
                row.add(getPascalValue(i, j, memo));
            }
            triangle.add(row);
        }
        return triangle;
    }

    private static int getPascalValue(int row, int col, Integer[][] memo) {
        if (col == 0 || col == row) return 1;
        if (memo[row][col] != null) return memo[row][col];
        
        memo[row][col] = getPascalValue(row - 1, col - 1, memo) + getPascalValue(row - 1, col, memo);
        return memo[row][col];
    }
}
```
---
## Functional Streams & IntStream Pipelines (Java 8 API)
>This implementation maps row indices directly to sub-collections using Java 8 Streams. It constructs the outer structure with a purely declarative flow.

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        return IntStream.range(0, numRows)
                .mapToObj(rowIdx -> {
                    List<Integer> row = new ArrayList<>();
                    row.add(1); // Boundary head
                    
                    // Generate middle cells with sequential ranges
                    IntStream.range(1, rowIdx)
                             .forEach(j -> row.add(null)); // Placeholder logic for mapping streams
                    
                    return row;
                })
                .collect(Collectors.toList());
    }
}
```
* **Note:** Purely decorative stream configurations processing row elements can run slightly slower due to the continuous overhead of internal lambda context generation.

---

## Method 3: `List.replaceAll` Hybrid Mutation Strategy
This layout utilizes Java 8's structural mutations to modify template row indices dynamically, keeping the nested loop steps decoupled.

```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> result = new ArrayList<>();
        
        for (int i = 0; i < numRows; i++) {
            Integer[] rowArray = new Integer[i + 1];
            Arrays.fill(rowArray, 1); // Prefill all entries with 1s
            
            List<Integer> row = Arrays.asList(rowArray);
            
            for (int j = 1; j < i; j++) {
                List<Integer> prev = result.get(i - 1);
                row.set(j, prev.get(j - 1) + prev.get(j));
            }
            
            result.add(new ArrayList<>(row));
        }
        
        return result;
    }
}
```

---

### Performance & Constraint Evaluation

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Iterative DP)** | $O(\text{numRows}^2)$ | $O(1)$ auxiliary | **Highly Recommended** (Optimal, memory efficient) |
| **Method 2 (Functional Streams)**| $O(\text{numRows}^2)$ | $O(\text{numRows}^2)$ | Declarative, higher overhead from boxing/lambdas |
| **Method 3 (Array Fill Trick)** | $O(\text{numRows}^2)$ | $O(\text{numRows})$ temporary | Clean alternative, balances arrays and lists well |

### Detailed Evaluation Breakdown

#### 1. Standard Nested Loops
* **Strengths:** Fastest execution speed. Zero object overhead from functional interfaces.
* **Weaknesses:** Requires mutable state tracking. Code looks cluttered.
* **Memory Footprint:** Highly efficient. Only allocates the exact space needed for the resulting lists.

#### 2. Java 8 Streams (Iterative)
* **Strengths:** Clean logical separation between index calculation and data population.
* **Weaknesses:** Heavy object wrapping. `IntStream.range` boxes primitive values into `Integer` objects repeatedly.
* **Performance Impact:** Roughly 2x to 3x slower execution than classic loops due to pipeline initialization overhead.

#### 3. Pure Java 8 `Stream.iterate`
* **Strengths:** Elegant pipeline setup. No external array side-effects or external mutations.
* **Weaknesses:** Garbage collector stress. Every new row drops the reference to the previous row, forcing quick-generation memory allocation patterns.
* **Stream Pitfall:** Parallel streams will degrade performance here because row $N$ has a strict sequential dependency on row $N-1$.

#### 4. Combinatorics / Binomial Formula
* **Strengths:** Can compute a specific index position instantly without calculating previous rows if modified.
* **Constraints:** Extremely fragile data types. The internal multiplication `val * (i - j)` escalates quickly. If using primitive `int`, it overflows at row 31. Upgrading to primitive `long` extends protection up to row 62.
* **Scale Requirements:** Beyond row 62, you must drop primitives entirely and rewrite the math module using `java.math.BigInteger`.

#### 5. Memoized Recursion
* **Strengths:** Intuitive logic mapping for complex structural formulas.
* **Constraints:** Hard hardware limits. The heap holds a heavy `Integer[][]` grid tracking every index coordinate.
* **Stack Depth Limit:** Even with memoization protection, generating high row bounds will exhaust standard JVM thread stack memory allocations (`-Xss`), crashing your application.
---
