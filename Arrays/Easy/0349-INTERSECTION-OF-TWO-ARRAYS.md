# LeetCode 349: Intersection of Two Arrays

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Hash Table &nbsp;&nbsp; 🏷️ Two Pointers &nbsp;&nbsp; 🏷️ Sorting

---

## 📝 Problem Description

Given two integer arrays `nums1` and `nums2`, return *an array of their intersection*. Each element in the result must be **unique** and you may return the result in **any order**.

### Examples

**Example 1:**
* **Input:** `nums1 =, nums2 =`
* **Output:** ``

**Example 2:**
* **Input:** `nums1 =, nums2 =`
* **Output:** `` or ``

### Constraints
* $1 \le \text{nums1.length, nums2.length} \le 1000$
* $0 \le \text{nums1}[i], \text{nums2}[i] \le 1000$

---

## 💡 Core Patterns Used

### 1. Hash Set Intersection Filtering Pattern
* **Concept:** Utilizing the constant-time membership lookups of hash structures to cross-reference distinct items across separate datasets.
* **Mechanism:** Collects all elements from `nums1` into a unique `HashSet`. Then, it loops through `nums2` and tests each item against the set using `.contains()`. Matching elements are transferred into a second set to guarantee result uniqueness before being copied back into a primitive array format.

### 2. Dual-Pointer Sorting Convergence Pattern
* **Concept:** Sorting both datasets to align values, then processing items with pointers moving at different speeds to identify matches without requiring heap allocations.
* **Mechanism:** Orders both input arrays. Pointers `i` (for `nums1`) and `j` (for `nums2`) move forward conditionally: the pointer resting on the smaller value advances. If the values match, the number is recorded (avoiding duplication by checking the last inserted element) and both pointers step forward together.

### 3. Java 8 Stream Pipeline Filtering
* **Concept:** Constructing an abstract pipeline that filters matching stream collections down into unique primitive arrays.
* **Mechanism:** Streams the unique items from the first array (`.distinct()`) and uses a `.filter()` condition to keep only the values that match elements from the second array.

---

## 💻 Java & Java 8 Implementations

### 1. Two-Set Lookups (Optimal $O(M + N)$ Time)
The standard industry architecture layout. It balances clean separation of logic with linear execution speed by leveraging hash tables under the hood.

```java
import java.util.HashSet;
import java.util.Set;

public class IntersectionTwoSets {
    public static int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set1 = new HashSet<>();
        for (int num : nums1) {
            set1.add(num);
        }

        Set<Integer> intersectionSet = new HashSet<>();
        for (int num : nums2) {
            if (set1.contains(num)) {
                intersectionSet.add(num);
            }
        }

        // Convert the tracking collection back to a primitive array format
        int[] result = new int[intersectionSet.size()];
        int index = 0;
        for (int num : intersectionSet) {
            result[index++] = num;
        }
        return result;
    }
}
```

### 2. In-Place Dual-Pointer Sorting (Optimal $O(1)$ Auxiliary Space)
An excellent solution for memory-constrained environments. It avoids heap allocations entirely by sorting the input data directly.

```java
import java.util.Arrays;

public class IntersectionTwoPointers {
    public static int[] intersection(int[] nums1, int[] nums2) {
        Arrays.sort(nums1);
        Arrays.sort(nums2);

        int i = 0, j = 0, index = 0;
        // Allocate a temporary buffer array bounded by the smallest input limit
        int[] tempBuffer = new int[Math.min(nums1.length, nums2.length)];

        while (i < nums1.length && j < nums2.length) {
            if (nums1[i] < nums2[j]) {
                i++;
            } else if (nums1[i] > nums2[j]) {
                j++;
            } else {
                // To maintain uniqueness, ensure the value doesn't match the last inserted element
                if (index == 0 || tempBuffer[index - 1] != nums1[i]) {
                    tempBuffer[index++] = nums1[i];
                }
                i++;
                j++;
            }
        }

        // Trim the buffer array down to its actual size before returning
        return Arrays.copyOf(tempBuffer, index);
    }
}
```

