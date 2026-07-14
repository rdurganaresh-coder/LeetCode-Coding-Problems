# LeetCode 137: Single Number II

> 🟡 **Medium** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Bit Manipulation**

---

## 📝 Problem Description

Given an integer array `nums` where every element appears **three times** except for one, which appears **exactly once**. Find the single element and return it.

You must implement a solution with a **linear runtime complexity** and use **only constant extra space**.

### Examples

**Example 1:**
* **Input:** `nums = [2,2,3,2]`
* **Output:** `3`

**Example 2:**
* **Input:** `nums = [0,1,0,1,0,1,99]`
* **Output:** `99`

### Constraints
* $1 \le \text{nums.length} \le 3 \times 10^4$
* $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$
* Each element in the array appears three times except for one element which appears only once.

---

## 💡 Core Patterns Used

### 1. Bitwise Digital Counter (Three-State Finite State Machine)
* **Concept:** Tracking the frequency of bits using a custom base-3 bitwise counter.
* **Mechanism:** Since elements appear three times, standard XOR (base-2 cancelling) doesn't work. We maintain two bitmasks, `ones` and `twos`. 
  * `ones` tracks bits that have appeared $1 \pmod 3$ times.
  * `twos` tracks bits that have appeared $2 \pmod 3$ times.
  When a bit appears a third time, it clears out of both masks, effectively resetting to $0 \pmod 3$. At the end, `ones` holds the answer.

### 2. General Bit-Position Summation (Modulo 3)
* **Concept:** Checking every single bit column independently across all array entries.
* **Mechanism:** Loops from bit position 0 to 31. For each position, sums up how many numbers have that specific bit set. If the sum is not a multiple of 3 ($\text{sum} \pmod 3 \neq 0$), that bit belongs to our unique single number.

### 3. Java 8 Stream Bitwise Array Reductions
* **Concept:** Packing positional bit accumulations using a custom state array with functional stream reductions.
* **Mechanism:** Bypasses stateless lambda restrictions by running structural mutations over a fixed primitive tracking grid.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal Bitwise Digital Counter ($O(N)$ Time & $O(1)$ Space)
The absolute gold standard for this problem. Uses bitwise logic gates to transition states smoothly with no branch conditions or heavy arithmetic.

```java
public class SingleNumberIIDigitalCounter {
    public static int singleNumber(int[] nums) {
        int ones = 0, twos = 0;
        
        for (int num : nums) {
            // 'ones' changes based on current input and what was already in 'twos'
            ones = (ones ^ num) & ~twos;
            // 'twos' changes based on current input and what is now in 'ones'
            twos = (twos ^ num) & ~ones;
        }
        
        return ones;
    }
}
```

### 2. Classic Bit-Position Summation ($O(32N) \rightarrow O(N)$ Time & $O(1)$ Space)
An intuitive approach that iterates through all 32 bits of an integer, counting bits column by column.

```java
public class SingleNumberIIBitSum {
    public static int singleNumber(int[] nums) {
        int result = 0;
        
        // Iterate through all 32 possible bit positions
        for (int i = 0; i < 32; i++) {
            int bitSum = 0;
            int bitMask = (1 << i);
            
            for (int num : nums) {
                if ((num & bitMask) != 0) {
                    bitSum++;
                }
            }
            
            // If the bit sum isn't divisible by 3, the unique number has this bit set
            if (bitSum % 3 != 0) {
                result |= bitMask;
            }
        }
        
        return result;
    }
}
```

### 3. Java 8 Stream Bit-Position Reduction ($O(N)$ Functional Time)
Implements bit position mapping over a primitive stream, gathering positional bits cleanly via index folding arrays.

```java
import java.util.Arrays;

public class SingleNumberIIStreamSum {
    public static int singleNumber(int[] nums) {
        // Index 0 to 31 tracks the bit counts for each position
        int[] bitCounts = Arrays.stream(nums)
            .reduce(new int[32], 
                (counts, num) -> {
                    for (int i = 0; i < 32; i++) {
                        if (((num >> i) & 1) == 1) {
                            counts[i]++;
                        }
                    }
                    return counts;
                }, 
                (c1, c2) -> {
                    for (int i = 0; i < 32; i++) c1[i] += c2[i];
                    return c1;
                });

        // Reconstruct the integer from the computed bit sums
        int result = 0;
        for (int i = 0; i < 32; i++) {
            if (bitCounts[i] % 3 != 0) {
                result |= (1 << i);
            }
        }
        return result;
    }
}
```

### 4. Java 8 Pure Stream Digital Counter
Wraps the optimal 3-state state machine inside a custom accumulator container to respect stream formatting pipelines.

```java
import java.util.Arrays;

public class SingleNumberIIStreamCounter {
    private static class CounterState {
        int ones = 0;
        int twos = 0;
    }

    public static int singleNumber(int[] nums) {
        CounterState finalState = Arrays.stream(nums)
            .collect(CounterState::new,
                (state, num) -> {
                    state.ones = (state.ones ^ num) & ~state.twos;
                    state.twos = (state.twos ^ num) & ~state.ones;
                },
                (s1, s2) -> {
                    // Combiner for parallel execution (not used in sequential)
                    s1.ones ^= s2.ones;
                    s1.twos ^= s2.twos;
                });

        return finalState.ones;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | LeetCode Constraint Safe? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Digital Counter** | $O(N)$ | $O(1)$ | Safe if unshared | Yes (Optimal) | Highly optimized bit logic; hardest to read |
| **2. Bit-Position Sum**| $O(32N) \rightarrow O(N)$ | $O(1)$ | Safe if unshared | Yes | Predictable loop bounds; high bit-shifting operations |
| **3. Stream Bit Sum** | $O(32N) \rightarrow O(N)$ | $O(1)$ | Safe | Yes | Hidden array allocations; clean mapping profile |
| **4. Stream Counter** | $O(N)$ | $O(1)$ | Safe | Yes | Object creation for State Container wrapper |

---

### Detailed Evaluation Breakdown

#### 1. Optimal Bitwise Digital Counter
* **Strengths:** Maximum performance index. It processes elements using standard register bit manipulations in exactly a single pass. Zero allocation overhead.
* **Weaknesses:** Low logical readability. Requires a strong background in logic gate transformations to modify or debug safely.

#### 2. Classic Bit-Position Summation
* **Strengths:** Highly readable and straightforward math logic. Scales simply if the frequency requirement changes from three times to any general count $K$ (just check $\text{sum} \pmod K$).
* **Performance Impact:** Constantly executes $32 \times N$ loop operations. While theoretically linear $O(N)$, it runs slower than the single-pass digital counter because it scans the dataset multiple times or shifts continuously.

#### 3. Java 8 Stream Bit-Position Reduction
* **Strengths:** Fully localized mutation container. Easily integrated into enterprise data mapping flows.
* **Weaknesses:** Allocates internal tracking arrays. Mutating the tracking array index counters inside a lambda stream loop can complicate thread boundaries if run concurrently.

#### 4. Java 8 Pure Stream Digital Counter
* **Strengths:** Blends optimal $O(N)$ runtime with clean, readable, declarative code structures.
* **Weaknesses:** Minor allocation penalty for initializing the `CounterState` object tracker at start-up.
