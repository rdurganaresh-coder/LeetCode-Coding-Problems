# LeetCode 495: Teemo Attacking

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Simulation** &nbsp;&nbsp; 🏷️ **Streams**

---

## 📝 Problem Description

Our hero Teemo is attacking an enemy Ashe with poison attacks! When Teemo attacks Ashe, Ashe gets poisoned for exactly `duration` seconds. More formally, an attack at second `t` will mean Ashe is poisoned during the inclusive time interval `[t, t + duration - 1]`. 

If Teemo attacks again before the poison effect ends, the timer for it is reset, and the poison effect will end `duration` seconds after the new attack.

You are given a non-decreasing integer array `timeSeries`, where `timeSeries[i]` denotes that Teemo attacks Ashe at second `timeSeries[i]`, and an integer `duration`.

Return *the total number of seconds that Ashe is poisoned*.

---

## 💡 Core Patterns Used

### 1. One-Pass Interval Delta Pattern
* **Concept:** Evaluating the time gap between adjacent attacks to determine if the poison duration was cut short.
* **Mechanism:** Iterates forward through `timeSeries`. For any two consecutive attacks, the poisoned duration is bounded by $\min(\text{duration}, \text{timeSeries}[i+1] - \text{timeSeries}[i])$. The absolute final attack is never interrupted and always adds the full `duration`.

### 2. Stateful Stream Custom Reduction
* **Concept:** Maintaining a running timeline state machine within an inherently stateless functional stream pipeline.
* **Mechanism:** Uses a custom state object wrapper to pass context (the timestamp of the previous attack) along the stream sequence, aggregating the active duration dynamically.

---

## 💻 Java & Java 8 Implementations

### 1. Classical Math.min Optimization (Standard DSA)
An elegant optimization of the basic simulation that replaces explicit `if/else` logic blocks with high-speed primitive `Math.min` register operations.

```java
public class TeemoMathMin {
    public static int findPoisonedDuration(int[] timeSeries, int duration) {
        if (numsNullOrEmpty(timeSeries) || duration == 0) return 0;

        int totalDuration = 0;
        int n = timeSeries.length;

        for (int i = 0; i < n - 1; i++) {
            // Add whichever is smaller: the full duration or the exact gap until the next attack
            totalDuration += Math.min(duration, timeSeries[i + 1] - timeSeries[i]);
        }

        // Complete the calculation by adding the final uninterrupted attack payload
        return totalDuration + duration;
    }

    private static boolean numsNullOrEmpty(int[] arr) {
        return arr == null || arr.length == 0;
    }
}
```

### 2. Java 8 Functional Stream Index Scan ($O(N)$ Time)
A clean declarative solution using an `IntStream` to generate index values, mapping them to structural interval calculations on the fly.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

public class TeemoStreamIndexScan {
    public static int findPoisonedDuration(int[] timeSeries, int duration) {
        if (timeSeries == null || timeSeries.length == 0 || duration == 0) return 0;

        // Map indices 0 to N-2 to their actual contribution and sum them up
        int innerDuration = IntStream.range(0, timeSeries.length - 1)
            .map(i -> Math.min(duration, timeSeries[i + 1] - timeSeries[i]))
            .sum();

        return innerDuration + duration;
    }
}
```

### 3. Stateful Stream Collector Reduction (Pure Functional Style)
This variant encapsulates timeline state parameters within an isolated tracking container class to strictly conform to standard map-reduce functional stream workflows.

```java
import java.util.Arrays;

public class TeemoStatefulStream {
    private static class PoisonAccumulator {
        int totalPoisonTime = 0;
        int lastAttackTime = -1;
        int durationSetting;

        public PoisonAccumulator(int duration) {
            this.durationSetting = duration;
        }

        public void accept(int currentAttack) {
            if (lastAttackTime == -1) {
                // First attack init edge case
                totalPoisonTime += durationSetting;
            } else {
                int delta = currentAttack - lastAttackTime;
                if (delta < durationSetting) {
                    // Overlap found: Add only the difference to extend the window correctly
                    totalPoisonTime += delta;
                } else {
                    // Clear gap found: Add the complete standalone duration block
                    totalPoisonTime += durationSetting;
                }
            }
            lastAttackTime = currentAttack;
        }

        public PoisonAccumulator combine(PoisonAccumulator other) {
            // Combined fallback logic wrapper for sequential fallback safety loops
            throw new UnsupportedOperationException("Parallel reductions not supported");
        }
    }

    public static int findPoisonedDuration(int[] timeSeries, int duration) {
        if (timeSeries == null || timeSeries.length == 0 || duration == 0) return 0;

        PoisonAccumulator finalState = Arrays.stream(timeSeries)
            .collect(() -> new PoisonAccumulator(duration), PoisonAccumulator::accept, PoisonAccumulator::combine);

        return finalState.totalPoisonTime;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | GC Heap Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Math.min DSA** | $O(N)$ | $O(1)$ | Safe if unshared | **Zero (Immune)** | Maximum possible performance profile; highly optimized for CPU cache lines. |
| **2. Stream Index Scan** | $O(N)$ | $O(1)$ | Safe | **Zero (Immune)** | Excellent balance of clean, single-line visibility and functional style without boxing costs. |
| **3. Stateful Stream** | $O(N)$ | $O(1)$ | Unsafe | Low | Clean domain object separation; introduces minor object instantiation overhead. |

---

### Detailed Evaluation Breakdown

#### 1. Math.min DSA Optimization
* **Strengths:** High execution efficiency. Replacing procedural conditional `if/else` checks with native `Math.min` assignments allows modern compilers to map commands directly into high-speed branchless assembly instructions (`CMOV` codes). This minimizes CPU branch misprediction latency penalties.

#### 2. Java 8 Stream Index Scan
* **Strengths:** Excellent declarative representation. Because `IntStream.range()` iterates over primitive data types directly, it skips autoboxing completely. This maintains the same lightweight memory usage as standard loops while offering a highly readable functional syntax.

#### 3. Stateful Stream Collector Reduction
* **Strengths:** Completely encapsulates timeline mutation parameters inside an isolated class. This isolates tracking state context smoothly, aligning with modern software development patterns.
* **Stream Pitfall:** Strictly bounded to sequential execution profiles. Because the state accumulator depends entirely on chronological context shifts (`lastAttackTime`), adding a `.parallel()` suffix will introduce race conditions and corrupt the accumulated total.
