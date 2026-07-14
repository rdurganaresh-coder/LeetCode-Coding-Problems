# LeetCode 268: Missing Number

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Bit Manipulation &nbsp;&nbsp; 🏷️ Math

---

## 📝 Problem Description

Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return *the only number in the range that is missing from the array*.

### Examples

**Example 1:**
* **Input:** `nums = [3,0,1]`
* **Output:** `2`
* **Explanation:** `n = 3` since there are 3 numbers, so all numbers are in the range `[0,3]`. 2 is the missing number in the range since it does not appear in `nums`.

**Example 2:**
* **Input:** `nums = [0,1]`
* **Output:** `2`
* **Explanation:** `n = 2` since there are 2 numbers, so all numbers are in the range `[0,2]`. 2 is the missing number in the range since it does not appear in `nums`.

**Example 3:**
* **Input:** `nums = [9,6,4,2,3,5,7,0,1]`
* **Output:** `8`
* **Explanation:** `n = 9` since there are 9 numbers, so all numbers are in the range `[0,9]`. 8 is the missing number in the range since it does not appear in `nums`.

### Constraints
* $n == \text{nums.length}$
* $1 \le n \le 10^4$
* $0 \le \text{nums}[i] \le n$
* All the numbers of `nums` are **unique**.

**Follow-up:** Could you solve it using only $O(1)$ extra space complexity and $O(n)$ runtime complexity?

---

## 💡 Core Patterns Used

### 1. Gauss Formula Math Pattern (Sum Difference)
* **Concept:** Utilizing the arithmetic series formula to calculate the absolute expected sum of the sequence.
* **Mechanism:** The sum of all numbers from $0$ to $n$ is defined by Gauss's formula: $\sum = \frac{n \times (n + 1)}{2}$. By subtracting the actual sum of the array elements from this mathematical expectation, the missing value is revealed instantly in a single step.

### 2. XOR Bit Manipulation Pattern (Index-Value Alignment)
* **Concept:** Leveraging bitwise XOR properties where identical bits cancel out to isolate the missing entry.
* **Mechanism:** XOR satisfies Commutativity ($A \oplus B = B \oplus A$) and Self-Cancellation ($A \oplus A = 0$). By initializing an accumulator to $n$ and sequentially XORing every index and its corresponding array value ($result \oplus i \oplus nums[i]$), all matching elements vanish, leaving only the unique missing number.

### 3. Java 8 Stream Reduction Accumulation
* **Concept:** Folding primitive integer data pipelines down to a singular scalar resolution.
* **Mechanism:** Pipelines array scans directly into terminal summing or bitwise reduction operations to bypass procedural tracking blocks.

---

## 💻 Java & Java 8 Implementations

### 1. Gauss Arithmetic Summation (Optimal $O(N)$ Time & $O(1)$ Space)
The cleanest math-based variation. It calculates the expected total instantly and reduces the array values linearly with no auxiliary memory requirements.

```java
public class MissingNumberGauss {
    public static int missingNumber(int[] nums) {
        int n = nums.length;
        // Calculate the theoretical full sum using the closed-form arithmetic sequence formula
        int expectedSum = n * (n + 1) / 2;
        
        int actualSum = 0;
        for (int num : nums) {
            actualSum += num;
        }
        
        return expectedSum - actualSum;
    }
}
```

### 2. Bitwise XOR Traversal (Optimal $O(N)$ Time & $O(1)$ Space)
The bitwise approach. It completely prevents any potential integer arithmetic overflow limits by shifting individual bits directly inside registers.

```java
public class MissingNumberXOR {
    public static int missingNumber(int[] nums) {
        int xorAccumulator = nums.length;
        
        // Simultaneously XOR both the array index and the value inside a single loop
        for (int i = 0; i < nums.length; i++) {
            xorAccumulator ^= i ^ nums[i];
        }
        
        return xorAccumulator;
    }
}
```

### 3. Java 8 Functional Stream Summation ($O(N)$ Time)
A highly expressive single-line expression that uses standard primitive stream aggregation pipelines to determine the missing sum value.

```java
import java.util.Arrays;

public class MissingNumberStreamSum {
    public static int missingNumber(int[] nums) {
        int n = nums.length;
        // Compute expected sum using math, and evaluate array sum via primitive streams
        return (n * (n + 1) / 2) - Arrays.stream(nums).sum();
    }
}
```

### 4. Java 8 Stream XOR Reduction ($O(N)$ Functional Time)
This variant converts the XOR cancellation loop into a stream reduction strategy, executing parallel computations safely if required.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class MissingNumberStreamXOR {
    public static int missingNumber(int[] nums) {
        // Run continuous stream evaluations combining array stream and index stream operations
        int arrayXor = Arrays.stream(nums).reduce(0, (a, b) -> a ^ b);
        int indexXor = IntStream.rangeClosed(0, nums.length).reduce(0, (a, b) -> a ^ b);
        
        return arrayXor ^ indexXor;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Overflow Vulnerability? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Gauss Sum** | $O(N)$ | $O(1)$ | Safe if unshared | Potential if $n > 65535$ (requires `long`) | Maximum speed using elementary arithmetic logic. |
| **2. Bitwise XOR** | $O(N)$ | $O(1)$ | Safe if unshared | **No (Immune)** | Exceptionally resilient bit logic; ignores data boundaries entirely. |
| **3. Stream Sum** | $O(N)$ | $O(1)$ | Safe | Potential if $n > 65535$ | Highly readable, standard business logic layout. |
| **4. Stream XOR** | $O(N)$ | $O(1)$ | Safe | **No (Immune)** | Declarative representation; double-pass stream loop latency. |

---

### Detailed Evaluation Breakdown

#### 1. Gauss Arithmetic Summation
* **Strengths:** Blazing execution efficiency. Loops directly across primitive allocations with optimal memory reference mapping.
* **Arithmetic Safety Note:** For LeetCode constraints ($n \le 10^4$), an `int` variable is perfectly sufficient because the largest possible sum ($\approx 5 \times 10^7$) fits safely within a standard 32-bit signed integer limit ($2.14 \times 10^9$). If scaled up past $n = 65,535$, `expectedSum` will rollover, requiring an immediate conversion to long data architectures.

#### 2. Bitwise XOR Traversal
* **Strengths:** Maximum engineering robustness. Because bitwise operations modify logical value patterns directly without running numeric additions, it is completely immune to intermediate integer overflow values.
* **Readability Profiles:** Can challenge junior development profiles unfamiliar with low-level binary state tracking properties.

#### 3. Java 8 Functional Stream Summation
* **Strengths:** Highly concise declarative syntax. Abstracts loop control structures away entirely by combining standard mathematical operations with native stream primitives (`.sum()`).
* **Performance Impact:** Negligibly slower execution speeds compared to raw procedural blocks due to stream setup and structural pipeline overhead.

#### 4. Java 8 Stream XOR Reduction
* **Strengths:** Fully side-effect free. It tracks variables inside isolated closure frames without altering any shared states.
* **Weaknesses:** Suboptimal double-pass runtime properties. It scans the memory data grid twice—once to compress values (`Arrays.stream(nums)`) and a second time to fold indices (`IntStream.rangeClosed`), increasing memory traversal overhead.
