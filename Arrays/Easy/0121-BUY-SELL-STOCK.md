# LeetCode 121: Best Time to Buy and Sell Stock

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays**

---

## 📝 Problem Description

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{th}$ day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return *the maximum profit you can achieve from this transaction*. If you cannot achieve any profit, return `0`.

### Examples

**Example 1:**
* **Input:** `prices = [7,1,5,3,6,4]`
* **Output:** `5`
* **Explanation:** Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6 - 1 = 5. Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.

**Example 2:**
* **Input:** `prices = [7,6,4,3,1]`
* **Output:** `0`
* **Explanation:** In this case, no transactions are done and the max profit = 0.

### Constraints
* $1 \le \text{prices.length} \le 10^5$
* $0 \le \text{prices}[i] \le 10^4$

---

## 💡 Core Patterns Used

### 1. Single-Pass Greedy Pattern
* **Concept:** Maintaining a running minimum historical value to find global maximum difference states instantly on the fly.
* **Mechanism:** Tracks `minPrice` sequentially while updating `maxProfit` via a single forward array scan.

### 2. Kadane's Algorithm Variant
* **Concept:** Maximizing local running subarrays by resetting intermediate accumulator offsets when values drop below zero boundaries.
* **Mechanism:** Converts raw historical assets into relative day-over-day price differences to isolate maximum contiguous sum sequences.

### 3. Stateful Stream Accumulation
* **Concept:** Simulating localized state engines using custom object containers within sequential stream reductions.
* **Mechanism:** Overcomes standard stateless pipeline constraints by modifying an external state wrapper closure dynamically.

---

## 💻 Java & Java 8 Implementations

### 1. Single Pass Greedy Approach (Optimal $O(N)$ Time)
This traditional approach tracks the lowest price seen so far and calculates the potential profit for each subsequent day, updating the maximum profit dynamically.

```java
public class StockClassicGreedy {
    public static int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) return 0;

        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;

        for (int price : prices) {
            if (price < minPrice) {
                minPrice = price; // Update the historical lowest price
            } else if (price - minPrice > maxProfit) {
                maxProfit = price - minPrice; // Update the maximum profit seen so far
            }
        }
        return maxProfit;
    }
}
```

### 2. Kadane's Algorithm Variant (Maximum Subarray Difference)
This approach transforms the stock prices into an array of daily price differences. Finding the maximum profit then becomes equivalent to finding the maximum subarray sum.

```java
public class StockKadanesAlgorithm {
    public static int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) return 0;

        int currentMax = 0;
        int globalMax = 0;

        for (int i = 1; i < prices.length; i++) {
            // Calculate daily price difference and add to the current running subarray
            currentMax += prices[i] - prices[i - 1];
            // Reset to 0 if the running sum drops below 0
            currentMax = Math.max(0, currentMax);
            globalMax = Math.max(globalMax, currentMax);
        }
        return globalMax;
    }
}
```

### 3. Java 8 Functional IntStream Reduction (Stateful Object Accumulator)
Since functional streams are inherently stateless, this approach maps the streaming pipeline using a custom state object accumulator to track both the minimum price and the max profit in a single pass.

```java
import java.util.Arrays;

public class StockStreamAccumulator {
    // Custom container to track stream state across iterations
    private static class StockState {
        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;
    }

    public static int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) return 0;

        // Use reduce to mutate the localized state object sequentially
        StockState finalState = Arrays.stream(prices)
            .boxed()
            .reduce(new StockState(), 
                (state, price) -> {
                    state.minPrice = Math.min(state.minPrice, price);
                    state.maxProfit = Math.max(state.maxProfit, price - state.minPrice);
                    return state;
                }, 
                (state1, state2) -> state1); // Combiner (unused in sequential streams)

        return finalState.maxProfit;
    }
}
```

### 4. Java 8 IntStream Index Range Scan
This functional approach scans across the primitive indices of the array. It computes the minimum value on-the-fly by tracking the state dynamically through a custom mapping function.

```java
import java.util.stream.IntStream;

public class StockStreamIndexScan {
    public static int maxProfit(int[] prices) {
        if (prices == null || prices.length < 2) return 0;

        int[] minPrice = { prices[0] }; // Single-element array wrapper to allow closure mutation

        return IntStream.range(1, prices.length)
            .map(i -> {
                int currentProfit = prices[i] - minPrice[0];
                minPrice[0] = Math.min(minPrice[0], prices[i]);
                return currentProfit;
            })
            .max() // Grab the absolute peak profit calculated
            .orElse(0); // Return 0 if no profit can be generated
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Safe Input Array Size | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Greedy** | $O(N)$ | $O(1)$ | Safe if unshared | Limited only by JVM memory | Fastest execution speed with no overhead |
| **2. Kadane's Variant** | $O(N)$ | $O(1)$ | Safe if unshared | Limited only by JVM memory | Slightly higher arithmetic operations than greedy |
| **3. Stream Accumulator**| $O(N)$ | $O(1)$ | Safe | Limited only by JVM memory | High boxing overhead (`int` to `Integer`) |
| **4. Stream Index Scan** | $O(N)$ | $O(1)$ | Unsafe for Parallel | Limited only by JVM memory | Side-effect mutation inside stream violates pure functional style |

---

### Detailed Evaluation Breakdown

#### 1. Single Pass Greedy Approach
* **Strengths:** High execution efficiency. Loops directly through primitive array values without initializing objects, achieving the lowest possible CPU cycle overhead.
* **Weaknesses:** Relies on imperative variable mutation tracking blocks, which some clean-code pipelines flag as old-school architecture.

#### 2. Kadane's Algorithm Variant
* **Strengths:** Highly adaptable. Easily extended to solve multi-transaction or bounded-transaction variants (e.g., LeetCode 122 or 123) by stacking regional running array boundaries.
* **Performance Impact:** Negligibly slower than the classic greedy method due to calculating daily relative array offsets (`prices[i] - prices[i - 1]`) at each loop iteration.

#### 3. Java 8 Functional IntStream Reduction
* **Strengths:** Implements a pure stream pipeline that matches standard factory map-reduce patterns cleanly.
* **Weaknesses:** Memory allocation penalties. `Arrays.stream(prices).boxed()` forces every single array index to box from a primitive `int` into an `Integer` object wrapper, triggering rapid garbage collection cycles on huge inputs.

#### 4. Java 8 IntStream Index Range Scan
* **Strengths:** Leverages primitive index mappings, completely avoiding the memory allocation penalties caused by object wrapper boxing.
* **Stream Pitfall:** Strictly dependent on sequential stream execution. The array wrapper closure (`minPrice`) forces stateful changes across records. If updated to run on a `.parallel()` pipeline, tracking indices will collide, generating race conditions and incorrect results.
