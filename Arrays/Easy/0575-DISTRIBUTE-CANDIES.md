# LeetCode 575: Distribute Candies

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table**

---

## 📝 Problem Description

Alice has `n` candies, where the $i^{th}$ candy is of type `candyType[i]`. Alice noticed that she started to gain weight, so she visited a doctor.

The doctor advised Alice to only eat `n / 2` of the candies she has (`n` is always even). Alice likes her candies very much, and she wants to eat the maximum number of **different** types of candies while still following the doctor's advice.

Given the integer array `candyType` of length `n`, return *the **maximum** number of different types of candies she can eat if she only eats `n / 2` of them*.

### Examples

**Example 1:**
* **Input:** `candyType = [1,1,2,2,3,3]`
* **Output:** `3`
* **Explanation:** Alice can only eat $6 / 2 = 3$ candies. There are 3 unique types (1, 2, and 3). She can take exactly 1 candy of each type to maximize variety: `[1, 2, 3]`.

**Example 2:**
* **Input:** `candyType = [1,1,2,3]`
* **Output:** `2`
* **Explanation:** Alice can only eat $4 / 2 = 2$ candies. There are 3 unique types (1, 2, and 3). She can choose two different types (e.g., `[2, 3]` or `[1, 2]`), obtaining a maximum variety of 2.

**Example 3:**
* **Input:** `candyType = [6,6,6,6]`
* **Output:** `1`
* **Explanation:** Alice can only eat $4 / 2 = 2$ candies. Even though she can eat 2 candies, there is only 1 type of candy present (6). Thus, she can only eat 1 unique type.

### Constraints
* `n == candyType.length`
* $2 \le n \le 10^4$
* `n` is an **even** number.
* $-10^5 \le \text{candyType}[i] \le 10^5$

---

## 💡 Core Patterns Used

### 1. Set Cardinality Bounding Pattern
* **Concept:** Comparing unique dataset variety counts against a structural upper allocation limit.
* **Mechanism:** The maximum number of unique candies Alice can eat is bound by two factors:
  1. The absolute count of different unique candy types available in the array.
  2. The strict doctor-imposed eating limit ($\frac{n}{2}$).
  The final solution is mathematically guaranteed to be the minimum of these two values:
  $$\text{Result} = \min(\text{Unique Types}, \frac{n}{2})$$

### 2. Direct-Address Lookup Buffer (Primitive Map Optimization)
* **Concept:** Swapping an object-heavy hash table for a fast, fixed-size primitive array cache when element value domains are tightly bounded.
* **Mechanism:** Since constraints guarantee $-10^5 \le \text{candyType}[i] \le 10^5$, the entire unique type spectrum spans a range of $200,001$ slots. Offsetting elements by adding $10^5$ maps negative values into valid array indices. This allows us to track unique items in a plain `boolean[]` array using single-cycle register hits.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Direct Array Cache (Standard DSA Style)
The definitive execution benchmark champion. It bypasses object allocation and hashing collison calculations entirely by tracking candy types inside plain primitive bits.

```java
public class DistributeCandiesCache {
    public static int distributeCandies(int[] candyType) {
        int maxAllowedEat = candyType.length / 2;
        
        // Constraints: -100000 <= candyType[i] <= 100000 -> Total range size is 200001
        int offset = 100000;
        boolean[] seenTypes = new boolean[200001];
        int uniqueCount = 0;

        for (int type : candyType) {
            int mappedIndex = type + offset;
            if (!seenTypes[mappedIndex]) {
                seenTypes[mappedIndex] = true;
                uniqueCount++;
                
                // Short-circuit: Variety can never exceed the doctor's eating limit
                if (uniqueCount == maxAllowedEat) {
                    return maxAllowedEat;
                }
            }
        }

        return Math.min(uniqueCount, maxAllowedEat);
    }
}
```

### 2. Standard HashSet Collection (Object-Oriented Approach)
The standard, flexible industry approach. It utilizes a `HashSet` to evaluate data variety safely without depending on element numeric value limits.

```java
import java.util.HashSet;
import java.util.Set;

public class DistributeCandiesHashSet {
    public static int distributeCandies(int[] candyType) {
        int maxAllowedEat = candyType.length / 2;
        Set<Integer> uniqueCandies = new HashSet<>();

        for (int type : candyType) {
            uniqueCandies.add(type);
            // Early exit: Stop tracking immediately if unique types saturate the doctor's limit
            if (uniqueCandies.size() == maxAllowedEat) {
                return maxAllowedEat;
            }
        }

        return Math.min(uniqueCandies.size(), maxAllowedEat);
    }
}
```

### 3. Java 8 Declarative Stream Reduction Pipeline
A clean functional version that calculates uniqueness using a single distinct matching pipeline statement.

```java
import java.util.Arrays;

public class DistributeCandiesStream {
    public static int distributeCandies(int[] candyType) {
        // Stream values, deduplicate them via distinct(), and count the total variety volume
        long uniqueCount = Arrays.stream(candyType)
                                 .distinct()
                                 .count();

        // Convert the long output safely and apply the minimum limit restriction rule
        return (int) Math.min(uniqueCount, candyType.length / 2);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Allocations | Short-Circuits Early? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Direct Cache** | $O(N)$ | $O(\text{Range})$ | **Zero (Immune)** | **Yes (Instant)** | Lightning fast; relies strictly on static parameter range boundaries. |
| **2. HashSet** | $O(N)$ | $O(\min(N, \text{Range}))$ | Medium | **Yes (Instant)** | Extremely robust; independent of structural element size limits. |
| **3. Stream Pipeline** | $O(N)$ | $O(N)$ | Low | No (Full stream pass) | High declarative visibility; forces complete array pipeline scans. |

---

### Detailed Evaluation Breakdown

#### 1. Direct Array Cache Strategy vs. Standard HashSet
* **Peak Performance Execution:** Implementation 1 runs exceptionally fast because it bypasses the inner workings of object map storage (e.g., auto-boxing primitives from `int` to `Integer`, hash code calculation, tree map node splitting).
* **Early-Exit Short-Circuiting:** If a candy log array contains $10^5$ items but the first 5000 items are already unique, both the array cache and `HashSet` strategies break out of execution instantly. This saves vast amounts of processing cycles compared to scanning the remainder of the dataset.

#### 2. Java 8 Stream Pipeline Realities
* **Strengths:** High expressive composition. It communicates intended structural requirements directly to the reader with no internal loops or flag manipulations.
* **Limitations:** Lacks intermediate termination limits. Because the pipeline relies on `.distinct().count()`, the underlying stream compiler is forced to read *every individual memory coordinate in the array* to lock in the final unique scalar metric, sacrificing short-circuit speed optimizations.
