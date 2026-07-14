# LeetCode 228: Summary Ranges

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Simulation

---

## 📝 Problem Description

You are given a **sorted unique** integer array `nums`.

A range `[a,b]` is the set of all integers from `a` to `b` (inclusive).

Return *the **smallest sorted** list of ranges that **cover all the numbers in the array exactly***. That is, each element of `nums` is covered by exactly one of the ranges, and there is no integer `x` such that `x` is in one of the ranges but not in `nums`.

Each range should be output as:
* `"a->b"` if `a != b`
* `"a"` if `a == b`

### Examples

**Example 1:**
* **Input:** `nums = [0,1,2,4,5,7]`
* **Output:** `["0->2","4->5","7"]`
* **Explanation:** The ranges are:
  * `[0,2]` --> `"0->2"`
  * `[4,5]` --> `"4->5"`
  * `[7,7]` --> `"7"`

**Example 2:**
* **Input:** `nums = [0,2,3,4,6,8,9]`
* **Output:** `["0","2->4","6","8->9"]`
* **Explanation:** The ranges are:
  * `[0,0]` --> `"0"`
  * `[2,4]` --> `"2->4"`
  * `[6,6]` --> `"6"`
  * `[8,9]` --> `"8->9"`

### Constraints
* $0 \le \text{nums.length} \le 20$
* $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$
* All the values of `nums` are **unique**.
* `nums` is sorted in ascending order.

---

## 💡 Core Patterns Used

### 1. Two-Pointer Window Anchor Pattern
* **Concept:** Securing a fixed beginning anchor point while moving a forward exploratory look-ahead index to group continuous blocks.
* **Mechanism:** Sets an anchor index `start`. A secondary runner pointer `i` scans ahead as long as consecutive elements differ by exactly 1 (`nums[i + 1] == nums[i] + 1`). When this sequential relationship breaks, the window boundary closes, a string format is produced, and the anchor point jumps up to begin the next partition block.

### 2. Standard Jump Gap Isolation
* **Concept:** Emitting interval properties immediately by comparing the direct index math difference against sequence offsets.
* **Mechanism:** Evaluates whether elements fit the sequential condition $nums[i] - nums[start] == i - start$. Any mathematical deviation flags a continuity break, isolating individual clusters.

---

## 💻 Java & Java 8 Implementations

### 1. Two-Pointer Window Scan (Optimal $O(N)$ Time & $O(1)$ Space)
The enterprise standard approach. It uses standard loops to process the data sequence in a single linear pass with absolute optimal memory locality.

```java
import java.util.ArrayList;
import java.util.List;

public class SummaryRangesClassic {
    public static List<String> summaryRanges(int[] nums) {
        List<String> result = new ArrayList<>();
        if (nums == null || nums.length == 0) return result;

        int n = nums.length;
        for (int i = 0; i < n; i++) {
            int start = nums[i];
            
            // Advance runner pointer while consecutive numbers are strictly contiguous
            while (i + 1 < n && nums[i + 1] == nums[i] + 1) {
                i++;
            }
            
            // Format output string based on the final computed window span length
            if (start == nums[i]) {
                result.add(String.valueOf(start));
            } else {
                result.add(start + "->" + nums[i]);
            }
        }
        
        return result;
    }
}
```

### 2. Java 8 Stream Multi-Pass Partitioning ($O(N)$ Time)
This approach transforms sequence bounds into localized stream components by detecting interval start-points via index lookup ranges.

```java
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class SummaryRangesStreamIndex {
    public static List<String> summaryRanges(int[] nums) {
        if (nums == null || nums.length == 0) return new ArrayList<>();

        int n = nums.length;

        // Filter and collect all matching index locations where a new interval begins
        List<Integer> startIndices = IntStream.range(0, n)
            .filter(i -> i == 0 || nums[i] != nums[i - 1] + 1)
            .boxed()
            .collect(Collectors.toList());

        // Process the collected intervals sequentially
        return IntStream.range(0, startIndices.size())
            .mapToObj(i -> {
                int startIdx = startIndices.get(i);
                // End index is either the start of the next interval minus 1, or the end of the array
                int endIdx = (i + 1 < startIndices.size()) ? startIndices.get(i + 1) - 1 : n - 1;
                
                return (startIdx == endIdx) 
                    ? String.valueOf(nums[startIdx]) 
                    : nums[startIdx] + "->" + nums[endIdx];
            })
            .collect(Collectors.toList());
    }
}
```

### 3. Stateful Stream Collector Reduction (Pure Functional Style)
This variant leverages a dedicated state container to process primitive elements sequentially inside a functional stream workflow.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class SummaryRangesStatefulStream {
    private static class RangeAccumulator {
        List<String> intervals = new ArrayList<>();
        Integer start = null;
        Integer last = null;

        void accept(int num) {
            if (start == null) {
                start = num;
                last = num;
            } else if (num == last + 1) {
                last = num;
            } else {
                flush();
                start = num;
                last = num;
            }
        }

        void flush() {
            if (start != null) {
                intervals.add(start.equals(last) ? String.valueOf(start) : start + "->" + last);
            }
        }

        RangeAccumulator combine(RangeAccumulator other) {
            // Sequential fallback stream handler
            throw new UnsupportedOperationException("Parallel stream evaluation not supported");
        }
    }

    public static List<String> summaryRanges(int[] nums) {
        if (nums == null || nums.length == 0) return new ArrayList<>();

        RangeAccumulator accumulator = Arrays.stream(nums)
            .collect(RangeAccumulator::new, RangeAccumulator::accept, RangeAccumulator::combine);
        
        accumulator.flush(); // Commit final trailing sequence left in flight
        return accumulator.intervals;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Numerical Safety | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Two-Pointer Window**| $O(N)$ | $O(1)$ | Safe if unshared | Full Integer Limits | Fast register execution; clean pointer-driven optimization layers. |
| **2. Stream Index** | $O(N)$ | $O(N)$ | Safe | Full Integer Limits | Highly expressive code breakdown; creates an intermediate array of index references. |
| **3. Stateful Stream**| $O(N)$ | $O(N)$ | Unsafe | Full Integer Limits | Pure abstract object separation; minor garbage collection object churn. |

---

### Detailed Evaluation Breakdown

#### 1. Two-Pointer Window Scan Approach
* **Strengths:** Maximum performance index. It processes raw memory indices linearly with zero intermediate object instantiation costs, matching low-latency system requirements.
* **Overflow Protection:** Because it uses neighbor offset tracking (`nums[i + 1] == nums[i] + 1`) instead of subtraction math (`nums[i + 1] - nums[i] == 1`), it avoids mathematical integer underflow or overflow crashes when handling edge arrays near `Integer.MAX_VALUE` or `MIN_VALUE`.

#### 2. Java 8 Stream Multi-Pass Partitioning
* **Strengths:** Explicit declarative code flow. Decouples structural detection logic from the final output serialization formatting.
* **Weaknesses:** Requires multiple passes. It iterates across structural ranges once to isolate the starting indices, allocates an tracking payload array list, and then loops a second time to assemble the formatted results.

#### 3. Stateful Stream Collector Reduction
* **Strengths:** Completely encapsulates loop logic inside object frameworks. Conforms perfectly to pure domain model object tracking patterns.
* **Limitations:** Performance overhead. Wrapping values into state objects repeatedly creates allocations on the application heap, making it less optimal for high-throughput execution tracks compared to raw array pointer manipulations.
