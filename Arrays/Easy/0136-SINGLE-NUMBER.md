# LeetCode 136: Single Number

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Bit Manipulation**

---

## 📝 Problem Description

Given a **non-empty** array of integers `nums`, every element appears *twice* except for one. Find that single one.

You must implement a solution with a **linear runtime complexity** and use **only constant extra space**.

### Examples

**Example 1:**
* **Input:** `nums = [2, 2, 1]`
* **Output:** `1`

**Example 2:**
* **Input:** `nums = [4, 1, 2, 1, 2]`
* **Output:** `4`

**Example 3:**
* **Input:** `nums = [1]`
* **Output:** `1`

### Constraints
* $1 \le \text{nums.length} \le 3 \times 10^4$
* $-3 \times 10^4 \le \text{nums}[i] \le 3 \times 10^4$
* Each element in the array appears twice except for one element which appears only once.

---

## 💡 Core Patterns Used

### 1. XOR Bit Manipulation Pattern
* **Concept:** Leveraging bitwise XOR properties to cancel out duplicate elements. 
* **Mechanism:** XOR satisfies Commutativity ($A \oplus B = B \oplus A$) and Associativity ($A \oplus (B \oplus C) = (A \oplus B) \oplus C$). Crucially, any number XORed with itself is zero ($A \oplus A = 0$), and any number XORed with zero remains itself ($A \oplus 0 = A$). Running a global XOR operation leaves only the unique non-duplicate variable standing.

### 2. Math/Set Difference Pattern
* **Concept:** Isolating structural element counts using unique sets.
* **Mechanism:** Calculates $2 \times (\text{Sum of Unique Elements}) - (\text{Sum of All Elements})$. The difference perfectly reveals the missing duplicate balance.

### 3. Stream Bitwise Reduction
* **Concept:** Compacting a primitive stream sequence down to a single computed state.
* **Mechanism:** Evaluates the continuous data collection by feeding seed inputs directly into sequential bitwise lambda accumulators.

---

## 💻 Java & Java 8 Implementations

### 1. Classic In-Place Bitwise XOR (Optimal $O(N)$ Time & $O(1)$ Space)
This traditional approach sets a tracking variable to 0 and sequentially XORs every item in the array. Duplicates inherently neutralise each other.

```java
public class SingleNumberXOR {
    public static int singleNumber(int[] nums) {
        int result = 0;
        for (int num : nums) {
            result ^= num; // XOR identity operation eliminates duplicate bit profiles
        }
        return result;
    }
}
```

### 2. Java 8 Streams `reduce` Pipeline
This functional variant condenses the XOR traversal into a clean, single-line factory pattern mapping pipeline using primitive array processing.

```java
import java.util.Arrays;

public class SingleNumberStreamReduce {
    public static int singleNumber(int[] nums) {
        // Leverages standard identity reduction accumulator to collapse arrays
        return Arrays.stream(nums)
                     .reduce(0, (accumulator, current) -> accumulator ^ current);
    }
}
```

### 3. Math & Set Evaluation (Alternative Structural Approach)
This approach collects all unique values inside a hash set to compute the unique sum vs the standard duplicate total array sum balance.

```java
import java.util.HashSet;
import java.util.Set;

public class SingleNumberMathSet {
    public static int singleNumber(int[] nums) {
        Set<Integer> uniqueSet = new HashSet<>();
        long totalSum = 0;
        long uniqueSum = 0;

        for (int num : nums) {
            if (uniqueSet.add(num)) {
                uniqueSum += num; // Tracks the sum of each unique entry once
            }
            totalSum += num; // Tracks the complete sequential literal sum
        }

        return (int) (2 * uniqueSum - totalSum);
    }
}
```

### 4. Java 8 Stream Math Pipeline
This approach mirrors the Set-Math concept using pure functional collectors, map layers, and sum reductions.

```java
import java.util.Arrays;
import java.util.stream.Collectors;

public class SingleNumberStreamMath {
    public static int singleNumber(int[] nums) {
        // Collect into distinct set to aggregate unique sums
        int uniqueSum = Arrays.stream(nums)
                              .distinct()
                              .sum();

        int totalSum = Arrays.stream(nums)
                             .sum();

        return (2 * uniqueSum) - totalSum;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | LeetCode Constraint Safe? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic XOR** | $O(N)$ | $O(1)$ | Safe if unshared | Yes (Exceeds baseline limits) | Fast primitive looping with zero object instantiation |
| **2. Stream Reduce** | $O(N)$ | $O(1)$ | Safe (Immutable) | Yes (Exceeds baseline limits) | Modern declarative style with minimal setup cost |
| **3. Math Set** | $O(N)$ | $O(N)$ | Safe if unshared | Yes (Fails $O(1)$ constraint) | Easy to comprehend but heavy heap overhead |
| **4. Stream Math** | $O(N)$ | $O(N)$ | Safe | Yes (Fails $O(1)$ constraint) | High boxing and double-iteration scan latency |

---

### Detailed Evaluation Breakdown

#### 1. Classic In-Place Bitwise XOR
* **Strengths:** Utterly optimal execution. Runs inside hardware registers using primitive loops without triggering reference trace allocations, matching the strict constant extra-space requirement.
* **Weaknesses:** Requires basic understanding of binary arithmetic logic states, which can challenge junior readability profiles.

#### 2. Java 8 Streams `reduce` Pipeline
* **Strengths:** Fully side-effect free. It wraps primitive logic loops without introducing tracking variables or index references.
* **Parallel Execution Benefit:** This stream variant can safely use a `.parallel()` suffix if dealing with massive arrays. Because the XOR operation is associative and commutative, chunked regional evaluations merge flawlessly without synchronization race hazards.

#### 3. Math & Set Evaluation
* **Strengths:** Clean mathematical elegance. Translates directly across data languages without relying on low-level bit operations.
* **Constraint Violations:** Fails LeetCode's explicit constant auxiliary space constraint ($O(1)$). Storing $N/2$ unique elements inside a `HashSet` increases space tracking limits to $O(N)$, risking memory issues on larger inputs.

#### 4. Java 8 Stream Math Pipeline
* **Strengths:** Elegant look and feel, cleanly isolating array operations into discrete functional blocks.
* **Performance Impact:** Slowest execution method. Internally, `.distinct()` dynamically allocates state trackers under the hood to filtering objects, and running dual scans (`.sum()` calls) duplicates data pipeline traversal time.
