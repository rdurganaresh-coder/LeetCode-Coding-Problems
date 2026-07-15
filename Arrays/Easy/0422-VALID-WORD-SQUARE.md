# LeetCode 422: Valid Word Square

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Matrix &nbsp;&nbsp; 🏷️ String

---

## 📝 Problem Description

Given an array of strings `words`, return `true` if it forms a valid **word square**.

A sequence of strings forms a valid word square if the $k^{th}$ row and $k^{th}$ column read the exact same string, where $0 \le k < \max(\text{numRows}, \text{numColumns})$.

### Examples

**Example 1:**
* **Input:** `words = ["abcd", "bnrt", "crmy", "dtye"]`
* **Output:** `true`
* **Explanation:** 
  * 1st row: `"abcd"`, 1st column: `"abcd"`
  * 2nd row: `"bnrt"`, 2nd column: `"bnrt"`
  * 3rd row: `"crmy"`, 3rd column: `"crmy"`
  * 4th row: `"dtye"`, 4th column: `"dtye"`
  Since every row-column pair matches identically, it is a valid word square.

**Example 2:**
* **Input:** `words = ["abcd", "bnrt", "crm", "dt"]`
* **Output:** `true`
* **Explanation:** 
  * 1st row: `"abcd"`, 1st column: `"abcd"`
  * 2nd row: `"bnrt"`, 2nd column: `"bnrt"`
  * 3rd row: `"crm"`, 3rd column: `"crm"`
  * 4th row: `"dt"`, 4th column: `"dt"`
  Note that jagged rows are allowed as long as the structural matrix shape remains perfectly symmetric across the main diagonal.

**Example 3:**
* **Input:** `words = ["ball", "area", "read", "lady"]`
* **Output:** `false`
* **Explanation:** 
  * 3rd row: `"read"`, 3rd column: `"lead"` (`words[2][2]` is 'a' but `words[2][2]` matches, however column 2 reads 'b','r','e','d' vs 'b','a','r','l').
  * Specifically, the 3rd row is `"read"` while the 3rd column reads `"lead"`. Since they differ, it returns false.

### Constraints
* $1 \le \text{words.length} \le 500$
* $1 \le \text{words}[i].\text{length} \le 500$
* `words[i]` consists of lowercase English letters.

---

## 💡 Core Patterns Used

### 1. Matrix Asymmetric Short-Circuit Guard Pattern
* **Concept:** Dynamically cross-checking boundary intersections of uneven, jagged nested character arrays.
* **Mechanism:** Iterates across character cells using row-index `r` and column-index `c`. Before extracting characters, it applies defensive length validation checks to catch structural asymmetry early:
  1. If column coordinate `c` points to a row index that doesn't exist ($\text{c} \ge \text{words.size()}$), the shape is asymmetric.
  2. If the current row's length does not match the target cross-reference row's length ($\text{words.get(r).length()} \neq \text{words.get(c).length()}$), the structure is invalid.
  Once these bounds are verified as safe, it checks if $\text{words.get(r).charAt(c)} == \text{words.get(c).charAt(r)}$.

### 2. Stream Predicate Index Alignment
* **Concept:** Compacting multi-dimensional coordinate verification tracks using short-circuiting functional streams.
* **Mechanism:** Converts row ranges into integer pipelines, testing structural layout symmetries via standard stream short-circuits like `.allMatch()`.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal In-Place Index Scan (Optimal $O(N^2)$ Time & $O(1)$ Space)
The ideal implementation pattern for runtime environments. It scans matrix coordinates sequentially using primitive variables, short-circuiting immediately if an structural layout mismatch occurs.

```java
import java.util.List;

public class ValidWordSquareClassic {
    public static boolean isValidWordSquare(List<String> words) {
        if (words == null || words.isEmpty()) return true;

        int numRows = words.size();

        for (int r = 0; r < numRows; r++) {
            String currentWord = words.get(r);
            int wordLength = currentWord.length();

            for (int c = 0; c < wordLength; c++) {
                // Out-of-bounds structural symmetry verification checks
                if (c >= numRows || r >= words.get(c).length()) {
                    return false;
                }

                // Explicit character value verification check
                if (currentWord.charAt(c) != words.get(c).charAt(r)) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

### 2. Java 8 Stream AllMatch Coordinate Mapping ($O(N^2)$ Time)
A clean functional approach that removes nested loop boilerplate code by mapping coordinate queries into declarative index ranges.

```java
import java.util.List;
import java.util.stream.IntStream;

