# LeetCode 645: Set Mismatch

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table** &nbsp;&nbsp; 🏷️ **Bit Manipulation** &nbsp;&nbsp; 🏷️ **Cyclic Sort**

---

## 📝 Problem Description

You have a set of integers which originally contains all the numbers from `1` to `n`. Unfortunately, due to some error, one of the numbers in the set got duplicated to another number in the set, which results in **repetition of one number** and **loss of another number**.

You are given an integer array `nums` representing the data status of this set after the error.

Find the number that occurs twice and the number that is missing and return them in the form of an array `[duplicate, missing]`.

### Examples

**Example 1:**
* **Input:** `nums =`
* **Output:** ``
* **Explanation:** 2 is duplicated, which takes the place of 3. Thus, 2 is the duplicate and 3 is the missing number.

**Example 2:**
* **Input:** `nums =`
* **Output:** ``

### Constraints
* $2 \le \text{nums.length} \le 10^4$
* $1 \le \text{nums}[i] \le 10^4$

---

## 💡 Core Standard DSA Patterns

### 1. In-Place Element-to-Index Sign Inversion Pattern
* **Concept:** Using the element values themselves as dynamic map keys, mutating the array in-place to track state changes.
* **Mechanism:** Since elements are bounded in the range $[1, n]$, each value corresponds to a valid array index when shifted down by 1 ($\text{index} = |nums[i]| - 1$). The algorithm iterates through the array and marks an element as "visited" by flipping the number at its corresponding target index to negative. If the number at that target index is *already* negative, it means we have visited it before, revealing the **duplicate value**. On a second pass, any index that still holds a positive number reveals the **missing element** ($\text{missing value} = \text{index} + 1$).

### 2. Cyclic Sort Pattern (In-Place Swapping)
* **Concept:** Correctly positioning each element at its mathematically natural index position.
* **Mechanism:** Continuously swaps elements until every number $x$ sits comfortably at index $x - 1$ (unless it is a duplicate). A final linear scan identifies mismatches: the value at the mismatched index is the duplicate, and the expected index coordinate value ($index + 1$) is the missing number.

---

## 💻 Standard DSA Implementations

### 1. In-Place Sign Inversion (Optimal $O(N)$ Time & $O(1)$ Space)
The absolute gold standard approach. It operates directly on the input array's sign bits as a built-in state tracker, avoiding extra heap allocations entirely.

```java
public class SetMismatchInversion {
    public static int[] findErrorNums(int[] nums) {
        if (nums == null || nums.length == 0) {
            return new int[0];
        }

        int duplicate = -1;
        int missing = -1;

        // Phase 1: Mark visited indices by converting their corresponding values to negative
        for (int i = 0; i < nums.length; i++) {
            int targetIndex = Math.abs(nums[i]) - 1; // Map element value to 0-indexed coordinate
            
            if (nums[targetIndex] < 0) {
                // If the target slot is already negative, this element is our duplicate
                duplicate = Math.abs(nums[i]);
            } else {
                nums[targetIndex] = -nums[targetIndex]; // Invert sign to track appearance state
            }
        }

        // Phase 2: Any index remaining positive indicates a number that never appeared
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0) {
                missing = i + 1; // Reverse offset translation
                break;
            }
        }

        return new int[]{duplicate, missing};
    }
}
```

### 2. Cyclic Sort Position Matching ($O(N)$ Time & $O(1)$ Space)
An elegant structural pattern. It groups matching variables together on-the-fly and handles duplicates by leaving them in mismatched slots for a final linear cleanup scan.

```java
public class SetMismatchCyclicSort {
    public static int[] findErrorNums(int[] nums) {
        if (nums == null || nums.length == 0) {
            return new int[0];
        }

        int i = 0;
        int n = nums.length;
        
        while (i < n) {
            int correctIndex = nums[i] - 1;
            // Swap elements if the current number is not at its natural index location
            // and ensure we don't get trapped in an infinite loop with duplicate elements
            if (nums[i] != nums[correctIndex]) {
                int temp = nums[i];
                nums[i] = nums[correctIndex];
                nums[correctIndex] = temp;
            } else {
                i++;
            }
        }

        // Identify the index slot mismatch to isolate both values simultaneously
        for (int j = 0; j < n; j++) {
            if (nums[j] != j + 1) {
                return new int[]{nums[j], j + 1};
            }
        }

        return new int[0];
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Object Allocations | Cache Line Locality | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Sign Inversion** | $O(N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Fastest option by far; mutates the input dataset in place. |
| **2. Cyclic Sort** | $O(N)$ | $O(1)$ | **Zero (Immune)** | Excellent | Conceptually clean layout; performs up to $N$ swaps in the worst-case scenario. |

---

### Detailed Evaluation Breakdown

#### 1. Hardware-Level Speed Benefits
* **Zero GC Overhead:** Completely avoiding Java 8 streams, boxed wrapper classes (`Integer`), or map allocations means zero memory allocations are requested on the heap. This prevents Garbage Collection execution pauses entirely, ensuring predictable runtime signatures.
* **Cache Line Locality:** Scanning a single contiguous primitive `int[]` array sequentially allows the internal CPU hardware controllers to seamlessly pull forthcoming memory cells into the high-speed L1/L2 cache before execution cycles look them up, minimizing latency.

#### 2. Why Cyclic Sort and Sign Inversion Trump HashMaps
A naive approach would utilize a `HashMap<Integer, Integer>` or a `HashSet<Integer>` to trace element frequency states. While this successfully solves the problem in $O(N)$ time, it pays a steep memory penalty by forcing space complexity up to $O(N)$. Because it allocates wrapper object layers repeatedly across the heap, performance degrades under garbage collection sweeps. By writing **in-place tracking patterns**, memory usage drops to a static constant profile ($O(1)$) with pristine performance scaling limits.
