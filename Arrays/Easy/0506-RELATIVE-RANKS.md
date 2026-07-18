# LeetCode 506: Relative Ranks

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Sorting** &nbsp;&nbsp; 🏷️ **Heap (Priority Queue)**

---

## 📝 Problem Description

You are given an integer array `score` of size `n`, where `score[i]` is the score of the $i^{th}$ athlete in a competition. All the scores are guaranteed to be **unique**.

The athletes are placed based on their scores, where the $1^{st}$ place athlete has the highest score, the $2^{nd}$ place athlete has the $2^{nd}$ highest score, and so on. The placement of each athlete determines their rank:

* The $1^{st}$ place athlete's rank is `"Gold Medal"`.
* The $2^{nd}$ place athlete's rank is `"Silver Medal"`.
* The $3^{nd}$ place athlete's rank is `"Bronze Medal"`.
* For the $4^{th}$ place to the $n^{th}$ place athlete, their rank is their placement number (i.e., the $x^{th}$ place athlete's rank is `"x"`).

Return *an array `answer` of size `n` where `answer[i]` is the rank of the $i^{th}$ athlete*.

### Examples

**Example 1:**
* **Input:** `score = [5,4,3,2,1]`
* **Output:** `["Gold Medal","Silver Medal","Bronze Medal","4","5"]`
* **Explanation:** The placements are straightforward ordered sequences from $1^{st}$ down to $5^{th}$.

**Example 2:**
* **Input:** `score = [10,3,8,9,4]`
* **Output:** `["Gold Medal","5","Bronze Medal","Silver Medal","4"]`
* **Explanation:** Sorted standings mapping reads 10 ($1^{st}$), 9 ($2^{nd}$), 8 ($3^{rd}$), 4 ($4^{th}$), 3 ($5^{th}$).

### Constraints
* `n == score.length`
* $1 \le n \le 10^4$
* $0 \le \text{score}[i] \le 10^6$
* All the values in `score` are **unique**.

---

## 💡 Core Patterns Used

### 1. Direct-Addressing Max-Array Indexing Pattern (Count Array)
* **Concept:** Utilizing a plain primitive tracking array to act as a direct map when score values are bound by clear upper limits.
* **Mechanism:** Since the constraints specify $0 \le \text{score}[i] \le 10^6$, an array of size $10^6 + 1$ can store the original input indices directly at the score's array offset location (`mapCache[score[i]] = i`). Iterating backward through the cache from the maximum index allows us to discover athletes in descending order, instantly assigning accurate rank strings back to their original position coordinates without sorting or heap tree-balancing costs.

### 2. Coordinate Pair Sorting (2D Array Mapping)
* **Concept:** Bundling scores together with their original array indices into tracking pairs to sort safely without destroying data placement rules.
* **Mechanism:** Builds a 2D array matrix of size $N \times 2$. Each row stores `[score[i], i]`. Sorting the matrix descending based on index `0` keeps original placement tags intact.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Direct Array Cache (Optimal $O(N + \text{Max\_Value})$ Time & $O(\text{Max\_Value})$ Space)
The absolute performance champion. It eliminates sorting comparison tree paths entirely, resolving ranks through direct lookups.

```java
public class RelativeRanksCache {
    public static String[] findRelativeRanks(int[] score) {
        if (score == null || score.length == 0) return new String[0];

        // Locate maximum value boundary to optimize cache array footprint sizes
        int maxScore = 0;
        for (int s : score) {
            if (s > maxScore) maxScore = s;
        }

        // Initialize a 1-indexed direct offset registry array tracking original placements
        // Storing 'i + 1' lets us reserve 0 as an unvisited/default cell indicator safely
        int[] scoreToOriginalIndex = new int[maxScore + 1];
        for (int i = 0; i < score.length; i++) {
            scoreToOriginalIndex[score[i]] = i + 1;
        }

        String[] result = new String[score.length];
        int placeCounter = 1;

        // Traverse backward from highest possible score down to 0
        for (int s = maxScore; s >= 0; s--) {
            if (scoreToOriginalIndex[s] > 0) {
                int originalIndex = scoreToOriginalIndex[s] - 1; // Reverse offset shift
                
                // Assign rank strings dynamically based on current standing boundaries
                if (placeCounter == 1) {
                    result[originalIndex] = "Gold Medal";
                } else if (placeCounter == 2) {
                    result[originalIndex] = "Silver Medal";
                } else if (placeCounter == 3) {
                    result[originalIndex] = "Bronze Medal";
                } else {
                    result[originalIndex] = String.valueOf(placeCounter);
                }
                placeCounter++;
            }
        }

        return result;
    }
}
```

### 2. Standard Coordinate Pair Sort (Optimal $O(N \log N)$ Time & $O(N)$ Space)
The classic textbook DSA approach. It groups values into sorted index structures, matching systems where data ranges can be arbitrary.

```java
import java.util.Arrays;

public class RelativeRanksPairSort {
    public static String[] findRelativeRanks(int[] score) {
        int n = score.length;
        int[][] pairGrid = new int[n][2];

        for (int i = 0; i < n; i++) {
            pairGrid[i][0] = score[i]; // Store score value
            pairGrid[i][1] = i;        // Track original index coordinate
        }

        // Sort descending based on score value index 0 properties
        Arrays.sort(pairGrid, (a, b) -> Integer.compare(b[0], a[0]));

        String[] result = new String[n];
        for (int i = 0; i < n; i++) {
            int originalIndex = pairGrid[i][1];
            
            if (i == 0) result[originalIndex] = "Gold Medal";
            else if (i == 1) result[originalIndex] = "Silver Medal";
            else if (i == 2) result[originalIndex] = "Bronze Medal";
            else result[originalIndex] = String.valueOf(i + 1);
        }

        return result;
    }
}
```

### 3. Java 8 Stream Index Box Pipeline (Declarative Style)
An exceptionally expressive functional solution that sorts index references descending based on their score mapping closures.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class RelativeRanksStream {
    public static String[] findRelativeRanks(int[] score) {
        int n = score.length;
        
        // Stream index bounds and collect them sorted descending by their score value weights
        List<Integer> sortedIndices = IntStream.range(0, n)
                .boxed()
                .sorted((i, j) -> Integer.compare(score[j], score[i]))
                .collect(Collectors.toList());

        String[] result = new String[n];
        for (int placement = 0; placement < n; placement++) {
            int originalIndex = sortedIndices.get(placement);
            
            if (placement == 0) result[originalIndex] = "Gold Medal";
            else if (placement == 1) result[originalIndex] = "Silver Medal";
            else if (placement == 2) result[originalIndex] = "Bronze Medal";
            else result[originalIndex] = String.valueOf(placement + 1);
        }

        return result;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Allocations | Max Input Score Scale Resilience | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Direct Cache** | $O(N + \text{Max\_Value})$ | $O(\text{Max\_Value})$ | **Zero (Immune)** | Limited (Fails if score $> 10^7$) | Blazing execution speeds; requires high contiguous primitive memory bounds. |
| **2. Pair Sort** | $O(N \log N)$ | $O(N)$ | Low | **Unlimited** | Standard robust sorting logic; independent of high data value boundaries. |
| **3. Stream Pipeline** | $O(N \log N)$ | $O(N)$ | High | **Unlimited** | Outstanding code transparency; introduces object boxing and collection pointer churn overhead. |

---

### Detailed Evaluation Breakdown

#### 1. High-Performance Direct Array Cache Optimization
* **Strengths:** Maximum performance champion. For standard input lengths ($N = 10^4$), skipping standard QuickSort loops entirely compresses lookup speeds into raw execution cycles. Because it maps values inside linear primitive matrices, it benefits from optimal data prefetching.
* **Constraints Warning:** Highly dependent on bounded parameters. While ideal for LeetCode's $10^6$ constraint range, if individual score limits update to numbers like $10^9$, this approach will instantly throw an `OutOfMemoryError` or crash the thread.

#### 2. Standard Coordinate Pair Sort
* **Strengths:** Exceptional architectural elasticity. It safely handles negative values, massive inputs, and arbitrary distribution weights because sorting scales relative to array lengths ($N$) rather than element scale boundaries.

#### 3. Java 8 Stream Index Box Pipeline
* **Strengths:** Exceptional abstract visibility. Sorting array indices directly based on exterior evaluation maps removes the need to create custom 2D wrapper grids manually.
* **Performance Penalties:** Heavy autoboxing overhead. Initializing primitive integer streams via `IntStream.range().boxed()` forces every entry to wrap into heavy `Integer` objects on the heap, increasing garbage collection sweep frequencies on large enterprise feeds.
