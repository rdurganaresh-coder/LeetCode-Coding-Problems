# LeetCode 252: Meeting Rooms

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Sorting

---

## 📝 Problem Description

Given an array of meeting time intervals where `intervals[i] = [start_i, end_i]`, determine if a person could attend all meetings.

### Examples

**Example 1:**
* **Input:** `intervals = [[0,30],[5,10],[15,20]]`
* **Output:** `false`
* **Explanation:** The first meeting `[0,30]` overlaps with the second meeting `[5,10]` and third meeting `[15,20]`, so a person cannot attend all of them.

**Example 2:**
* **Input:** `intervals = [[7,10],[2,4]]`
* **Output:** `true`
* **Explanation:** The two meetings do not overlap in time, so a person can attend both.

### Constraints
* $0 \le \text{intervals.length} \le 10^4$
* $\text{intervals}[i].\text{length} == 2$
* $0 \le \text{start}_i < \text{end}_i \le 10^6$

---

## 💡 Core Patterns Used

### 1. Chronological Sorting Pattern
* **Concept:** Transforming a multi-dimensional timeline arrangement into a linear sequence sorted by start times.
* **Mechanism:** Orders intervals by their starting times ($start_i$). Once sorted, detecting an overlap simplifies to a single linear scan: if the start time of meeting $i+1$ occurs before the end time of meeting $i$ ($start_{i+1} < end_i$), an overlap exists.

### 2. Stream Comparator Mapping Pipeline
* **Concept:** Constructing custom sorting rules within an abstract functional filter stream.
* **Mechanism:** Combines sorting behaviors with a short-circuiting predicate function (`.allMatch()` or `.anyMatch()`) to quickly check for chronological breaks across adjacent elements.

---

## 💻 Java & Java 8 Implementations

### 1. Classic Comparator Sort & Scan (Optimal $O(N \log N)$ Time & $O(1)$ Space)
The definitive industry standard approach. It sorts the array in-place using a custom lambda comparator and checks adjacent items linearly.

```java
import java.util.Arrays;

public class MeetingRoomsClassic {
    public static boolean canAttendMeetings(int[][] intervals) {
        if (intervals == null || intervals.length <= 1) return true;

        // Sort intervals based on their start time
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        // Check if the current meeting starts before the previous one ends
        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] < intervals[i - 1][1]) {
                return false; // Found an overlapping time boundary conflict
            }
        }

        return true;
    }
}
```

### 2. Java 8 Stream Short-Circuit Match (Declarative Style)
This functional variant pipelines index traversal by combining array stream sorting with a clean short-circuit matching condition.

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.stream.IntStream;

public class MeetingRoomsStreamMatch {
    public static boolean canAttendMeetings(int[][] intervals) {
        if (intervals == null || intervals.length <= 1) return true;

        // Sort the array using stream sorting mechanics
        int[][] sortedIntervals = Arrays.stream(intervals)
            .sorted(Comparator.comparingInt(a -> a[0]))
            .toArray(int[][]::new);

        // anyMatch short-circuits instantly upon detecting the first overlapping pair
        boolean hasOverlap = IntStream.range(0, sortedIntervals.length - 1)
            .anyMatch(i -> sortedIntervals[i + 1][0] < sortedIntervals[i][1]);

        return !hasOverlap;
    }
}
```

### 3. Java 8 Priority Queue Simulation
An alternative structural model that reads the raw data matrix into a Min-Heap. It avoids modifying the input array array while tracking timeline events.

```java
import java.util.PriorityQueue;

public class MeetingRoomsPriorityQueue {
    public static boolean canAttendMeetings(int[][] intervals) {
        if (intervals == null || intervals.length <= 1) return true;

        // Min-Heap prioritized strictly by meeting start time
        PriorityQueue<int[]> heap = new PriorityQueue<>(intervals.length, (a, b) -> Integer.compare(a[0], b[0]));
        
        for (int[] interval : intervals) {
            heap.offer(interval);
        }

        int[] current = heap.poll();
        while (!heap.isEmpty()) {
            int[] next = heap.poll();
            // If the next meeting starts before the current one finishes, a clash occurs
            if (next[0] < current[1]) {
                return false;
            }
            current = next;
        }

        return true;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Modifies Input Array? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Sort** | $O(N \log N)$ | $O(1)$ (or $O(\log N)$ internal stack) | Safe if unshared | **Yes (In-Place)** | Fastest execution profile; relies on mutating the incoming array parameters. |
| **2. Stream Match** | $O(N \log N)$ | $O(N)$ | Safe | No | Exceptional declarative style; creates temporary structural array duplicates. |
| **3. Priority Queue**| $O(N \log N)$ | $O(N)$ | Safe if unshared | No | Protects original data references; incurs high object insertion tree-balancing penalties. |

---

### Detailed Evaluation Breakdown

#### 1. Classic Comparator Sort & Scan
* **Strengths:** Maximum performance tuning. Processing the primitive `int[][]` addresses inside the internal cache layer maximizes data locality and eliminates garbage collection overhead.
* **Side Effects:** Destructive mutation. Modifying the initial arrangement order of the array can break downstream workflows if they depend on the original layout.

#### 2. Java 8 Stream Short-Circuit Match
* **Strengths:** Functional purity. It treats incoming variables as immutable records, matching functional safety standards perfectly.
* **Performance Impact:** Higher auxiliary space complexity. Constructing `sortedIntervals` via `.toArray(int[][]::new)` replicates reference pointer arrays across the heap, which adds a minor memory tax on massive timelines.

#### 3. Java 8 Priority Queue Simulation
* **Strengths:** Highly resilient structure if scaled into continuous online data streaming feeds where meetings arrive out of order dynamically.
* **Weaknesses:** Suboptimal heap synchronization overhead. Moving primitive elements into tree structures triggers multiple continuous object reference rebalancing cycles, dragging down raw execution speed compared to direct memory array sorting blocks.
