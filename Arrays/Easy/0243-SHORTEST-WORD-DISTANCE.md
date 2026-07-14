# LeetCode 243: Shortest Word Distance

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ String &nbsp;&nbsp; 🏷️ Two Pointers

---

## 📝 Problem Description

Given an array of strings `wordsDict` and two different strings that already exist in the array, `word1` and `word2`, return *the shortest distance between these two words in the list*.

### Examples

**Example 1:**
* **Input:** `wordsDict = ["practice", "makes", "perfect", "coding", "makes"], word1 = "coding", word2 = "practice"`
* **Output:** `3`
* **Explanation:** `"coding"` is at index 3 and `"practice"` is at index 0. The absolute distance is $|3 - 0| = 3$.

**Example 2:**
* **Input:** `wordsDict = ["practice", "makes", "perfect", "coding", "makes"], word1 = "makes", word2 = "coding"`
* **Output:** `1`
* **Explanation:** The closest occurrence of `"makes"` is at index 4 and `"coding"` is at index 3. The absolute distance is $|4 - 3| = 1$.

### Constraints
* $2 \le \text{wordsDict.length} \le 3 \times 10^4$
* $1 \le \text{wordsDict}[i].length \le 10$
* `wordsDict[i]` consists of lowercase English letters.
* `word1` and `word2` are both in `wordsDict`.
* `word1 != word2`.

---

## 💡 Core Patterns Used

### 1. Dynamic Coordinate Distance Convergence (Single Pass)
* **Concept:** Tracking the most recent position coordinates of target items inside a single forward sweep.
* **Mechanism:** Maintains two index location pointers, `idx1` and `idx2`, initialized to `-1`. As we iterate through the list, when `word1` is encountered, `idx1` is updated. When `word2` is encountered, `idx2` is updated. After any update, if both indices are valid ($\ge 0$), the absolute difference $|idx1 - idx2|$ is evaluated against a global minimum tracker variable.

### 2. Stream Index Reduction Closure
* **Concept:** Folding array string matching operations down to an atomic integer state using side-effecting closure tracking variables.
* **Mechanism:** Feeds indices into custom state accumulators that continuously perform lookups against the primitive reference sequence without splitting execution across separate loop blocks.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal Single-Pass Two-Pointer Tracking ($O(N)$ Time & $O(1)$ Space)
The cleanest production standard. It passes across the array exactly once, calculating localized delta adjustments using constant auxiliary space.

```java
public class ShortestWordDistanceClassic {
    public static int shortestDistance(String[] wordsDict, String word1, String word2) {
        int idx1 = -1;
        int idx2 = -1;
        int minDistance = wordsDict.length;

        for (int i = 0; i < wordsDict.length; i++) {
            if (wordsDict[i].equals(word1)) {
                idx1 = i;
            } else if (wordsDict[i].equals(word2)) {
                idx2 = i;
            }

            // If both indices have been located at least once, compute relative distance
            if (idx1 != -1 && idx2 != -1) {
                minDistance = Math.min(minDistance, Math.abs(idx1 - idx2));
            }
        }

        return minDistance;
    }
}
```

### 2. Java 8 Stateful Index Stream Reduction ($O(N)$ Functional Time)
A functional stream pipeline that wraps index states inside a single-pass processing boundary.

```java
import java.util.stream.IntStream;

public class ShortestWordDistanceStream {
    private static class IndexState {
        int idx1 = -1;
        int idx2 = -1;
        int minDistance = Integer.MAX_VALUE;
    }

    public static int shortestDistance(String[] wordsDict, String word1, String word2) {
        IndexState state = IntStream.range(0, wordsDict.length)
            .collect(IndexState::new,
                (acc, i) -> {
                    if (wordsDict[i].equals(word1)) {
                        acc.idx1 = i;
                    } else if (wordsDict[i].equals(word2)) {
                        acc.idx2 = i;
                    }
                    
                    if (acc.idx1 != -1 && acc.idx2 != -1) {
                        acc.minDistance = Math.min(acc.minDistance, Math.abs(acc.idx1 - acc.idx2));
                    }
                },
                (s1, s2) -> {
                    // Combiner fallback logic block
                    s1.minDistance = Math.min(s1.minDistance, s2.minDistance);
                });

        return state.minDistance;
    }
}
```

### 3. Java 8 IntStream Mapping Pipeline
This alternative avoids custom helper classes by streaming array indices, updating an array wrapper closure dynamically, and using stream math to extract the minimum valid value.

```java
import java.util.stream.IntStream;

public class ShortestWordDistanceIndexScan {
    public static int shortestDistance(String[] wordsDict, String word1, String word2) {
        // Index 0 tracks word1, Index 1 tracks word2
        int[] trackingIndices = { -1, -1 };

        return IntStream.range(0, wordsDict.length)
            .map(i -> {
                if (wordsDict[i].equals(word1)) trackingIndices[0] = i;
                else if (wordsDict[i].equals(word2)) trackingIndices[1] = i;

                // Return maximum value if either position remains unassigned
                if (trackingIndices[0] == -1 || trackingIndices[1] == -1) {
                    return Integer.MAX_VALUE;
                }
                return Math.abs(trackingIndices[0] - trackingIndices[1]);
            })
            .min() // Filter out the lowest overall computation
            .orElse(wordsDict.length);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Dict Limit Capacity | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Pass** | $O(N)$ | $O(1)$ | Safe if unshared | Limited only by JVM array memory | Fastest performance profile with zero memory overhead. |
| **2. Stateful Stream**| $O(N)$ | $O(1)$ | Safe | Limited only by JVM array memory | Clean data layout abstraction; introduces minor state wrapper overhead. |
| **3. Index Scan** | $O(N)$ | $O(1)$ | Unsafe for Parallel | Limited only by JVM array memory | Side-effect mutation inside stream prevents parallel scaling. |

---

## 🔍 Detailed Evaluation Breakdown

#### 1. Optimal Single-Pass Two-Pointer Tracking
* **Strengths:** Utterly optimal execution speeds. Avoids any heavy collection parsing or string list wrappers. Because it computes distance properties using simple variables, it takes full advantage of CPU register performance and excellent L1/L2 cache locality.
* **Early Exit Reality:** Note that while it tracks min-distance updates cleanly, it must evaluate the entire array stream because shorter distance possibilities could always hide at the tail end of the data file.

#### 2. Java 8 Stateful Index Stream Reduction
* **Strengths:** Implements modern object-oriented stream formatting. The state accumulator class encapsulates mutation parameters safely, isolating tracking logic from the broader application thread footprint.
* **Weaknesses:** Minor allocation overhead for spawning the initial `IndexState` container wrapper.

#### 3. Java 8 IntStream Mapping Pipeline
* **Strengths:** Highly compact declarative representation. Removes explicit loop control lines entirely by converting calculations into clear index maps.
* **Stream Pitfall:** Strictly bounded to sequential execution profiles. Mutating the external `trackingIndices` array state closure directly inside a stream lambda function means that appending a `.parallel()` suffix will induce data race conditions and corrupt index comparisons.
