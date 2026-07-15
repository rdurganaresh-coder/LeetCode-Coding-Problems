# LeetCode 350: Intersection of Two Arrays II

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Hash Table &nbsp;&nbsp; 🏷️ Two Pointers &nbsp;&nbsp; 🏷️ Binary Search &nbsp;&nbsp; 🏷️ Sorting

---

## 📝 Problem Description

Given two integer arrays `nums1` and `nums2`, return *an array of their intersection*. Each element in the result must appear **as many times as it shows in both arrays** and you may return the result in **any order**.

### Examples

**Example 1:**
* **Input:** `nums1 =, nums2 = [2,2]`
* **Output:** `[2,2]`

**Example 2:**
* **Input:** `nums1 =, nums2 = [9,4,9,8,4]`
* **Output:** `[4,9]` or `[9,4]`

### Constraints
* $1 \le \text{nums1.length, nums2.length} \le 1000$
* $0 \le \text{nums1}[i], \text{nums2}[i] \le 1000$

### Follow-up Questions
1. What if the given array is already sorted? How would you optimize your algorithm?
2. What if `nums1`'s size is small compared to `nums2`'s size? Which algorithm is better?
3. What if elements of `nums2` are stored on disk, and the memory is limited such that you cannot load all elements into the memory at once?

---

## 💡 Core Patterns Used

### 1. Hash Table Frequency Matching Pattern
* **Concept:** Recording element occurrence frequencies inside a dictionary map to handle multiset intersection constraints.
* **Mechanism:** Populates a frequency map (`HashMap<Integer, Integer>`) using the elements of `nums1`. Next, it scans through `nums2`. If an element is found in the map with a frequency count greater than zero, it is added to the result buffer, and its stored frequency count in the map is decremented by one.

### 2. Dual-Pointer Sorting Convergence Pattern
* **Concept:** Aligning elements via sorting, then stepping dual pointers forward based on numerical comparisons to capture overlapping duplicates without allocating tracking structures.
* **Mechanism:** Sorts both input arrays. If `nums1[i] == nums2[j]`, the matching item is added directly to the result array, and both pointers advance. If the elements don't match, only the pointer resting on the smaller value is advanced.

### 3. Java 8 Stateful Map Stream Pipeline
* **Concept:** Constructing an abstract pipeline that filters matching stream collections down using functional dictionary transitions.
* **Mechanism:** Uses an intermediate frequency mapping table inside a sequential `.filter()` structure to check and decrement balance boundaries dynamically on the fly.

---

## 💻 Java & Java 8 Implementations

### 1. Frequency Map Tracking (Optimal $O(M + N)$ Time)
The ideal industry standard layout for handling un-sorted, general data inputs. It processes elements linearly by leveraging internal hash tables.

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

public class IntersectionIIFrequencyMap {
    public static int[] intersect(int[] nums1, int[] nums2) {
        // Optimize space: Ensure the map is always populated with the smaller array
        if (nums1.length > nums2.length) {
            return intersect(nums2, nums1);
        }

        Map<Integer, Integer> frequencyMap = new HashMap<>();
        for (int num : nums1) {
            frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
        }

        int[] tempBuffer = new int[nums1.length];
        int writeIndex = 0;

        for (int num : nums2) {
            int count = frequencyMap.getOrDefault(num, 0);
            if (count > 0) {
                tempBuffer[writeIndex++] = num;
                frequencyMap.put(num, count - 1); // Decrement occurrence balance
            }
        }

        return Arrays.copyOf(tempBuffer, writeIndex);
    }
}
```

### 2. In-Place Dual-Pointer Sorting (Optimal for Pre-Sorted Arrays)
The absolute best choice when input data arrays are already ordered. It satisfies the first follow-up question by skipping map overhead completely to achieve $O(1)$ auxiliary space complexity.

```java
import java.util.Arrays;

