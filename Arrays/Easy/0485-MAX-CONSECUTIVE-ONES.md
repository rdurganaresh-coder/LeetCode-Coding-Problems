# LeetCode 485: Max Consecutive Ones

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays**

---

## 📝 Problem Description

Given a binary array `nums`, return *the maximum number of consecutive `1`'s in the array*.

### Examples

**Example 1:**
* **Input:** `nums = [1,1,0,1,1,1]`
* **Output:** `3`
* **Explanation:** The first two digits or the last three digits are consecutive 1s. The maximum number of consecutive 1s is 3.

**Example 2:**
* **Input:** `nums = [1,0,1,1,0,1]`
* **Output:** `2`

### Constraints
* $1 \le \text{nums.length} \le 10^5$
* `nums[i]` is either `0` or `1`.

---

## 💡 Core Patterns Used

### 1. Sequential Reset Accumulator Pattern
* **Concept:** Maintaining a running tally that increases on a matching value and completely clears to zero when hitting a delimiter element.
* **Mechanism:** Tracks a `currentCount` and a `maxCount`. As the array cursor sweeps forward, encountering a `1` increments `currentCount`. Encountering a `0` means the continuous streak has broken; the algorithm updates `maxCount` if `currentCount` has surpassed it, and then instantly resets `currentCount` back to `0`.

### 2. Java 8 Stateful Stream Reduction Closure
* **Concept:** Processing primitive item streams using custom state structures to bypass functional stateless constraints.
* **Mechanism:** Uses `Arrays.stream(nums).reduce()` or an explicit state holder object to run conditional modifications across elements sequentially.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal Single-Pass Scan (Optimal $O(N)$ Time & $O(1)$ Space)
The absolute industry benchmark standard. It processes elements in a single forward pass using primitive register variables, achieving maximum execution efficiency.

```java
public class MaxConsecutiveOnesClassic {
    public static int findMaxConsecutiveOnes(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        int maxCount = 0;
        int currentCount = 0;

        for (int num : nums) {
            if (num == 1) {
                currentCount++;
            } else {
                // Streak broken: update global maximum and reset the counter
                if (currentCount > maxCount) {
                    maxCount = currentCount;
                }
                currentCount = 0;
            }
        }

        // Post-loop check: verify if the longest streak extended up to the absolute end of the array
        return currentCount > maxCount ? currentCount : maxCount;
    }
}
```

### 2. Java 8 Pure Stream String Split ($O(N)$ Time & $O(N)$ Space)
An exceptionally expressive declarative alternative. It transforms the primitive binary input array into a string sequence, splits it using the zero character delimiter, and queries subsegment lengths.

```java
import java.util.Arrays;

public class MaxConsecutiveOnesStringSplit {
    public static int findMaxConsecutiveOnes(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        // Use a continuous builder pipeline to convert binary elements into a clean string stream
        StringBuilder sb = new StringBuilder(nums.length);
        for (int num : nums) {
            sb.append(num);
        }

        // Split by "0" and use streams to identify the maximum length fragment string standing
        return Arrays.stream(sb.toString().split("0"))
                     .mapToInt(String::length)
                     .max()
                     .orElse(0);
    }
}
```

### 3. Java 8 Stateful Stream Counter ($O(N)$ Functional Time & $O(1)$ Space)
This model embeds the sliding tally counters inside an isolated custom state class to comply cleanly with pure functional stream pipelines.

