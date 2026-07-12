# 26. Remove Duplicates from Sorted Array

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Two Pointers &nbsp;&nbsp;

---

## Problem Description
Given an integer array `nums` sorted in **non-decreasing order**, remove the duplicates **in-place** such that each unique element appears only **once**. The **relative order** of the elements should be kept the same. Then return *the number of unique elements in `nums`*.

Consider the number of unique elements of `nums` to be `k`, to get accepted, you need to do the following things:
1. Modify the array `nums` such that the first `k` elements of `nums` contain the unique elements in the order they were present in `nums` initially.
2. Return `k`.

### Example 1:
> **Input:** `nums = [1,1,2]`
> **Output:** `2, nums = [1,2,_]`
> **Explanation:** Your function should return `k = 2`, with the first two elements of `nums` being `1` and `2` respectively. It does not matter what you leave beyond the returned `k` (hence they are underscores).

### Example 2:
> **Input:** `nums = [0,0,1,1,1,2,2,3,3,4]`
> **Output:** `5, nums = [0,1,2,3,4,_,_,_,_,_]`
> **Explanation:** Your function should return `k = 5`, with the first five elements of `nums` being `0`, `1`, `2`, `3`, and `4` respectively.

### Constraints:
* `1 <= nums.length <= 3 * 10^4`
* `-100 <= nums[i] <= 100`
* `nums` is sorted in **non-decreasing** order.

---

## Optimal Approach
> Use a two-pointer technique. Track a slower write-index pointer to update unique elements in-place only when the faster read-index pointer discovers a brand new value.

---

## Two-Pointer Technique 

```java
    public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        // Pointer for the position of the last unique element found
        int uniqueIdx = 0;
        
        for (int i = 1; i < nums.length; i++) {
            // If we find a new unique value, move uniqueIdx forward and update it
            if (nums[i] != nums[uniqueIdx]) {
                uniqueIdx++;
                nums[uniqueIdx] = nums[i];
            }
        }
        
        // Total count of unique elements is index + 1
        return uniqueIdx + 1;
    }
```

## Java 8 Approach 1: Clean Functional Stream Solution (One-Liner Variant)
```
  public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        // Java 8: Filters down to distinct elements and writes them back sequentially
        int[] uniqueNums = Arrays.stream(nums).distinct().toArray();
        
        // Overwrite the original array in-place to satisfy LeetCode constraints
        System.arraycopy(uniqueNums, 0, nums, 0, uniqueNums.length);
        
        return uniqueNums.length;
    }
```

## Java 8 Approach 2: Java 8 IntStream In-Place Filter
```
 public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        // Java 8: Get indices where the element changes value
        int[] uniqueIndices = IntStream.range(0, nums.length)
                .filter(i -> i == 0 || nums[i] != nums[i - 1])
                .toArray();

        // Update the original array array locations sequentially
        IntStream.range(0, uniqueIndices.length)
                .forEach(i -> nums[i] = nums[uniqueIndices[i]]);

        return uniqueIndices.length;
    }
```

## Java 8 Approach 3: The Highly-Optimized Hybrid Approach (Best LeetCode Score)
```
public int removeDuplicates(int[] nums) {
        // Java 8 Objects validation check
        Objects.requireNonNull(nums, "Input array cannot be null");
        if (nums.length == 0) return 0;
        
        int uniqueIdx = 0;
        
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[uniqueIdx]) {
                uniqueIdx++;
                nums[uniqueIdx] = nums[i];
            }
        }
        
        return uniqueIdx + 1;
    }
```


| Approach | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Stream distinct)** | $O(n)$ | $O(n)$ | Good for small microservices / Clean code |
| **Method 2 (IntStream range)** | $O(n)$ | $O(n)$ | Functional approach to index extraction |
| **Method 3 (Hybrid / Loops)** | $O(n)$ | $O(1)$ | **Highly Recommended** (Passes strict constraints) |


### Complexity Analysis
* **Time Complexity:** $O(n)$ where $n$ is the length of the array. We scan the array elements with the fast pointer exactly once.
* **Space Complexity:** $O(1)$ auxiliary constant space because the modifications are done entirely in-place.