### 3. Java 8 Stream Pipeline Set Validation
A declarative solution that uses standard functional streams to check for matching unique elements.

```java
import java.util.Arrays;
import java.util.Set;
import java.util.stream.Collectors;

public class IntersectionStreamFilter {
    public static int[] intersection(int[] nums1, int[] nums2) {
        // Collect nums2 elements into a Set to keep lookups inside filter loops fast
        Set<Integer> set2 = Arrays.stream(nums2).boxed().collect(Collectors.toSet());

        return Arrays.stream(nums1)
                     .distinct() // Process each unique element from nums1 exactly once
                     .filter(set2::contains) // Keep elements that exist in the second dataset
                     .toArray(); // Collapse down to a primitive int[] array format
    }
}
```

### 4. Direct Loop Primitive Cache Optimization (Maximum Performance)
This approach takes advantage of the specific problem constraint ($0 \le \text{nums}[i] \le 1000$) by using a simple boolean tracking lookup table, outperforming standard hash-based methods.

```java
public class IntersectionPrimitiveCache {
    public static int[] intersection(int[] nums1, int[] nums2) {
        // Fixed-size primitive array matches constraint boundaries perfectly
        boolean[] seen = new boolean[1001];
        for (int num : nums1) {
            seen[num] = true;
        }

        int[] temp = new int[1001];
        int count = 0;
        for (int num : nums2) {
            if (seen[num]) {
                temp[count++] = num;
                seen[num] = false; // Prevents recording duplicate values
            }
        }

        int[] result = new int[count];
        System.arraycopy(temp, 0, result, 0, count);
        return result;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Two-Set Lookups** | $O(M + N)$ | $O(M + N)$ | Safe if unshared | Medium | Well-balanced performance; handles arbitrary integer bounds smoothly. |
| **2. Dual-Pointer Sort**| $O(M \log M + N \log N)$ | $O(1)$ (In-Place) | Safe if unshared | **Zero (Immune)** | Eliminates garbage collection allocation pressure entirely; requires mutating input data. |
| **3. Stream Filter** | $O(M + N)$ | $O(N)$ | Safe | High | Exceptional declarative visibility; suffers from boxed object allocation overhead. |
| **4. Primitive Cache** | $O(M + N)$ | $O(1)$ (Bounded constant) | Safe if unshared | **Zero (Immune)** | Fastest option by far, but relies strictly on narrow, bounded element limits. |

---

### Detailed Evaluation Breakdown

#### 1. Two-Set Lookups
* **Strengths:** Predictable linear runtime performance. It scales gracefully regardless of whether the incoming numbers are tightly clustered or sparse.
* **Limitations:** Spawning multiple `HashSet` container instances requires object allocations on the application heap, which adds a minor garbage collection tax.

#### 2. In-Place Dual-Pointer Sorting
* **Strengths:** Excellent memory footprint management. It processes values linearly using standard registers, matching the requirements of memory-restricted hardware or real-time systems.
* **Weaknesses:** Sort cost latency. If arrays are already unique but out of order, forcing full QuickSort cycles before checking values adds an $O(\log N)$ performance penalty.

#### 3. Java 8 Stream Pipeline Set Validation
* **Strengths:** Highly concise declarative composition. It clearly separates distinct filtering logic from the underlying storage code.
* **Performance Impact:** Autoboxing overhead. Converting elements repeatedly between primitive `int` and boxed `Integer` wrappers creates object churn inside the application memory pool.

#### 4. Direct Loop Primitive Cache Optimization
* **Strengths:** The performance champion. Because the constraints guarantee that elements never exceed 1000, replacing complex hash maps with a simple `boolean[]` lookup array converts tracking operations into single-cycle direct memory hits.
* **Weaknesses:** Highly brittle architecture. If the database updates to include values outside the 0–1000 range, this method will throw an immediate `ArrayIndexOutOfBoundsException`.