```java
import java.util.Arrays;

public class MaxConsecutiveOnesStatefulStream {
    private static class StreakState {
        int max = 0;
        int current = 0;

        void accept(int num) {
            if (num == 1) {
                current++;
            } else {
                max = Math.max(max, current);
                current = 0;
            }
        }

        void flush() {
            max = Math.max(max, current);
        }

        StreakState combine(StreakState other) {
            // Sequential fallback stream handler
            throw new UnsupportedOperationException("Parallel reductions are not supported for this logic");
        }
    }

    public static int findMaxConsecutiveOnes(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        StreakState finalState = Arrays.stream(nums)
            .collect(StreakState::new, StreakState::accept, StreakState::combine);
        
        finalState.flush(); // Capture any active trailing segment left in flight
        return finalState.max;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Single-Pass Scan** | $O(N)$ | $O(1)$ | Safe if unshared | **Zero (Immune)** | Fastest performance profile by far; utilizes raw memory locality. |
| **2. String Split** | $O(N)$ | $O(N)$ | Safe | High | Highly expressive; creates multiple large intermediate string copies. |
| **3. Stateful Stream**  | $O(N)$ | $O(1)$ | Unsafe | Low | Clean declarative architecture; introduces minor state class initialization overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Optimal Single-Pass Scan
* **Strengths:** Maximum possible performance tuning. Because it processes a plain primitive `int[]` array linearly via register comparisons, it takes full advantage of hardware prefetching and outstanding sequential cache line hits.
* **Trailing Boundary Safety:** Including the final ternary return check (`currentCount > maxCount ? currentCount : maxCount`) ensures that arrays ending with continuous ones (e.g., `[0, 1, 1]`) are recorded accurately without losing track of the terminal buffer data.

#### 2. Java 8 Pure Stream String Split
* **Strengths:** High abstract clarity. The sequence reads like plain English sentences (Map to String $\rightarrow$ Split by Delimiter $\rightarrow$ Extract Max Length).
* **Performance Failure Risks:** Highly memory intensive. Calling `sb.toString().split("0")` forces the JVM to instantiate a massive parent string along with a string array layer containing hundreds of subsegments. This causes severe garbage collection pressure if run on maximum constraint inputs ($N = 10^5$).

#### 3. Java 8 Stateful Stream Counter
* **Strengths:** Exceptional object encapsulation. It completely hides data mutations inside a localized tracker module, preventing unintended side effects across outer execution bounds.
* **Stream Limitation:** Because the logic evaluates state transitions sequentially based on chronological updates, appending a `.parallel()` suffix will induce thread race conditions and corrupt the final tally.


# LeetCode 485: Max Consecutive Ones

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Two Pointers** &nbsp;&nbsp; 🏷️ **Sliding Window**

---

## 📝 Problem Description

Given a binary array `nums`, return *the maximum number of consecutive `1`'s in the array*.

### Examples

**Example 1:**
* **Input:** `nums = [1,1,0,1,1,1]`
* **Output:** `3`
* **Explanation:** The first two digits or the last three digits are consecutive 1s. The maximum number of consecutive 1s is 3.

**Example 2:**
* **Input:** `nums = [1,0,1,1,0,1]`
* **Output:** `2`

### Constraints
* $1 \le \text{nums.length} \le 10^5$
* `nums[i]` is either `0` or `1`.

---

## 💡 Core Standard DSA Patterns

### 1. Sequential Counting State Machine
* **Concept:** Maintaining a single running counter that increments on a target value and resets on a delimiter.
* **Mechanism:** Tracks a `currentWindow` count and a global `maxWindow` count. As a single pointer sweeps forward, encountering a `1` increments the current streak. Encountering a `0` breaks the streak, forces a comparison to update the global maximum, and resets the current counter to `0`.

### 2. Two-Pointer Sliding Window (Dynamic Variant)
* **Concept:** Defining a continuous valid subarray using a left anchor pointer and an exploratory right pointer.
* **Mechanism:** The `right` pointer expands the window forward. When a `0` is encountered, the window becomes invalid, and the `left` pointer jumps directly to `right + 1` to clear the zero element and establish a fresh sliding window baseline.

---

## 💻 Standard DSA Implementations

### 1. Iterative Accumulator Scan (Optimal $O(N)$ Time & $O(1)$ Space)
The cleanest production standard. It processes elements in a single forward pass, optimizing register usage and data locality.

```java
public class MaxConsecutiveOnesAccumulator {
    public static int findMaxConsecutiveOnes(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxConsecutive = 0;
        int currentStreak = 0;

        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 1) {
                currentStreak++;
            } else {
                // Window broken: Update global maximum using basic math comparisons
                if (currentStreak > maxConsecutive) {
                    maxConsecutive = currentStreak;
                }
                currentStreak = 0; // Reset state engine
            }
        }

        // Post-loop check: Guard against streaks extending to the final index boundary
        if (currentStreak > maxConsecutive) {
            maxConsecutive = currentStreak;
        }

        return maxConsecutive;
    }
}
```

### 2. Two-Pointer Sliding Window (Explicit Pointer Boundaries)
This variant frames the problem as an explicit sliding window interval, tracking distances between boundary offsets mathematically.

```java
public class MaxConsecutiveOnesSlidingWindow {
    public static int findMaxConsecutiveOnes(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxWindow = 0;
        int left = 0;

        // 'right' expands the window boundary linearly
        for (int right = 0; right < nums.length; right++) {
            if (nums[right] == 0) {
                // A zero breaks the window. Calculate length of the valid 1-zone behind it
                int currentWindowWidth = right - left;
                if (currentWindowWidth > maxWindow) {
                    maxWindow = currentWindowWidth;
                }
                // Slide the left anchor forward to start a fresh window past the zero element
                left = right + 1;
            }
        }

        // Final boundary evaluation check for windows closing at the absolute tail of the array
        int trailingWindowWidth = nums.length - left;
        if (trailingWindowWidth > maxWindow) {
            maxWindow = trailingWindowWidth;
        }

        return maxWindow;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Memory Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Accumulator Scan** | $O(N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Fastest performance profile; simple and easy to read. |
| **2. Sliding Window** | $O(N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Standardized interview layout; uses pointer arithmetic instead of direct element tallies. |

---

### Detailed Evaluation Breakdown

#### 1. Performance and Hardware Architecture Advantage
Both solutions are highly optimized for modern CPU execution:
* **Zero Object Allocations:** By completely avoiding Java 8 streams, boxed object wrappers (`Integer`), or string arrays, zero objects are created on the heap. This immunizes the codebase from Garbage Collection (GC) execution pauses.
* **Cache Line Locality:** Iterating through a primitive `int[]` array in order allows the CPU's hardware prefetcher to load adjacent blocks into the high-speed L1/L2 data cache ahead of time, minimizing memory access latency.

#### 2. Subarray Boundary Security
* **The Trailing Element Pitfall:** In problems tracking consecutive items, a common error is failing to record a valid sequence if it stretches up to the final index of the array (e.g., `[1, 1, 0, 1, 1]`). 
* **The Solution:** Both patterns incorporate an explicit post-loop comparison block (`nums.length - left` or `currentStreak > maxConsecutive`) to ensure that terminal sequence buffers are captured accurately before returning.
