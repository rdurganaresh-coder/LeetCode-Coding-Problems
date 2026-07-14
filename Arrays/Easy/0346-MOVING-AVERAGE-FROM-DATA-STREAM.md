# LeetCode 346: Moving Average from Data Stream

> 🟢 Easy &nbsp;&nbsp; 🏷️ Design &nbsp;&nbsp; 🏷️ Queue &nbsp;&nbsp; 🏷️ Sliding Window &nbsp;&nbsp; 🏷️ Data Stream

---

## 📝 Problem Description

Given a stream of integers and a window size, calculate the moving average of all integers inside the sliding window.

Implement the `MovingAverage` class:
* `MovingAverage(int size)` Initializes the object with the size of the window.
* `double next(int val)` Returns the moving average of the last `size` values of the stream.

### Examples

**Example 1:**
```java
MovingAverage movingAverage = new MovingAverage(3);
movingAverage.next(1); // return 1.0 = 1 / 1
movingAverage.next(10); // return 5.5 = (1 + 10) / 2
movingAverage.next(3); // return 4.66667 = (1 + 10 + 3) / 3
movingAverage.next(5); // return 6.0 = (10 + 3 + 5) / 3
```

### Constraints
* $1 \le \text{size} \le 1000$
* $-10^5 \le \text{val} \le 10^5$
* At most $10^4$ calls will be made to `next`.

---

## 💡 Core Patterns Used

### 1. First-In-First-Out (FIFO) Window Eviction Pattern
* **Concept:** Maintaining a rolling temporal memory cache using double-ended nodes or linear rings to strictly enforce size constraints.
* **Mechanism:** When a new element arrives via `.next(val)`, it is added to the tail of a data tracking collection. If the window limit is exceeded ($\text{collection.size()} > \text{maxSize}$), the oldest element at the head of the collection is evicted.

### 2. Constant-Time Window Sum Tracking
* **Concept:** Modifying an atomic scalar running total continuously to prevent recalculation overhead during range queries.
* **Mechanism:** Rather than summing up all elements inside the window on every single API call (which costs $O(\text{size})$ time), an internal `runningSum` accumulator variable is updated dynamically:
  $$\text{runningSum}_{\text{new}} = \text{runningSum}_{\text{old}} + \text{val}_{\text{new}} - \text{val}_{\text{evicted}}$$
  This ensures that calculating the average remains an $O(1)$ constant-time calculation.

---

## 💻 Java & Java 8 Implementations

### 1. Optimal Deque Interface Tracking (Standard Industry Approach)
This approach combines a standard `ArrayDeque` with constant-time running sum modifications to maximize structural efficiency and code clarity.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class MovingAverageDeque {
    private final Deque<Integer> window;
    private final int maxSize;
    private double runningSum;

    public MovingAverageDeque(int size) {
        this.maxSize = size;
        this.window = new ArrayDeque<>(size); // Pre-allocate initial boundary size safely
        this.runningSum = 0.0;
    }

    // O(1) Time complexity per transaction pass
    public double next(int val) {
        runningSum += val;
        window.offerLast(val);

        // If the window has expanded past the maximum size limit, evict the stale head node
        if (window.size() > maxSize) {
            runningSum -= window.pollFirst();
        }

        // Divide total accumulated assets against active structural bounds
        return runningSum / window.size();
    }
}
```

### 2. Low-Level Primitive Circular Ring Buffer (Maximum Performance)
This approach removes object allocations completely by utilizing a fixed-size primitive `int[]` array alongside a tracking index cursor, offering unmatched performance for zero-latency execution loops.

```java
public class MovingAverageCircularBuffer {
    private final int[] buffer;
    private final int maxSize;
    private int elementCount;
    private int writeHead;
    private double runningSum;

    public MovingAverageCircularBuffer(int size) {
        this.maxSize = size;
        this.buffer = new int[size];
        this.writeHead = 0;
        this.elementCount = 0;
        this.runningSum = 0.0;
    }

    // O(1) Time complexity with zero memory management pressure
    public double next(int val) {
        if (elementCount < maxSize) {
            elementCount++;
        } else {
            // Overwriting a cell implies evicting its historical value from the active sum
            runningSum -= buffer[writeHead];
        }

        runningSum += val;
        buffer[writeHead] = val;
        
        // Loop cursor pointer back around to index 0 using basic ring modulo arithmetic
        writeHead = (writeHead + 1) % maxSize;

        return runningSum / elementCount;
    }
}
```

### 3. Java 8 Functional Stream Collection Pipeline
This declarative variant uses a standard queue data layout but abstracts away terminal loop tracking structures using functional wrapper pipelines.

```java
import java.util.LinkedList;
import java.util.Queue;

public class MovingAverageStream {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int maxSize;

    public MovingAverageStream(int size) {
        this.maxSize = size;
    }

    // Functional representation with minor object tracking overhead
    public double next(int val) {
        queue.offer(val);
        if (queue.size() > maxSize) {
            queue.poll();
        }

        // Map primitive objects safely into terminal average collections
        return queue.stream()
                    .mapToInt(Integer::intValue)
                    .average()
                    .orElse(0.0);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | `next` Time Complexity | Space Complexity (Auxiliary) | GC Memory Management Pressure | Primitive Safety Array Profile | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Deque Tracking** | $O(1)$ | $O(\text{size})$ | Low | No (Uses boxed integer wrappers) | Highly standardized and maintainable code layout. |
| **2. Circular Buffer**| $O(1)$ | $O(\text{size})$ | **Zero (Immune)** | **Yes (Maximum Performance)** | Lightning-fast low-level register optimization; marginally harder to modify. |
| **3. Functional Stream**| $O(\text{size})$ | $O(\text{size})$ | High | No | Clean declarative visibility; suffers from substantial streaming query performance degradation. |

---

### Detailed Evaluation Breakdown

#### 1. Optimal Deque Interface Tracking
* **Strengths:** High execution determinism. `ArrayDeque` is far superior to standard `LinkedList` structures as a backing data store because it avoids node-pointer wrapping metadata costs, preserving solid heap locality.
* **Limitations:** Every call to `.next(val)` boxes the incoming primitive parameter into an `Integer` object wrapper, which can prompt routine memory sweeps if processed continuously inside huge cloud logging layers.

#### 2. Low-Level Primitive Circular Ring Buffer
* **Strengths:** Unrivaled engineering benchmark champion. Because it allocates a plain primitive `int[]` array exactly once during constructor execution, it creates zero objects on the heap during active lookup streaming cycles. This prevents GC garbage compilation freezes entirely, which is essential for telemetry and high-frequency trading platforms.
* **Optimization Secret:** Bypassing pointer jumps allows the CPU to fetch neighboring array elements ahead of time via internal hardware prefetching tracks.

#### 3. Java 8 Functional Stream Collection Pipeline
* **Strengths:** Elegant, visual, concise codebase. Bypasses tracking accumulator variables inside object boundaries completely.
* **Performance Malfunction:** **Fails real-world high-throughput performance requirements.** Because it recalculates the entire sliding window array layout from scratch via `.stream().average()` on every incoming message, its time complexity scales down to $O(\text{size})$. If window limits scale up to $k = 10^3$, this approach drops throughput significantly compared to the constant-time ($O(1)$) alternatives.
