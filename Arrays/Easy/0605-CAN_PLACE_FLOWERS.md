# LeetCode 605: Can Place Flowers

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Greedy** &nbsp;&nbsp; 🏷️ **Simulation**

---

## 📝 Problem Description

You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.

Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` *if `n` new flowers can be planted in the `flowerbed` without violating the no-adjacent-flowers rule and `false` otherwise*.

### Examples

**Example 1:**
* **Input:** `flowerbed =, n = 1`
* **Output:** `true`
* **Explanation:** We can plant exactly one flower at index 2, turning the bed into `[1,0,1,0,1]`.

**Example 2:**
* **Input:** `flowerbed =, n = 2`
* **Output:** `false`
* **Explanation:** Planting at index 2 leaves no other valid slots. It is impossible to place 2 flowers safely.

### Constraints
* $1 \le \text{flowerbed.length} \le 2 \times 10^4$
* `flowerbed[i]` is `0` or `1`.
* `flowerbed` contains no two adjacent flowers initially.
* $0 \le n \le \text{flowerbed.length}$

---

## 💡 Core Patterns Used

### 1. Greedy Single-Pass Scanning Pattern
* **Concept:** Seizing every single valid planting opportunity the absolute millisecond it becomes available from left to right.
* **Mechanism:** Iterates forward across the array. For any cell `i`, a flower can be safely placed if and only if three continuous conditions are satisfied simultaneously:
  1. The current cell is empty (`flowerbed[i] == 0`).
  2. The left neighbor is empty or it is the head boundary edge (`i == 0 || flowerbed[i - 1] == 0`).
  3. The right neighbor is empty or it is the tail boundary edge (`i == flowerbed.length - 1 || flowerbed[i + 1] == 0`).
  When these rules match, a flower is greedily committed in-place (`flowerbed[i] = 1`) and our objective requirement count `n` decrements.

### 2. Early-Exit Short-Circuiting
* **Concept:** Intercepting loop executions early when logical goals are secured ahead of schedule.
* **Mechanism:** Checks `n <= 0` at the very top of the function and immediately after any successful planting action. This skips parsing the rest of the array if our budget is already satisfied.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Greedy Scan (Standard DSA Style)
The definitive production standard approach. It sweeps the array linearly using primitive registers, resulting in maximum speed and zero heap allocations.

```java
public class CanPlaceFlowersClassic {
    public static boolean canPlaceFlowers(int[] flowerbed, int n) {
        // Short-circuit: If no flowers are requested, the requirement is satisfied immediately
        if (n <= 0) return true;
        if (flowerbed == null || flowerbed.length == 0) return false;

        int len = flowerbed.length;

        for (int i = 0; i < len; i++) {
            if (flowerbed[i] == 0) {
                // Check if left and right neighbors are safely vacant (respecting boundary edges)
                boolean leftEmpty = (i == 0 || flowerbed[i - 1] == 0);
                boolean rightEmpty = (i == len - 1 || flowerbed[i + 1] == 0);

                if (leftEmpty && rightEmpty) {
                    flowerbed[i] = 1; // Greedily commit the plot placement in-place
                    n--;

                    // Early exit if all requested flowers are successfully planted
                    if (n <= 0) {
                        return true;
                    }
                }
            }
        }

        return n <= 0;
    }
}
```

### 2. Zero-Mutation Variable Jump Optimization (Alternative DSA Pattern)
An elegant procedural variant that skips altering the original input array completely by mathematically skipping indexes forward when an empty zone is evaluated.

```java
public class CanPlaceFlowersNoMutation {
    public static boolean canPlaceFlowers(int[] flowerbed, int n) {
        if (n <= 0) return true;
        
        int i = 0;
        int len = flowerbed.length;

        while (i < len) {
            if (flowerbed[i] == 1) {
                // If a flower exists, the next plot is automatically blocked. Jump past it.
                i += 2;
            } else if (i == len - 1 || flowerbed[i + 1] == 0) {
                // Current is 0, and right neighbor is also 0 (or end of array). 
                // Since we skip through indices safely, the left side is guaranteed clean.
                n--;
                if (n <= 0) return true;
                i += 2; // Jump past the newly occupied slot's blocked neighbor zone
            } else {
                // Current is 0, but right neighbor is 1. Jump directly into that flower's zone.
                i += 3;
            }
        }
        return n <= 0;
    }
}
```

### 3. Java 8 Stateful Stream Accumulator (Pure Functional Style)
This model embeds the timeline state counters inside an isolated custom tracker class container to strictly conform to pure object stream pipelines.

```java
import java.util.Arrays;

public class CanPlaceFlowersStream {
    private static class PlantingState {
        int flowersRemaining;
        int prevVal = 0; // Pretend index -1 was empty (0) to handle boundary logic smoothly

        public PlantingState(int n) {
            this.flowersRemaining = n;
        }

        // Sequential item processing logic block
        public void accept(int current, int next) {
            if (flowersRemaining <= 0) return;

            if (current == 0 && prevVal == 0 && next == 0) {
                flowersRemaining--;
                prevVal = 1; // Mark as occupied to block the immediate next item sequence
            } else {
                prevVal = current;
            }
        }
    }

    public static boolean canPlaceFlowers(int[] flowerbed, int n) {
        if (n <= 0) return true;
        if (flowerbed == null || flowerbed.length == 0) return false;

        int len = flowerbed.length;
        PlantingState state = new PlantingState(n);

        // Map indices to context look-ahead checks using IntStream mapping
        java.util.stream.IntStream.range(0, len).forEach(i -> {
            int current = flowerbed[i];
            int next = (i == len - 1) ? 0 : flowerbed[i + 1]; // Virtual edge tracking padding
            state.accept(current, next);
        });

        return state.flowersRemaining <= 0;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Total Heap Allocations | Modifies Input Array? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Greedy Scan** | $O(N)$ | $O(1)$ | **Zero (Immune)** | **Yes (In-Place)** | Fastest execution profiles; mutates incoming parameters. |
| **2. No-Mutation Jump**| $O(N)$ | $O(1)$ | **Zero (Immune)** | No | Peak speed with pure read-only safety; complex index jumping paths. |
| **3. Functional Stream**| $O(N)$ | $O(1)$ | Low | No | High cohesive structure; minor object tracker initialization overhead. |

---

### Detailed Evaluation Breakdown

#### 1. The Value of Primitive Register Execution
Traditional naive solutions might clone the array up front or append dummy padding values (`0`) at both ends to simplify boundary conditions. While intuitive, copying an array of length $2 \times 10^4$ introduces heap allocation costs and cache line missing penalties. 

By tracking boundaries explicitly through logic bounds (`i == 0 || flowerbed[i-1] == 0`), Implementation 1 and 2 operate on raw memory with absolute zero allocation footprint overhead.

#### 2. Index-Jumping Arithmetic Optimization
Implementation 2 demonstrates a classic DSA stride pattern. By evaluating cells strategically, it avoids checking elements sequentially when constraints indicate a zone is blocked:
* Encountering a `1` means index `i+1` is unusable. We skip both indices entirely via `i += 2`.
* This compresses instruction iteration counts significantly under heavy-density plots, outperforming flat element-by-element loops.

#### 3. Java 8 Stateful Stream Constraints
* **Strengths:** High expressive encapsulation. It isolates internal coordinate checks inside a distinct functional tracker frame, avoiding global flag leaks.
* **Stream Limitation:** Because structural adjustments depend strictly on sequential history context (`prevVal`), appending a `.parallel()` suffix will induce data race hazards and corrupt the outcome.
