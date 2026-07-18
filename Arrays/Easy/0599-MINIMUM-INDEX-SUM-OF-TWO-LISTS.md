# LeetCode 599: Minimum Index Sum of Two Lists

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table** &nbsp;&nbsp; 🏷️ **String**

---

## 📝 Problem Description

Given two arrays of strings `list1` and `list2`, find the **common strings with the least index sum**.

A **common string** is a string that appears in both `list1` and `list2`.

A **common string with the least index sum** is a common string such that the sum of its index in `list1` and its index in `list2` is the **minimum** among all common strings.

Return *an array of all the **common strings with the least index sum***. Return the answer in **any order**.

### Examples

**Example 1:**
* **Input:** `list1 = ["Shogun","Tapioca Express","Burger King","KFC"], list2 = ["Piatti","The Grill Hunter","Tapioca Express","Shogun"]`
* **Output:** `["Shogun"]`
* **Explanation:** The common strings are:
  * `"Shogun"` with index sum $= 0 + 3 = 3$.
  * `"Tapioca Express"` with index sum $= 1 + 2 = 3$.
  Since both have the minimum index sum of 3, both can be returned. (LeetCode accepts any valid selection matching the minimum bound; if multiple matches exist, return all of them).

**Example 2:**
* **Input:** `list1 = ["Shogun","Tapioca Express","Burger King","KFC"], list2 = ["KFC","Shogun","Burger King"]`
* **Output:** `["Shogun"]`
* **Explanation:** The common string with the least index sum is `"Shogun"` with index sum $= 0 + 1 = 1$.

**Example 3:**
* **Input:** `list1 = ["happy","sad","good"], list2 = ["sad","happy","good"]`
* **Output:** `["sad","happy"]`
* **Explanation:** Both `"happy"` and `"sad"` have an index sum of $0 + 1 = 1$, which is the minimum.

### Constraints
* $1 \le \text{list1.length, list2.length} \le 1000$
* $1 \le \text{list1}[i].\text{length}, \text{list2}[i].\text{length} \le 30$
* `list1[i]` and `list2[i]` consist of spaces and English letters.
* All the strings of `list1` are **unique**.
* All the strings of `list2` are **unique**.

---

## 💡 Core Patterns Used

### 1. Hash Table Coordinate Cross-Referencing
* **Concept:** Pre-indexing string locations into a hash mapping layer to turn nested lookup scans ($O(M \times N)$) into sequential linear reviews ($O(M + N)$).
* **Mechanism:** Populates a map (`HashMap<String, Integer>`) with the strings of `list1` as keys and their corresponding array indices as values. Next, it iterates through `list2`. If a string from `list2` exists in the map, its index sum is calculated ($i_{\text{list1}} + j_{\text{list2}}$).

### 2. Dynamic Minimum List Reset Pattern
* **Concept:** Managing a running global minimum tracker alongside a clearing results buffer list.
* **Mechanism:** As index sums are evaluated:
  1. If a new calculation is strictly smaller than our current global minimum, the previous results are outdated. The minimum tracker updates, the results list is completely cleared (`list.clear()`), and the new string is added.
  2. If the calculation perfectly matches the current minimum, the string is appended to the results list.
  3. If it is larger, it is discarded.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Index Map Tracking (Standard DSA Style)
The textbook optimal solution. It balances clean separation of logic with linear execution speed by leveraging hash tables and dynamic list clearing bounds.

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MinimumIndexSumClassic {
    public static String[] findRestaurant(String[] list1, String[] list2) {
        // Space Optimization: Map the smaller array to minimize map allocations
        if (list1.length > list2.length) {
            return findRestaurant(list2, list1);
        }

        Map<String, Integer> indexMap = new HashMap<>(list1.length);
        for (int i = 0; i < list1.length; i++) {
            indexMap.put(list1[i], i);
        }

        List<String> resultList = new ArrayList<>();
        int minIndexSum = Integer.MAX_VALUE;

        for (int j = 0; j < list2.length; j++) {
            String currentStr = list2[j];
            
            if (indexMap.containsKey(currentStr)) {
                int i = indexMap.get(currentStr);
                int currentIndexSum = i + j;

                if (currentIndexSum < minIndexSum) {
                    minIndexSum = currentIndexSum;
                    resultList.clear(); // Clear out dated strings with higher index sums
                    resultList.add(currentStr);
                } else if (currentIndexSum == minIndexSum) {
                    resultList.add(currentStr);
                }
            }
        }

        return resultList.toArray(new String[0]);
    }
}
```

### 2. Java 8 Functional Stream Map-Reduce Pipeline
A clean functional version that uses stream operators to extract matches, grouping index positions via map collector frames.

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class MinimumIndexSumStream {
    public static String[] findRestaurant(String[] list1, String[] list2) {
        // Build index lookup registry map via standard loop to preserve performance
        Map<String, Integer> map1 = IntStream.range(0, list1.length)
                .boxed()
                .collect(Collectors.toMap(i -> list1[i], Function.identity()));

        // Stream elements of list2 that exist in list1, mapping them to their index sum pairs
        Map<Integer, List<String>> groupedBySum = IntStream.range(0, list2.length)
                .filter(j -> map1.containsKey(list2[j]))
                .boxed()
                .collect(Collectors.groupingBy(
                        j -> map1.get(list2[j]) + j,
                        Collectors.mapping(j -> list2[j], Collectors.toList())
                ));

        // Find the absolute minimum key entry within our index sum grouping table
        return groupedBySum.keySet().stream()
                .min(Integer::compare)
                .map(groupedBySum::get)
                .orElse(Arrays.asList())
                .toArray(new String[0]);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Heap Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Index Map Tracking** | $O(M + N)$ | $O(\min(M, N))$ | Medium | Good | Well-balanced performance; utilizes clean array clearing mechanisms. |
| **2. Stream Pipeline** | $O(M + N)$ | $O(M + N)$ | High | Fair | Outstanding code transparency; introduces object boxing and group allocation overhead. |

*Where **$M$** is the length of `list1` and **$N$** is the length of `list2` ($1 \le M, N \le 1000$).*

---

### Detailed Evaluation Breakdown

#### 1. Optimization via Initial Parameter Flipping
In Implementation 1, adding the quick condition check `if (list1.length > list2.length)` acts as an architectural memory saver. By forcing the internal `HashMap` to always construct its coordinate mapping registry over the *smaller* of the two arrays, auxiliary storage overhead remains strictly capped at $O(\min(M, N))$. This effectively minimizes hash bucket memory usage.

#### 2. Dynamic List Clearing vs. Multi-Pass Collections
* Traditional naive solutions loop through the array once to calculate all index sums, store them in a temporary structure, sort the sums, and then extract the smallest ones. This requires a time complexity of $O(K \log K)$ for sorting.
* By using the **Dynamic Minimum List Reset Pattern**, we update our state variables on-the-fly inside a single loop. Calling `resultList.clear()` ensures that we only retain the optimal matches, keeping auxiliary space clean and minimal.

#### 3. Java 8 Stream Pipeline Nuances
* **Strengths:** High declarative composition. It cleanly partitions element matches into isolated mathematical index-bucket groups without mutating exterior global pointer flags manually.
* **Performance Impact:** Heavy allocation overhead. Using `.boxed()` alongside `Collectors.groupingBy()` forces primitive indexes to continuously wrap into object types (`Integer`) while generating sub-lists inside the heap. For the given small constraint limits ($1000$ elements), this runs securely, but it introduces extra garbage collection overhead compared to raw register loops.
