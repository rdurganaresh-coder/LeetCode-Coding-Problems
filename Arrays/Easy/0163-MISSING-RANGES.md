# LeetCode 163: Missing Ranges

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp; 🏷️ Simulation

---

## 📝 Problem Description

You are given an inclusive range `[lower, upper]` and a **sorted unique** integer array `nums`, where all elements are within the inclusive range.

A number `x` is considered **missing** if `x` is in the range `[lower, upper]` and `x` is not in `nums`.

Return *the **shortest sorted** list of ranges that **exactly covers all the missing numbers***. That is, no element of `nums` is included in any of the ranges, and each missing number is covered by one of the ranges.

Each range should be output as a list of two integers `[start, end]`.

### Examples

**Example 1:**
* **Input:** `nums =, lower = 0, upper = 99`
* **Output:** `[[2, 2], [4, 49], [51, 74], [76, 99]]`
* **Explanation:** The missing ranges are:
  * `[2, 2]` covering missing number 2
  * `[4, 49]` covering missing numbers 4 to 49
  * `[51, 74]` covering missing numbers 51 to 74
  * `[76, 99]` covering missing numbers 76 to 99

**Example 2:**
* **Input:** `nums = [], lower = 1, upper = 1`
* **Output:** `[[1, 1]]`
* **Explanation:** The entire range `[1, 1]` is missing.

### Constraints
* $0 \le \text{nums.length} \le 100$
* $-10^9 \le \text{lower} \le \text{upper} \le 10^9$
* $-10^9 \le \text{nums}[i] \le 10^9$
* All the values of `nums` are **unique**.
* `nums` is sorted in ascending order.

---

## 💡 Core Patterns Used

### 1. Virtual Boundary Padding Pattern
* **Concept:** Treating the bounds `lower` and `upper` as implicit start and end anchors to unify logic loop constraints.
* **Mechanism:** Tracks a sequential cursor starting at `lower`. By comparing it to each current element `nums[i]`, missing segments are identified when the array entry creates a positive gap. The final gap is handled symmetrically by imagining a virtual element at `upper + 1`.

### 2. Linear Scan Interval Convergence
* **Concept:** Emitting boundary tracking coordinates dynamically based on element-to-element differences.
* **Mechanism:** Scans elements one by one. If an item is greater than the expected tracking pointer, an interval spanning from the pointer to `item - 1` is immediately generated and added to the tracking payload.

---

## 💻 Java & Java 8 Implementations

### 1. Classic Pointer Scan (Optimal $O(N)$ Time & $O(1)$ Space)
This standard procedural approach loops through the array, using a `long` tracking cursor to prevent overflow errors during boundary incrementing.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class MissingRangesClassic {
    public static List<List<Integer>> findMissingRanges(int[] nums, int lower, int upper) {
        List<List<Integer>> result = new ArrayList<>();
        long nextExpected = lower; // Uses long to bypass integer boundary overflows

        for (int num : nums) {
            // Check if there is a gap between current expected value and the number
            if (num > nextExpected) {
                result.add(Arrays.asList((int) nextExpected, num - 1));
            }
            nextExpected = (long) num + 1;
        }

        // Handle trailing boundary gap up to the upper threshold
        if (nextExpected <= upper) {
            result.add(Arrays.asList((int) nextExpected, upper));
        }

        return result;
    }
}
```

### 2. Segmented Boundary Check (Explicit Edge Case Isolation)
This method separates the logic into three explicit phases: leading gap, inner gaps between adjacent array elements, and trailing gap. It avoids changing types to `long`.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class MissingRangesSegmented {
    public static List<List<Integer>> findMissingRanges(int[] nums, int lower, int upper) {
        List<List<Integer>> result = new ArrayList<>();
        int n = nums.length;

        // 1. Handle empty array edge case
        if (n == 0) {
            result.add(Arrays.asList(lower, upper));
            return result;
        }

        // 2. Head boundary check
        if (nums[0] > lower) {
            result.add(Arrays.asList(lower, nums[0] - 1));
        }

        // 3. Iterative internal array gaps check
        for (int i = 0; i < n - 1; i++) {
            if (nums[i + 1] - nums[i] > 1) {
                result.add(Arrays.asList(nums[i] + 1, nums[i + 1] - 1));
            }
        }

        // 4. Tail boundary check
        if (nums[n - 1] < upper) {
            result.add(Arrays.asList(nums[n - 1] + 1, upper));
        }

        return result;
    }
}
```

