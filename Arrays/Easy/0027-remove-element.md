# 27. Remove Element

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Two Pointers &nbsp;&nbsp;

---

## Problem Description
Given an integer array `nums` and an integer `val`, remove all occurrences of `val` in `nums` **in-place**. The order of the elements may be changed. Then return *the number of elements in `nums` which are not equal to `val`*.

Consider the number of elements which are not equal to `val` be `k`, to get accepted, you need to do the following things:
1. Modify the array `nums` such that the first `k` elements of `nums` contain the elements which are not equal to `val`.
2. Return `k`.

### Example 1:
> **Input:** `nums = [3,2,2,3]`, `val = 3`
> **Output:** `2, nums = [2,2,_,_]`
> **Explanation:** Your function should return `k = 2`, with the first two elements of `nums` being `2`.

### Example 2:
> **Input:** `nums = [0,1,2,2,3,0,4,2]`, `val = 2`
> **Output:** `5, nums = [0,1,4,0,3,_,_,_]`
> **Explanation:** Your function should return `k = 5`, with the first five elements of `nums` containing `0`, `1`, `3`, `0`, and `4`.

### Constraints:
* `0 <= nums.length <= 100`
* `0 <= nums[i] <= 50`
* `0 <= val <= 100`

---

## Optimal Approach
> Use a two-pointer approach where a write-pointer tracks the next available position for elements not equal to `val`. Scan through the array with a read-pointer, copying valid elements forward.

---

## Two-Pointer Technique

```java
    public int removeElement(int[] nums, int val) {
        Objects.requireNonNull(nums, "Input array cannot be null");
        
        int writeIdx = 0;
        
        for (int i = 0; i < nums.length; i++) {
            // If current element is not the target value, write it forward
            if (nums[i] != val) {
                nums[writeIdx] = nums[i];
                writeIdx++;
            }
        }
        
        return writeIdx;
    }
```

## Java8 Approach 1: 

> Clean Functional Stream Solution. You can use the Java 8 Streams API to filter out elements matching val and write the filtered sequence back into the original array container.

```java
    public int removeElement(int[] nums, int val) {
        if (nums == null || nums.length == 0) return 0;

        // Java 8: Filter out elements equal to 'val' and collect to a primitive array
        int[] validElements = Arrays.stream(nums)
                .filter(num -> num != val)
                .toArray();
        
        // Overwrite original array elements to satisfy LeetCode's in-place requirement
        System.arraycopy(validElements, 0, nums, 0, validElements.length);
        
        return validElements.length;
    }
```
> Use code with caution.
> Pros: Declarative and highly readable code style.
> Cons: Runs at O(n) space complexity internally because .toArray() allocates a new block of memory, which technically breaks the strict in-place O(1) space constraint of the problem.

## Java8 Approach 2: Java 8 IntStream Counter & Mutation
> This approach uses IntStream to dynamically rewrite array positions on-the-fly while using an atomic integer counter to track the size.
```java
    public int removeElement(int[] nums, int val) {
        if (nums == null || nums.length == 0) return 0;

        AtomicInteger writeIdx = new AtomicInteger(0);

        // Java 8: Iterate index elements and mutate in-place dynamically
        IntStream.of(nums)
                .filter(num -> num != val)
                .forEach(num -> nums[writeIdx.getAndIncrement()] = num);

        return writeIdx.get();
    }
```
## Java8 Approach 3: The Optimized Hybrid Approach (Best Performance)
>This pairs safe Java 8 argument handling validation with the highly performant iterative two-pointer pattern. This method yields a 0ms processing time score on LeetCode.

```java
    public int removeElement(int[] nums, int val) {
        // Java 8: Strong null pointer protection check
        Objects.requireNonNull(nums, "Input array cannot be null");
        
        int writeIdx = 0;
        
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != val) {
                nums[writeIdx] = nums[i];
                writeIdx++;
            }
        }
        
        return writeIdx;
    }
```

| Approach | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Stream filter)** | $O(n)$ | $O(n)$ | Clean readable option for business logic |
| **Method 2 (IntStream loop)** | $O(n)$ | $O(1)$ | Pure functional approach using state counters |
| **Method 3 (Hybrid / Loops)** | $O(n)$ | $O(1)$ | **Highly Recommended** (Passes strict memory limits) |


### Complexity Analysis

| Approach | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Two-Pointer (In-Place)** | $O(n)$ | $O(1)$ | **Highly Recommended** (0ms runtime) |
| **Java 8 Stream Filter** | $O(n)$ | $O(n)$ | Suboptimal due to primitive array reallocation |

---