public class ValidWordSquareStream {
    public static boolean isValidWordSquare(List<String> words) {
        if (words == null || words.isEmpty()) return true;

        int numRows = words.size();

        // allMatch short-circuits to false immediately if any predicate rule fails
        return IntStream.range(0, numRows).allMatch(r -> {
            String currentWord = words.get(r);
            int wordLength = currentWord.length();

            return IntStream.range(0, wordLength).allMatch(c -> 
                c < numRows && 
                r < words.get(c).length() && 
                currentWord.charAt(c) == words.get(c).charAt(r)
            );
        });
    }
}
```

### 3. Primitive 2D Flattened Grid Validation (Alternative Architecture)
This approach copies character values into a uniform 2D primitive cache grid up front. It replaces jagged string lookup calls with direct memory reads, maximizing cache efficiency for denser matrices.

```java
import java.util.List;

public class ValidWordSquareGridCache {
    public static boolean isValidWordSquare(List<String> words) {
        if (words == null || words.isEmpty()) return true;

        int n = words.size();
        
        // Quick check: Find the maximum width to verify if the matrix can match square constraints
        int maxLen = 0;
        for (String word : words) {
            maxLen = Math.max(maxLen, word.length());
        }
        if (maxLen != n) return false;

        // Allocate a uniform primitive 2D matrix canvas grid initialized to null character '\0'
        char[][] grid = new char[n][n];
        for (int r = 0; r < n; r++) {
            String word = words.get(r);
            for (int c = 0; c < word.length(); c++) {
                grid[r][c] = word.charAt(c);
            }
        }

        // Run direct, high-speed memory-aligned diagonal mirror comparisons
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] != grid[c][r]) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Short-Circuits Early? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. In-Place Index Scan**| $O(N^2)$ | $O(1)$ | Safe if unshared | **Yes (Instant)** | Fast, lightweight execution profile with zero memory overhead. |
| **2. Stream AllMatch** | $O(N^2)$ | $O(1)$ | Safe | **Yes (Instant)** | Clean declarative representation; introduces minor lambda stack layer overhead. |
| **3. Grid Cache** | $O(N^2)$ | $O(N^2)$ | Safe if unshared | No (Post-allocation) | Eliminates index out-of-bound pointer calculations, but has a larger memory footprint. |

---

### Detailed Evaluation Breakdown

#### 1. Optimal In-Place Index Scan
* **Strengths:** Maximum performance layout efficiency. Accessing characters directly via pointers completely bypasses intermediate heap allocations. In jagged asymmetric matrices, it exits immediately upon identifying the first structural error, preserving CPU cycles.
* **Defensive Boundary Architecture:** Placing `c >= numRows || r >= words.get(c).length()` before char lookup calls perfectly immunizes the system against throwing erratic `StringIndexOutOfBoundsException` failures.

#### 2. Java 8 Stream AllMatch Coordinate Mapping
* **Strengths:** Exceptional functional visibility. The structural combination of nested `.allMatch()` calls ensures that the stream pipeline maintains critical short-circuit properties, exiting the collection early if a failure is met.
* **Limitations:** Marginally higher call-stack trace overhead compared to traditional loops due to context switching inside sequential functional interfaces.

#### 3. Primitive 2D Flattened Grid Validation
* **Strengths:** High memory access efficiency. Flattening the variable list down into a plain `char[][]` array allows the CPU to load rows into L1/L2 hardware memory caches sequentially, speeding up character checks.
* **Weaknesses:** Suboptimal resource utilization. It allocates memory for the entire matrix grid up front. If the matrix is invalid at the very first index cell (`words[0][1] != words[1][0]`), this solution still pays the full time and memory cost of allocating the $N \times N$ cache grid unnecessarily.