### 3. Java 8 Stream Reduction with Index Map
This functional strategy relies on scanning primitive sequence indices via `IntStream` to generate coordinate calculations based on neighbor offsets.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class MissingRangesStreamIndex {
    public static List<List<Integer>> findMissingRanges(int[] nums, int lower, int upper) {
        int n = nums.length;
        if (n == 0) {
            return Arrays.asList(Arrays.asList(lower, upper));
        }

        List<List<Integer>> result = new ArrayList<>();

        // Process head boundary gap
        if (nums[0] > lower) {
            result.add(Arrays.asList(lower, nums[0] - 1));
        }

        // Functional middle gaps pipeline calculation
        List<List<Integer>> midGaps = IntStream.range(0, n - 1)
            .filter(i -> nums[i + 1] > (long) nums[i] + 1)
            .mapToObj(i -> Arrays.asList(nums[i] + 1, nums[i + 1] - 1))
            .collect(Collectors.toList());
        
        result.addAll(midGaps);

        // Process tail boundary gap
        if (nums[n - 1] < upper) {
            result.add(Arrays.asList(nums[n - 1] + 1, upper));
        }

        return result;
    }
}
```

### 4. Java 8 Pure Stream FlatMapping (Unified Padding)
This approach creates a completely uniform pipeline by converting the array into a stream that explicitly appends virtual boundaries, avoiding manual head/tail conditional checks.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class MissingRangesPureStream {
    public static List<List<Integer>> findMissingRanges(int[] nums, int lower, int upper) {
        // Construct an augmented array padded with lower-1 and upper+1 bounds
        long[] padded = IntStream.concat(
            IntStream.concat(IntStream.of(lower - 1), Arrays.stream(nums)), 
            IntStream.of(upper + 1)
        ).mapToLong(i -> i).toArray();

        return IntStream.range(0, padded.length - 1)
            .filter(i -> padded[i + 1] > padded[i] + 1)
            .mapToObj(i -> Arrays.asList((int) (padded[i] + 1), (int) (padded[i + 1] - 1)))
            .collect(Collectors.toList());
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Max Safe Range Bounds | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Scan** | $O(N)$ | $O(1)$ | Safe if unshared | Full Integer Limits ($\pm 2^{31}-1$) | High mutation loop; fastest execution profiles |
| **2. Segmented Check** | $O(N)$ | $O(1)$ | Safe if unshared | Full Integer Limits ($\pm 2^{31}-1$) | Clean pointer separation; avoids switching data types |
| **3. Stream Index Map**| $O(N)$ | $O(1)$ | Safe | Full Integer Limits ($\pm 2^{31}-1$) | Structural segment tracking code isolation |
| **4. Pure Stream Pad** | $O(N)$ | $O(N)$ | Safe | Full Integer Limits ($\pm 2^{31}-1$) | High array allocation memory footprint |

---

### Detailed Evaluation Breakdown

#### 1. Classic Pointer Scan
* **Strengths:** Maximum performance optimization. Executes inside primitive register loops without allocating dummy values or array metadata tracks, conforming instantly to real-time embedded logic limits.
* **Arithmetic Safety:** Casting parameters to `long` via `(long) num + 1` safeguards execution when handling edge cases like `upper = Integer.MAX_VALUE`.

#### 2. Segmented Boundary Check
* **Strengths:** Avoids type promotions to `long` by checking bounds natively using plain array indices. Highly defensive coding style.
* **Weaknesses:** Requires multiple branch conditions up front to prevent `ArrayIndexOutOfBoundsException` on small or empty arrays.

#### 3. Java 8 Stream Index Map
* **Strengths:** Explicit declarative code flow. Separates intermediate list creation from boundary edge checks, making layout modifications easier to apply.
* **Weaknesses:** Requires explicit edge blocks at the start and end of the method to capture leading and trailing missing boundaries correctly.

#### 4. Java 8 Pure Stream FlatMapping
* **Strengths:** Exceptional logical uniformity. Zero manual condition handling logic or nested structural loops needed.
* **Performance Impact:** Higher auxiliary space complexity. It duplicates and expands the incoming integer collection by allocating a temporary `long[] padded` array on the heap. While trivial for LeetCode limits ($N \le 100$), this approach can become a bottleneck when scaling up to large enterprise production streams.
