# LeetCode 170: Two Sum III - Data structure design

> 🟢 Easy &nbsp;&nbsp; 🏷️ Hash Table &nbsp;&nbsp; 🏷️ Design &nbsp;&nbsp; 🏷️ Two Pointers

---

## 📝 Problem Description

Design a data structure that accepts a stream of integers and checks if it contains a pair of numbers that sum up to a specific target value.

Implement the `TwoSum` class:
* `TwoSum()` Initializes the object with an empty data collection.
* `void add(int number)` Adds `number` to the internal data collection.
* `boolean find(int value)` Returns `true` if there exists any pair of numbers whose sum is equal to `value`, otherwise returns `false`.

### Examples

**Example 1:**
```java
TwoSum twoSum = new TwoSum();
twoSum.add(1);   // Internal state: [1]
twoSum.add(3);   // Internal state: [1, 3]
twoSum.add(5);   // Internal state: [1, 3, 5]
twoSum.find(4);  // Returns true (1 + 3 = 4)
twoSum.find(7);  // Returns false (No two numbers sum up to 7)
```

### Constraints
* $-10^5 \le \text{number} \le 10^5$
* $-2^{31} \le \text{value} \le 2^{31} - 1$
* At most $10^4$ calls total will be made to `add` and `find`.

---

## 💡 Core Patterns Used

### 1. Hash Table Frequency Map Pattern (Optimized for `add`)
* **Concept:** Sacrificing find execution latency to make record additions near instantaneous.
* **Mechanism:** Stores incoming data inside a frequency map (`HashMap<Integer, Integer>`). When validating a target sum via `find(value)`, it iterates across the unique keys. For each key `num1`, it looks for a complement `num2 = value - num1`. If `num1 == num2`, it checks if the frequency count is $\ge 2$; otherwise, it simply checks if `num2` exists in the map.

### 2. Sorted Matrix Two-Pointer Pattern (Optimized for `find`)
* **Concept:** Balancing dynamic array expansion with fast binary scanning ranges.
* **Mechanism:** Maintains an active sorted collection (`ArrayList`) or inserts data sorted. During a lookup execution, a low pointer and high pointer converge from opposite ends to pinpoint matching bounds in linear time.

### 3. Java 8 Stream Entry Verification
* **Concept:** Running key-set reductions using clean functional predicate pipelines.
* **Mechanism:** Replaces explicit for-loops with a mapping stream pipeline that evaluates value intersections against the underlying frequency registry dynamically.

---

## 💻 Java & Java 8 Implementations

### 1. Frequency Map Tracking (Optimal for Heavy `add` Systems)
This approach prioritizes $O(1)$ operations when writing to the data stream, using the internal map key set during lookup validation steps.

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSumMapOptimized {
    private final Map<Integer, Integer> numCounts;

    public TwoSumMapOptimized() {
        this.numCounts = new HashMap<>();
    }
    
    // O(1) Time complexity
    public void add(int number) {
        this.numCounts.put(number, this.numCounts.getOrDefault(number, 0) + 1);
    }
    
    // O(N) Time complexity
    public boolean find(int value) {
        for (Map.Entry<Integer, Integer> entry : numCounts.entrySet()) {
            int num1 = entry.getKey();
            long complement = (long) value - num1; // Prevent integer underflow/overflow
            
            // Cast down safely once arithmetic bounds are checked
            if (complement < Integer.MIN_VALUE || complement > Integer.MAX_VALUE) {
                continue;
            }
            int target = (int) complement;

            if (num1 == target) {
                // If checking the same number, we need a frequency of at least 2
                if (entry.getValue() >= 2) return true;
            } else {
                if (numCounts.containsKey(target)) return true;
            }
        }
        return false;
    }
}
```

### 2. Java 8 Stream Entry Pipeline (Functional Style)
This variant converts the lookup map entries into a declarative stream sequence, using predicate matching rules to enforce complement conditions.

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSumStreamMap {
    private final Map<Integer, Integer> numCounts = new HashMap<>();

    public void add(int number) {
        numCounts.merge(number, 1, Integer::sum);
    }

    public boolean find(int value) {
        return numCounts.keySet().stream().anyMatch(num -> {
            long complement = (long) value - num;
            if (complement < Integer.MIN_VALUE || complement > Integer.MAX_VALUE) {
                return false;
            }
            int target = (int) complement;
            return (num == target) ? numCounts.get(num) >= 2 : numCounts.containsKey(target);
        });
    }
}
```

### 3. Dynamic Array Two-Pointer Approach (Optimized for Heavy `find` Systems)
This model maintains a sorted layout when adding numbers, allowing the `find` method to run a fast two-pointer contraction layout across primitives.

```java
import java.util.ArrayList;
import java.util.List;

public class TwoSumTwoPointerOptimized {
    private final List<Integer> sortedNums;

    public TwoSumTwoPointerOptimized() {
        this.sortedNums = new ArrayList<>();
    }

    // O(N) Time complexity due to insertion shift constraints
    public void add(int number) {
        int index = 0;
        // Binary search style insertion scan to keep properties sorted inline
        while (index < sortedNums.size() && sortedNums.get(index) < number) {
            index++;
        }
        sortedNums.add(index, number);
    }

    // O(N) Time complexity running across clean primitive allocations
    public boolean find(int value) {
        int left = 0;
        int right = sortedNums.size() - 1;

        while (left < right) {
            long currentSum = (long) sortedNums.get(left) + sortedNums.get(right);
            if (currentSum == value) {
                return true;
            } else if (currentSum < value) {
                left++;
            } else {
                right--;
            }
        }
        return false;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | `add` Complexity | `find` Complexity | Space Complexity | Thread Safety | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Frequency Map** | $O(1)$ | $O(N)$ | $O(N)$ | Unsafe | Tailored for write-heavy streams (logging, auditing logs). |
| **2. Stream Map** | $O(1)$ | $O(N)$ | $O(N)$ | Unsafe | Clean abstract pipeline; tiny execution boxing overhead. |
| **3. Two-Pointer** | $O(N)$ | $O(N)$ | $O(N)$ | Unsafe | Tailored for systems where reads balance array mutations evenly. |

---

### Detailed Evaluation Breakdown

#### 1. Frequency Map Tracking
* **Strengths:** Maximum entry throughput performance. Logging incoming sequence numbers takes constant execution cycles, protecting input pipelines from buffering bottlenecks.
* **Arithmetic Edge Cases:** Utilizing `long complement = (long) value - num1` prevents numerical rollover bugs when `value` approaches `Integer.MAX_VALUE` or `MIN_VALUE`.

#### 2. Java 8 Stream Entry Pipeline
* **Strengths:** Short, clean expression flow that removes manual loop boilerplate by leveraging short-circuit matches (`.anyMatch()`).
* **Limitations:** Marginally slower execution than raw entry iterators due to inner lambda layer stack allocations.

#### 3. Dynamic Array Two-Pointer Approach
* **Strengths:** Excellent evaluation layout when traversing memory addresses. Iterating pointers consecutively across continuous data buffers benefits from CPU cache locality.
* **Weaknesses:** Suboptimal insertion performance. Forcing insertions via sorting positions forces memory shifts across the underlying `ArrayList` memory track, degrading throughput under write-heavy loads.
