# LeetCode 496: Next Greater Element I

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table** &nbsp;&nbsp; 🏷️ **Stack** &nbsp;&nbsp; 🏷️ **Monotonic Stack**

---

## 📝 Problem Description

The **next greater element** of some element `x` in an array is the first greater element that is to the right of `x` in the same array.

You are given two distinct 0-indexed integer arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`.

For each `0 <= i < nums1.length`, find the index `j` such that `nums1[i] == nums2[j]` and determine the next greater element of `nums2[j]` in `nums2`. If there is no next greater element, then the answer for this query is `-1`.

Return *an array `ans` of length `nums1.length` such that `ans[i]` is the next greater element as described above*.

### Examples

**Example 1:**
* **Input:** `nums1 =, nums2 = [1,3,4,2]`
* **Output:** `[-1,3,-1]`
* **Explanation:** The next greater element for each value of nums1 is as follows:
  * 4: `nums1[0] = 4` is at `nums2[2] = 4`. There is no next greater element to its right, so output is `-1`.
  * 1: `nums1[1] = 1` is at `nums2[0] = 1`. The next greater element to its right is 3, so output is `3`.
  * 2: `nums1[2] = 2` is at `nums2[3] = 2`. There is no next greater element to its right, so output is `-1`.

**Example 2:**
* **Input:** `nums1 =, nums2 = [1,2,3,4]`
* **Output:** `[3,-1]`
* **Explanation:** The next greater element for each value of nums1 is as follows:
  * 2: `nums1[0] = 2` is at `nums2[1] = 2`. The next greater element to its right is 3, so output is `3`.
  * 4: `nums1[1] = 4` is at `nums2[3] = 4`. There is no next greater element to its right, so output is `-1`.

### Constraints
* $1 \le \text{nums1.length} \le \text{nums2.length} \le 1000$
* $0 \le \text{nums1}[i], \text{nums2}[i] \le 10^4$
* All integers in `nums1` and `nums2` are unique.
* All the integers of `nums1` also appear in `nums2`.

---

## 💡 Core Patterns Used

### 1. Monotonic Decreasing Stack Pattern
* **Concept:** Maintaining elements in a strictly decreasing structural order within a stack to locate the first larger neighbor in a single linear pass.
* **Mechanism:** Iterates forward through `nums2`. While the stack is not empty and the current element `num` is greater than the element at the top of the stack (`stack.peek()`), the current element is confirmed as the *next greater element* for that top item. The top item is popped and recorded into a lookup map. The current element is then pushed onto the stack. Any elements remaining in the stack at the end have no greater element to their right.

### 2. Bounded Array Index Cache Map
* **Concept:** Utilizing a plain primitive array instead of a complex object hash table when the input values are tightly bounded.
* **Mechanism:** Since the constraints guarantee $0 \le \text{nums2}[i] \le 10^4$, a fixed-size `int[]` array can act as a direct-addressing map. This replaces `HashMap<Integer, Integer>` with single-cycle direct memory hits.

---

## 💻 Java & Java 8 Implementations

### 1. Monotonic Stack with HashMap (Standard Production DSA)
The textbook optimal solution. It solves the next greater element problem for all items in `nums2` in linear time, then queries the map to build the final answer array.

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Stack;

public class NextGreaterStackMap {
    public static int[] nextGreaterElement(int[] nums1, int[] nums2) {
        Map<Integer, Integer> nextGreaterMap = new HashMap<>();
        Stack<Integer> stack = new Stack<>();

        // Process nums2 to map each element to its next greater neighbor
        for (int num : nums2) {
            while (!stack.isEmpty() && stack.peek() < num) {
                nextGreaterMap.put(stack.pop(), num);
            }
            stack.push(num);
        }

        // Map values into the final output array
        int[] result = new int[nums1.length];
        for (int i = 0; i < nums1.length; i++) {
            result[i] = nextGreaterMap.getOrDefault(nums1[i], -1);
        }

        return result;
    }
}
```

### 2. Array Cache Pointer Optimization (Peak Performance Strategy)
This architecture optimizes the monotonic stack pattern by substituting standard collection objects with high-speed primitive arrays, eliminating heap object allocations entirely.

```java
public class NextGreaterPrimitiveCache {
    public static int[] nextGreaterElement(int[] nums1, int[] nums2) {
        // Bounded constraint map: 0 <= nums2[i] <= 10000
        int[] mapCache = new int[10001];
        int[] stack = new int[nums2.length];
        int top = -1;

        // Populate the lookup table cache using a high-speed primitive array stack
        for (int num : nums2) {
            while (top >= 0 && stack[top] < num) {
                mapCache[stack[top--]] = num;
            }
            stack[++top] = num;
        }

        // Elements left in the stack have no greater element; set their cache entries to -1
        while (top >= 0) {
            mapCache[stack[top--]] = -1;
        }

        // Direct-addressing mapping minimizes cycle overhead
        int[] result = new int[nums1.length];
        for (int i = 0; i < nums1.length; i++) {
            int val = mapCache[nums1[i]];
            result[i] = (val == 0) ? -1 : val;
        }

        return result;
    }
}
```

### 3. Java 8 Declarative Stream Pipeline ($O(N)$ Time)
This variant leverages functional mappings to build the answer array from the monotonic stack results using clean, concise stream syntax.

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.Stack;

public class NextGreaterStreamMap {
    public static int[] nextGreaterElement(int[] nums1, int[] nums2) {
        Map<Integer, Integer> map = new HashMap<>();
        Stack<Integer> stack = new Stack<>();

        // Procedural pass over nums2 keeps lookup processing highly optimized
        for (int num : nums2) {
            while (!stack.isEmpty() && stack.peek() < num) {
                map.put(stack.pop(), num);
            }
            stack.push(num);
        }

        // Functional stream generation mapping values smoothly
        return Arrays.stream(nums1)
                .map(num -> map.getOrDefault(num, -1))
                .toArray();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Stack + HashMap**| $O(M + N)$ | $O(N)$ | Safe if unshared | Medium | High readability; uses standard API components. |
| **2. Primitive Cache** | $O(M + N)$ | $O(\text{Max\_Val})$ | Safe if unshared | **Zero (Immune)** | Fastest performance; relies strictly on narrow, bounded element limits. |
| **3. Stream Map** | $O(M + N)$ | $O(N)$ | Safe | Medium | Exceptional declarative visibility at the output construction layer. |

---

### Detailed Evaluation Breakdown

#### 1. Linear Time Complexity Guarantee ($O(M+N)$)
Even though the solutions contain nested loops (`while` inside `for`), every single element from `nums2` is pushed onto the stack exactly once and popped from the stack at most once. The total number of internal stack evaluations is strictly bounded by $2N$, ensuring a true linear runtime.

#### 2. Hardware-Level Speed Benefits (Primitive Cache Optimization)
* **Zero GC Overhead:** Completely avoiding Java 8 streams, boxed wrapper classes (`Integer`), or object tables means zero memory allocations are requested on the heap. This prevents Garbage Collection execution pauses entirely, ensuring predictable runtime signatures.
* **Cache Line Locality:** Scanning a single contiguous primitive `int[]` array sequentially allows the internal CPU hardware controllers to seamlessly pull forthcoming memory cells into the high-speed L1/L2 cache before execution cycles look them up, minimizing latency.
