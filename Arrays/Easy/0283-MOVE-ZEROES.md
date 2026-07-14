# LeetCode 283: Move Zeroes

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Two Pointers

---

## 📝 Problem Description

Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.

Note that you must do this **in-place** without making a copy of the array.

### Examples

**Example 1:**
* **Input:** `nums = [0,1,0,3,12]`
* **Output:** `[1,3,12,0,0]`

**Example 2:**
* **Input:** `nums = [0]`
* **Output:** `[0]`

### Constraints
* $1 \le \text{nums.length} \le 10^4$
* $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$

**Follow-up:** Could you minimize the total number of operations done?

---

## 💡 Core Patterns Used

### 1. Two-Pointer Read/Write Head Pattern
* **Concept:** Maintaining an exploratory "read head" pointer along with a trailing conditional "write head" pointer to consolidate target data elements in a single forward pass.
* **Mechanism:** The write pointer (`lastNonZeroFoundAt`) tracks where the next non-zero element belongs. The read pointer (`i`) scans ahead through the array. Whenever a non-zero element is encountered, it is written to the position of the write pointer, and the write pointer increments. After the read pointer reaches the end, all remaining indexes up to the array length are filled with zeros.

### 2. Two-Pointer Swapping Pattern (Optimal Operation Count)
* **Concept:** Swapping values between the read and write heads dynamically rather than overwriting values and backfilling with loops.
* **Mechanism:** Instead of filling trailing indexes with zeros at the tail end of execution, every non-zero element is swapped directly with the element at the write pointer. This automatically pushes zeros toward the back of the collection in real-time.

---

## 💻 Java & Java 8 Implementations

### 1. In-Place Element Shift & Backfill (Standard Approach)
This traditional approach compacts all non-zero values into the front of the array and uses a clean trailing loop to backfill the remaining index slots with zeros.

```java
public class MoveZeroesShift {
    public static void moveZeroes(int[] nums) {
        if (nums == null || nums.length <= 1) return;

        int writeHead = 0;

        // Phase 1: Shift all non-zero elements forward
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != 0) {
                nums[writeHead++] = nums[i];
            }
        }

        // Phase 2: Backfill the remaining index boundaries with zeros
        while (writeHead < nums.length) {
            nums[writeHead++] = 0;
        }
    }
}
```

### 2. Optimized Two-Pointer Swapping (Minimum Operations)
The absolute gold standard optimization for this problem. It satisfies the follow-up prompt by reducing memory writes drastically on arrays dominated by zeros.

```java
public class MoveZeroesSwap {
    public static void moveZeroes(int[] nums) {
        if (nums == null || nums.length <= 1) return;

        int writeHead = 0;
        
        for (int readHead = 0; readHead < nums.length; readHead++) {
            if (nums[readHead] != 0) {
                // Only perform a swap if the pointers have actually diverged
                if (readHead != writeHead) {
                    int temp = nums[readHead];
                    nums[readHead] = nums[writeHead];
                    nums[writeHead] = temp;
                }
                writeHead++;
            }
        }
    }
}
```

### 3. Java 8 Functional Stream Array Partitioning
A declarative approach that leverages primitive streams to construct a fresh layout profile matching the target structure seamlessly.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class MoveZeroesStreamPartition {
    public static void moveZeroes(int[] nums) {
        if (nums == null || nums.length <= 1) return;

        // Filter non-zero items out into a primitive stream sequence array
        int[] nonZeros = Arrays.stream(nums).filter(num -> num != 0).toArray();
        
        // Copy the filtered non-zero elements back into the start of the source array
        System.arraycopy(nonZeros, 0, nums, 0, nonZeros.length);
        
        // Fill the remaining index positions with zeros safely
        Arrays.fill(nums, nonZeros.length, nums.length, 0);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Array Writes (Worst-Case) | In-Place Modifies? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Shift & Backfill** | $O(N)$ | $O(1)$ | $N$ writes | **Yes** | Predictable layout loop; write overhead is constant regardless of zero density. |
| **2. Pointer Swap** | $O(N)$ | $O(1)$ | $2 \times (\text{Count of Non-Zeros})$ | **Yes** | Highly optimized for zero-heavy arrays; minimizes memory access operations. |
| **3. Stream Partition** | $O(N)$ | $O(N)$ | $2N$ writes | No (Allocates helper) | Exceptional declarative purity; violates the strict $O(1)$ auxiliary space constraint. |

---

### Detailed Evaluation Breakdown

#### 1. In-Place Element Shift & Backfill
* **Strengths:** Intuitive logic and high execution determinism. The code will execute exactly $N$ array write operations every time, making it highly stable across diverse test cases.
* **Weaknesses:** Suboptimal for arrays that are already mostly clean or contain very few zeros. For instance, an array containing no zeros still writes every single value to its exact same position unnecessarily.

#### 2. Optimized Two-Pointer Swapping
* **Strengths:** Exceptional operation minimization. By wrapping swaps inside a safety guard (`if (readHead != writeHead)`), it entirely skips writing to memory when an array contains zero elements. If an array consists of $10^4$ non-zeros and only 2 zeros, it performs exactly 2 swaps (4 writes total), maximizing L1 cache write efficiency.
* **Limitations:** Marginally more complex logical branching compared to standard procedural loops.

#### 3. Java 8 Functional Stream Array Partitioning
* **Strengths:** Highly expressive, clean syntax profile that abstracts away variable updates completely.
* **Constraint Violations:** **Fails LeetCode's explicit constant auxiliary space constraint ($O(1)$).** Calling `.toArray()` on a stream allocates a temporary array on the heap tracking up to $N$ entries. This triggers garbage collection pressure on large inputs and should be avoided in low-memory execution tracks.
