# LeetCode 448: Find All Numbers Disappeared in an Array

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table**

---

## 📝 Problem Description

Given an array `nums` of `n` integers where `nums[i]` is in the range `[1, n]`, return *an array of all the integers in the range `[1, n]` that do not appear in `nums`*.

### Examples

**Example 1:**
* **Input:** `nums =`
* **Output:** ``

**Example 2:**
* **Input:** `nums =`
* **Output:** ``

### Constraints
* $n == \text{nums.length}$
* $1 \le n \le 10^5$
* $1 \le \text{nums}[i] \le n$

**Follow-up:** Could you do it without extra space and in $O(n)$ runtime? You may assume that the returned list does not count as extra space.

---

## 💡 Core Patterns Used

### 1. In-Place Element-to-Index Sign Inversion Pattern
* **Concept:** Using the element values themselves as dynamic map keys, mutating the array in-place to track state changes.
* **Mechanism:** Since elements are bounded in the range $[1, n]$, each value corresponds to a valid array index when shifted down by 1 ($\text{index} = |nums[i]| - 1$). The algorithm iterates through the array and marks an element as "visited" by flipping the number at its corresponding target index to negative. On a second pass, any index that still holds a positive number reveals a missing element ($\text{missing value} = \text{index} + 1$).

### 2. Cyclic Sort Pattern (In-Place Swapping)
* **Concept:** Correctly positioning each element at its mathematically natural index position.
* **Mechanism:** Continuously swaps elements until every number $x$ sits comfortably at index $x - 1$ (unless it is a duplicate). A final linear scan identifies mismatches, revealing the missing numbers.

### 3. Java 8 Functional Boolean Array Filter
* **Concept:** Creating an isolated boolean primitive array lookup grid to filter functional streams.
* **Mechanism:** Maps inputs to a fixed-size state map tracker, running clean predicate index reductions via `IntStream` to avoid structural nested loops.

---

## 💻 Java & Java 8 Implementations

### 1. In-Place Sign Inversion (Optimal $O(N)$ Time & $O(1)$ Space)
The absolute gold standard approach. It satisfies the strict follow-up constraints by using the input array's sign bits as a built-in state tracker, avoiding extra heap allocations.

```java
import java.util.ArrayList;
import java.util.List;

public class DisappearedNumbersInversion {
    public static List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> result = new ArrayList<>();
        if (nums == null || nums.length == 0) return result;

        // Phase 1: Mark visited indices by converting their corresponding values to negative
        for (int i = 0; i < nums.length; i++) {
            int targetIndex = Math.abs(nums[i]) - 1; // Map element value to 0-indexed coordinate
            if (nums[targetIndex] > 0) {
                nums[targetIndex] = -nums[targetIndex]; // Invert sign to track appearance state
            }
        }

        // Phase 2: Any index remaining positive indicates a number that never appeared
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0) {
                result.add(i + 1); // Reverse offset translation
            }
        }

        return result;
    }
}
```

### 2. Cyclic Sort Position Matching ($O(N)$ Time & $O(1)$ Space)
An elegant structural pattern. It groups matching variables together on-the-fly and handles duplicates by leaving them in place.

```java
import java.util.ArrayList;
import java.util.List;

public class DisappearedNumbersCyclicSort {
    public static List<List<Integer>> findDisappearedNumbers(int[] nums) {
        List<Integer> result = new ArrayList<>();
        if (nums == null || nums.length == 0) return result;

        int i = 0;
        while (i < nums.length) {
            int correctIndex = nums[i] - 1;
            // Swap elements if the current number is not at its natural index location
            if (nums[i] != nums[correctIndex]) {
                int temp = nums[i];
                nums[i] = nums[correctIndex];
                nums[correctIndex] = temp;
            } else {
                i++;
            }
        }

        // Identify elements that do not match their index slot positions
        for (int j = 0; j < nums.length; j++) {
            if (nums[j] != j + 1) {
                result.add(j + 1);
            }
        }

        return result;
    }
}
```

### 3. Java 8 Stream Lookup Table Filter ($O(N)$ Functional Time)
A highly expressive declarative pipeline. It trades a tiny constant memory allocation for completely side-effect-free code execution blocks.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class DisappearedNumbersStreamFilter {
    public static List<Integer> findDisappearedNumbers(int[] nums) {
        if (nums == null || nums.length == 0) return Arrays.asList();

        // Create a fast, primitive boolean lookup tracking table map
        boolean[] isPresent = new boolean[nums.length + 1];
        Arrays.stream(nums).forEach(num -> isPresent[num] = true);

        // Filter valid index numbers that remain unvisited inside the stream pipeline
        return IntStream.rangeClosed(1, nums.length)
                        .filter(i -> !isPresent[i])
                        .boxed()
                        .collect(Collectors.toList());
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Meets Follow-Up Constraints? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Sign Inversion** | $O(N)$ | $O(1)$ | Safe if unshared | **Yes (Perfect)** | Maximum execution speed; mutates the input dataset in place. |
| **2. Cyclic Sort** | $O(N)$ | $O(1)$ | Safe if unshared | **Yes (Perfect)** | Conceptually clean layout; may perform up to $2N$ swaps in the worst-case scenario. |
| **3. Stream Filter** | $O(N)$ | $O(N)$ | Safe | No ($O(N)$ space allocation) | Exceptional declarative transparency; introduces a minor memory allocation tax. |

---

### Detailed Evaluation Breakdown

#### 1. In-Place Element-to-Index Sign Inversion
* **Strengths:** Maximum performance optimization. Processing raw memory indices inside register loops avoids heap allocation completely, resulting in an optimal runtime footprint.
* **Side Effects:** Destructive mutation. Overwriting the input data with negative values can break downstream components if they expect the original, unmodified dataset.

#### 2. Cyclic Sort Position Matching
* **Strengths:** Incredibly straightforward layout tracking logic. It clusters matching elements together naturally, making it easy to adapt for similar problems (like finding duplicates or the first missing positive number).
* **Performance Impact:** Though it uses nested loops, the pointer `i` only increments when an element is in its correct place. Since each swap places at least one number in its permanent home, the total number of swaps is bounded by $N$, keeping total runtime strictly linear $O(N)$.

#### 3. Java 8 Stream Lookup Table Filter
* **Strengths:** High declarative clarity. It uses a modern syntax profile that isolates state mutations completely, making the code highly readable.
* **Weaknesses:** Fails the $O(1)$ constant auxiliary space limit. Allocating `isPresent` table variables maps tracking to an $O(N)$ memory footprint, and calling `.boxed()` creates object wrappers that increase GC pressure.
