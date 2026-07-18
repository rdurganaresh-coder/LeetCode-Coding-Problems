# LeetCode 455: Assign Cookies

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Two Pointers** &nbsp;&nbsp; 🏷️ **Greedy** &nbsp;&nbsp; 🏷️ **Sorting**

---

## 📝 Problem Description

Assume you are an awesome parent and want to give your children some cookies. But, you should give each child at most one cookie.

Each child `i` has a greed factor `g[i]`, which is the minimum size of a cookie that the child will be content with; and each cookie `j` has a size `s[j]`. If `s[j] >= g[i]`, we can assign the cookie `j` to the child `i`, and the child `i` will be content. 

Your goal is to maximize the number of your content children and output the maximum number.

### Examples

**Example 1:**
* **Input:** `g =, s = [1,1]`
* **Output:** `1`
* **Explanation:** You have 3 children and 2 cookies. The greed factors of 3 children are 1, 2, 3. 
And even though you have 2 cookies, since their size is both 1, you could only make the child whose greed factor is 1 content.
You need to output 1.

**Example 2:**
* **Input:** `g =, s = [1,2,3]`
* **Output:** `2`
* **Explanation:** You have 2 children and 3 cookies. The greed factors of 2 children are 1, 2. 
You have 3 cookies and their sizes are big enough to gratify all of the children, 
You need to output 2.

### Constraints
* $1 \le \text{g.length} \le 3 \times 10^4$
* $0 \le \text{s.length} \le 3 \times 10^4$
* $1 \le \text{g}[i], \text{s}[j] \le 2^{31} - 1$

---

## 💡 Core Patterns Used

### 1. Greedy Choice Property (Smallest Feasible Match)
* **Concept:** Satisfying the child with the lowest greed factor first using the smallest possible cookie that meets their requirement.
* **Mechanism:** Sorting both arrays allows us to make locally optimal choices. If the smallest available cookie cannot satisfy the least greedy child, it cannot satisfy any other child either, so it can be safely discarded. If it can satisfy the child, we pair them up, maximizing our resource efficiency.

### 2. Dual-Array Two-Pointer Convergence
* **Concept:** Advancing structural cursor pointers independently across sorted primitive boundaries.
* **Mechanism:** Pointer `child` tracks the active child being evaluated, and pointer `cookie` scans the cookie sizes. The cookie pointer increments on every iteration, while the child pointer only steps forward when a valid assignment occurs ($s[cookie] \ge g[child]$). At the end, the `child` index perfectly represents the total number of content children.

---

## 💻 Java & Java 8 Implementations

### 1. Classic Sorting & Two-Pointer Scan (Optimal $O(N \log N)$ Time & $O(1)$ Space)
The standard production configuration. It sorts both arrays in place and uses simple primitive registers to calculate allocations in a single linear sweep.

```java
import java.util.Arrays;

public class AssignCookiesClassic {
    public static int findContentChildren(int[] g, int[] s) {
        if (g == null || s == null || g.length == 0 || s.length == 0) {
            return 0;
        }

        // Sort both arrays to align values chronologically for the greedy strategy
        Arrays.sort(g);
        Arrays.sort(s);

        int childPointer = 0;
        int cookiePointer = 0;

        // Iterate until we run out of children or cookies
        while (childPointer < g.length && cookiePointer < s.length) {
            // If the current cookie satisfies the current child's greed factor
            if (s[cookiePointer] >= g[childPointer]) {
                childPointer++; // Move to the next child
            }
            cookiePointer++; // Always consume or skip the current cookie
        }

        return childPointer; // Index matches the count of successful allocations
    }
}
```

### 2. Java 8 Stream Pipeline Initialization ($O(N \log N)$ Time)
This variant utilizes functional mapping and stream sorting rules to process inputs without modifying the original source array configurations.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class AssignCookiesStream {
    public static int findContentChildren(int[] g, int[] s) {
        if (g == null || s == null || g.length == 0 || s.length == 0) {
            return 0;
        }

        // Sort both streams and map them into ordered primitive arrays
        int[] sortedChildren = Arrays.stream(g).sorted().toArray();
        int[] sortedCookies = Arrays.stream(s).sorted().toArray();

        // Use a single-element array wrapper closure to track child matching state safely
        int[] childIndexWrapper = { 0 };

        Arrays.stream(sortedCookies).forEach(cookieSize -> {
            if (childIndexWrapper[0] < sortedChildren.length && 
                cookieSize >= sortedChildren[childIndexWrapper[0]]) {
                childIndexWrapper[0]++;
            }
        });

        return childIndexWrapper[0];
    }
}
```

### 3. Stateful Stream Accumulator (Pure Functional Style)
This model embeds the greedy state counters inside an isolated custom tracker class container to strictly conform to pure object stream pipelines.

```java
import java.util.Arrays;

public class AssignCookiesStatefulStream {
    private static class CookieState {
        private final int[] sortedChildren;
        private int childIndex = 0;

        public CookieState(int[] children) {
            this.sortedChildren = Arrays.stream(children).sorted().toArray();
        }

        public void evaluateCookie(int cookieSize) {
            if (childIndex < sortedChildren.length && cookieSize >= sortedChildren[childIndex]) {
                childIndex++;
            }
        }

        public CookieState combine(CookieState other) {
            // Fallback tracking handler for sequential stream alignment safety
            throw new UnsupportedOperationException("Parallel reductions are not supported");
        }
    }

    public static int findContentChildren(int[] g, int[] s) {
        if (g == null || s == null || g.length == 0 || s.length == 0) {
            return 0;
        }

        // Stream the sorted cookie array into our custom cookie state engine
        CookieState trackingState = Arrays.stream(s)
            .sorted()
            .collect(() -> new CookieState(g), CookieState::evaluateCookie, CookieState::combine);

        return trackingState.childIndex;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Modifies Input Array? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Scan** | $O(N \log N + M \log M)$ | $O(1)$ (or $O(\log N)$ internal stack) | Safe if unshared | **Yes (In-Place)** | Fastest option by far; uses high-speed memory layout tracking. |
| **2. Stream Init** | $O(N \log N + M \log M)$ | $O(N + M)$ | Safe | No | Preserves immutable inputs; pays an allocation penalty for duplicated array grids. |
| **3. Stateful Stream**| $O(N \log N + M \log M)$ | $O(N + M)$ | Unsafe | No | Exceptional object separation; high functional class mapping overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Classic Sorting & Two-Pointer Scan
* **Strengths:** Utterly optimal execution efficiency. Iterating over primitive arrays utilizing simple registers maximizes CPU spatial data caching layouts, easily completing within minimal execution periods.
* **Side Effects:** Destructive mutation. Overwriting the sorting sequence of incoming properties (`g` and `s`) might cause unintended bugs down the chain if subsequent microservices expect data to match original database log tracks.

#### 2. Java 8 Stream Pipeline Initialization
* **Strengths:** Functional purity. Treating parameters as read-only constants guarantees safety across application layers.
* **Weaknesses:** Increased heap footprint. Invoking `.toArray()` clones elements into fresh allocations on the heap, triggering unnecessary GC cycles on large inputs.

#### 3. Stateful Stream Accumulator
* **Strengths:** High cohesive structure. It cleanly moves loop control statements inside an isolated state machine, completely matching pure object-oriented design standards.
* **Stream Limitation:** Because the logic updates the index mapping sequentially based on continuous elements, appending a `.parallel()` suffix will introduce synchronization hazards and corrupt matching counts.
