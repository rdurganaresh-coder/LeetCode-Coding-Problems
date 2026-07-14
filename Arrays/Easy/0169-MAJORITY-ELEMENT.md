# LeetCode 169: Majority Element

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Divide and Conquer &nbsp;&nbsp; 🏷️ Counting

---

## 📝 Problem Description

Given an array `nums` of size `n`, return the *majority element*.

The majority element is the element that appears more than `⌊n / 2⌋` times. You may assume that the majority element always exists in the array.

### Examples

**Example 1:**
* **Input:** `nums = [3,2,3]`
* **Output:** `3`

**Example 2:**
* **Input:** `nums = [2,2,1,1,1,2,2]`
* **Output:** `2`

### Constraints
* $n == \text{nums.length}$
* $1 \le n \le 5 \times 10^4$
* $-10^9 \le \text{nums}[i] \le 10^9$

**Follow-up:** Could you solve the problem in linear time $O(n)$ and in $O(1)$ space?

---

## 💡 Core Patterns Used

### 1. Boyer-Moore Voting Pattern
* **Concept:** Cancelling out distinct element pairs until only the absolute dominant variable remains standing.
* **Mechanism:** Maintains a running `candidate` and a `count`. For each element, if the count hits 0, a new candidate is assigned. If the current element matches the candidate, the count is incremented; otherwise, it is decremented. Because the majority element appears more than $\lfloor n / 2 \rfloor$ times, it will always survive the cancellation loops.

### 2. Median Sorting Pattern
* **Concept:** Utilizing position alignment guarantees on sorted structures.
* **Mechanism:** Sorting the data automatically clusters matching elements together. Since the target element spans more than half the array, it is mathematically guaranteed to occupy the absolute middle index position $\lfloor n / 2 \rfloor$.

### 3. Java 8 Stream Grouping & Frequency Reduction
* **Concept:** Dynamically partitioning input elements into value-frequency pairs.
* **Mechanism:** Collects items into a frequency count map via classification collectors, then filters entries based on maximum occurrence values.

---

## 💻 Java & Java 8 Implementations

### 1. Boyer-Moore Voting Algorithm (Optimal $O(N)$ Time & $O(1)$ Space)
The gold standard implementation. It meets the strict follow-up criteria perfectly by processing elements in a single pass with absolutely no heap allocations.

```java
public class MajorityElementBoyerMoore {
    public static int majorityElement(int[] nums) {
        int count = 0;
        int candidate = 0;

        for (int num : nums) {
            // If the count hits zero, update the current candidate to the active element
            if (count == 0) {
                candidate = num;
            }
            // Increment or decrement the balance indicator score accordingly
            count += (num == candidate) ? 1 : -1;
        }

        return candidate;
    }
}
```

### 2. Midpoint Sorting Approach ($O(N \log N)$ Time & $O(1)$ Space)
An exceptionally simple alternative. Sorting the array guarantees that the majority element occupies the midpoint index.

```java
import java.util.Arrays;

public class MajorityElementSorting {
    public static int majorityElement(int[] nums) {
        // Sorts the array in place via dual-pivot Quicksort execution
        Arrays.sort(nums);
        
        // The element at n/2 is guaranteed to be the majority element
        return nums[nums.length / 2];
    }
}
```

### 3. Java 8 Functional Stream Grouping Map ($O(N)$ Time & $O(N)$ Space)
A declarative approach that uses stream collectors to count element frequencies and returns the one with the highest occurrence count.

```java
import java.util.Arrays;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class MajorityElementStreamGroup {
    public static int majorityElement(int[] nums) {
        return Arrays.stream(nums)
            .boxed()
            // Map values to their total occurrence count frequency pairs
            .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
            .entrySet()
            .stream()
            // Extract the map key holding the maximum value entry criteria
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElseThrow(() -> new IllegalArgumentException("Invalid input array"));
    }
}
```

### 4. Stateful Stream Accumulator ($O(N)$ Functional Time & $O(1)$ Space)
This variant fits the linear, allocation-free Boyer-Moore logic into a functional stream structure by tracking state through a primitive custom object.

```java
import java.util.Arrays;

public class MajorityElementStreamAccumulator {
    private static class VoteState {
        int candidate = 0;
        int count = 0;
    }

    public static int majorityElement(int[] nums) {
        VoteState finalState = Arrays.stream(nums)
            .collect(VoteState::new,
                (state, num) -> {
                    if (state.count == 0) {
                        state.candidate = num;
                    }
                    state.count += (num == state.candidate) ? 1 : -1;
                },
                (s1, s2) -> {
                    // Combiner block used for parallel streaming isolation splits
                    if (s1.candidate == s2.candidate) {
                        s1.count += s2.count;
                    } else {
                        s1.candidate = (s1.count >= s2.count) ? s1.candidate : s2.candidate;
                        s1.count = Math.abs(s1.count - s2.count);
                    }
                });

        return finalState.candidate;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Meets Follow-Up Constraints? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Boyer-Moore** | $O(N)$ | $O(1)$ | Safe if unshared | **Yes (Perfect)** | Fast execution speed; relies on an intuitive, specialized mathematical property. |
| **2. Midpoint Sort** | $O(N \log N)$ | $O(1)$ (or $O(\log N)$ internal stack) | Safe if unshared | No ($O(N \log N)$ time) | Dead simple implementation, but sorting introduces performance penalties on large arrays. |
| **3. Stream Grouping**| $O(N)$ | $O(N)$ | Safe | No ($O(N)$ space) | Highly expressive code; suffers from high boxing overhead and hash table tracking memory footprints. |
| **4. Stream Counter** | $O(N)$ | $O(1)$ | Safe | **Yes** | Integrates clean stream processing with optimal complexity, but allocates a minor tracking state object wrapper. |

---

### Detailed Evaluation Breakdown

#### 1. Boyer-Moore Voting Algorithm
* **Strengths:** Maximum possible performance. It runs in exact linear time, processing primitives sequentially with zero extra allocations.
* **Limitations:** It implicitly assumes a majority element *always exists*. If used on an array without a majority element, it will return an arbitrary invalid candidate. (For unverified inputs, a quick second validation pass is required).

#### 2. Midpoint Sorting Approach
* **Strengths:** Modifies no local states or loop flags manually. Extremely short codebase profile that is highly maintainable.
* **Performance Impact:** Sorting forces execution costs to $O(N \log N)$. This can result in significant latency overhead when deployed within performance-critical, real-time application pipelines.

#### 3. Java 8 Functional Stream Grouping Map
* **Strengths:** Declarative and transparent. It visually documents the underlying logic by breaking it down into an intuitive frequency mapping flow.
* **Weaknesses:** Heavy memory overhead. Calling `.boxed()` forces every primitive `int` to map into a heavy `Integer` object wrapper, causing significant garbage collection pressure on large datasets.

#### 4. Stateful Stream Accumulator
* **Strengths:** Combines the optimal $O(1)$ memory blueprint of Boyer-Moore with a modern functional coding style.
* **Parallel Execution Benefit:** The inclusion of an explicit combiner logic block allows this stream pattern to scale safely across multiple processor cores via `.parallel()` transformations without risking data race hazards.