public class IntersectionIITwoPointers {
    public static int[] intersect(int[] nums1, int[] nums2) {
        // In-place sorting step (Skip this step if inputs are already sorted)
        Arrays.sort(nums1);
        Arrays.sort(nums2);

        int i = 0, j = 0, writeIndex = 0;
        int[] tempBuffer = new int[Math.min(nums1.length, nums2.length)];

        while (i < nums1.length && j < nums2.length) {
            if (nums1[i] < nums2[j]) {
                i++; // Advance the smaller element track
            } else if (nums1[i] > nums2[j]) {
                j++; // Advance the smaller element track
            } else {
                // Record the match and advance both pointers simultaneously
                tempBuffer[writeIndex++] = nums1[i];
                i++;
                j++;
            }
        }

        return Arrays.copyOf(tempBuffer, writeIndex);
    }
}
```

### 3. Java 8 Functional Stream Stateful Filter
A declarative variant that incorporates an active dictionary inside an abstract streaming mapping sequence.

```java
import java.util.Arrays;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class IntersectionIIStream {
    public static int[] intersect(int[] nums1, int[] nums2) {
        // Aggregate elements of the first array into a frequency map
        Map<Integer, Long> frequencyMap = Arrays.stream(nums1)
            .boxed()
            .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

        // Filter elements of the second array using the map state
        return Arrays.stream(nums2)
            .filter(num -> {
                if (frequencyMap.containsKey(num) && frequencyMap.get(num) > 0) {
                    frequencyMap.put(num, frequencyMap.get(num) - 1);
                    return true;
                }
                return false;
            })
            .toArray();
    }
}
```

### 4. Direct Loop Primitive Cache Optimization (Maximum Performance)
This architecture takes advantage of the specific problem constraint ($0 \le \text{nums}[i] \le 1000$). By swapping out complex `HashMap` structures for a simple primitive array look-up cache, it eliminates memory overhead completely.

```java
import java.util.Arrays;

public class IntersectionIIPrimitiveCache {
    public static int[] intersect(int[] nums1, int[] nums2) {
        // Fixed-size primitive array matches the element value constraints perfectly
        int[] countCache = new int;
        for (int num : nums1) {
            countCache[num]++;
        }

        int[] tempBuffer = new int[Math.min(nums1.length, nums2.length)];
        int writeIndex = 0;

        for (int num : nums2) {
            if (countCache[num] > 0) {
                tempBuffer[writeIndex++] = num;
                countCache[num]--; // Safely track frequency balances down to zero
            }
        }

        return Arrays.copyOf(tempBuffer, writeIndex);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Frequency Map** | $O(M + N)$ | $O(\min(M, N))$ | Safe if unshared | Medium | Well-balanced performance; handles general unsorted integer bounds seamlessly. |
| **2. Dual-Pointer Sort**| $O(M \log M + N \log N)$ | $O(1)$ (In-Place) | Safe if unshared | **Zero (Immune)** | Eliminates garbage collection allocation pressure entirely; requires sorting logic unless pre-arranged. |
| **3. Stream Filter** | $O(M + N)$ | $O(M)$ | Safe | High | Exceptional declarative visibility; suffers from boxed object allocation overhead. |
| **4. Primitive Cache** | $O(M + N)$ | $O(1)$ (Bounded constant) | Safe if unshared | **Zero (Immune)** | Fastest option by far, but relies strictly on narrow, bounded element limits. |

---

## 🎯 Follow-up Architectural Answers

#### 1. What if the given array is already sorted?
Use the **Dual-Pointer Sorting Convergence Pattern** (Implementation 2), but omit the explicit `Arrays.sort` initialization calls. This achieves an optimal time complexity of $O(M + N)$ while using $O(1)$ constant auxiliary space. It updates pointers via simple registers with no object allocation overhead.

#### 2. What if `nums1`'s size is small compared to `nums2`'s size?
The **Frequency Map Tracking approach** (Implementation 1) is optimal. By forcing the map initialization to execute exclusively over the smaller collection via the initial argument-swap block (`if (nums1.length > nums2.length)`), auxiliary space complexity remains strictly bounded to $O(\min(M, N))$, saving memory.

#### 3. What if elements of `nums2` are stored on disk and memory is limited?
If `nums2` cannot fit into memory at once, use the **Frequency Map Tracking approach** (Implementation 1). Load the smaller array (`nums1`) completely into an in-memory frequency map. Then, stream chunks of `nums2` sequentially from disk into memory buffer batches. Process each chunk against the active map to write matching intersection elements to an output stream on the fly. This keeps the memory footprint bound to $O(M)$ instead of loading the massive $N$-sized dataset all at once.

***

If you would like to tackle another high-frequency array structure problem, such as **LeetCode 448: Find All Numbers Disappeared in an Array** (using index sign inversion) or want a custom **JUnit benchmark test script**, let me know how you would like to proceed!
