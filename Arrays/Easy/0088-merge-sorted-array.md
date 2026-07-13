# 88. Merge Sorted Array

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Two Pointers &nbsp;&nbsp;

---

## Problem Description
You are given two integer arrays `nums1` and `nums2`, sorted in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.

**Merge** `nums1` and `nums2` into a single array sorted in **non-decreasing order**.

The final sorted array should not be returned by the function, but instead be *stored inside the array `nums1`*. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`.

### Example 1:
> **Input:** `nums1 = [1,2,3,0,0,0]`, `m = 3`, `nums2 = [2,5,6]`, `n = 3`
> **Output:** `[1,2,2,3,5,6]`
> **Explanation:** The arrays we are merging are `[1,2,3]` and `[2,5,6]`. The result of the merge is `[1,2,2,3,5,6]` with the underlined elements coming from `nums1`.

### Example 2:
> **Input:** `nums1 = [1]`, `m = 1`, `nums2 = []`, `n = 0`
> **Output:** `[1]`
> **Explanation:** The arrays we are merging are `[1]` and `[]`. The result of the merge is `[1]`.

### Example 3:
> **Input:** `nums1 = [0]`, `m = 0`, `nums2 = [1]`, `n = 1`
> **Output:** `[1]`
> **Explanation:** The arrays we are merging are `[]` and `[1]`. The result of the merge is `[1]`. Note that because `m = 0`, there are no elements in `nums1`. The `0` is only there to ensure the merge result can fit in `nums1`.

### Constraints:
* `nums1.length == m + n`
* `nums2.length == n`
* `0 <= m, n <= 200`
* `1 <= m + n <= 200`
* `-10^9 <= nums1[i], nums2[j] <= 10^9`

---

## Optimal Approach
> Start filling `nums1` from the **very back** (index `m + n - 1`) moving forwards. By comparing the largest available numbers at the end of both valid boundaries, you can overwrite the trailing `0` elements without needing a separate temporary array buffer.

---

## Method 1: Three-Pointer Reverse Scan (Recommended)
This approach handles the element comparison backwards from right to left. It prevents item shifting, runs at a pure **0ms** processing speed, and respects the constant space constraint perfectly.

```java
import java.util.Objects;

class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        Objects.requireNonNull(nums1, "nums1 cannot be null");
        Objects.requireNonNull(nums2, "nums2 cannot be null");

        int p1 = m - 1;      // Pointer for end of initialized elements in nums1
        int p2 = n - 1;      // Pointer for end of nums2
        int pWrite = m + n - 1; // Pointer for the write position at the back of nums1

        // Compare elements from the back and move the larger one to the write position
        while (p2 >= 0) {
            if (p1 >= 0 && nums1[p1] > nums2[p2]) {
                nums1[pWrite] = nums1[p1];
                p1--;
            } else {
                nums1[pWrite] = nums2[p2];
                p2--;
            }
            pWrite--;
        }
    }
}
```

---

## Method 2: System Arraycopy with Stream Sort (Java 8 API)
This method drops `nums2` directly into the open buffer slot at the back of `nums1` using low-level memory copies, then cleans up the positioning using internal primitive array sorting bytecode.

```java
import java.util.Arrays;

class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        // Step 1: Copy nums2 directly into the tail section of nums1
        System.arraycopy(nums2, 0, nums1, m, n);
        
        // Step 2: Sort the combined layout in place using dual-pivot quicksort
        Arrays.sort(nums1); 
    }
}
```
* **Pros:** Minimal lines of code; delegates structural reorganization to native compiled system methods.
* **Cons:** Increases time complexity to **$O((m+n) \log(m+n))$** due to the sorting operation.

---

## Method 3: Functional Stream Rebuild Pipeline
If you wanted to see how this looks inside a continuous Java 8 stream transformation, you can isolate valid parts, concatenate them, sort them functionally, and collect them back.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        // Java 8: Slice valid subarrays, merge, sort, and output to a primitive array
        int[] sortedResult = IntStream.concat(
                    Arrays.stream(nums1).limit(m), 
                    Arrays.stream(nums2).limit(n)
                )
                .sorted()
                .toArray();

        // Write the sorted sequence back into the reference container
        System.arraycopy(sortedResult, 0, nums1, 0, sortedResult.length);
    }
}
```
* **Cons:** Bypasses in-place limitations by generating an internal heap allocation array buffer under the hood ($O(m+n)$ space complexity overhead).

---

### Performance & Constraint Evaluation

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Three-Pointer Reverse)** | $O(m + n)$ | $O(1)$ | **Highly Recommended** (Optimal 0ms runtime) |
| **Method 2 (`System.arraycopy`)** | $O((m+n) \log(m+n))$ | $O(1)$ | Compact code, but slower execution due to sorting |
| **Method 3 (`IntStream` Pipeline)** | $O((m+n) \log(m+n))$ | $O(m + n)$ | Suboptimal due to primitive pipeline buffer streaming |

---
