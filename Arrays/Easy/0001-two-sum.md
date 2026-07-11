# 1. Two Sum

`Easy` • `Array` • `Hash Table`

---

## Problem Description
Given an array of integers `nums` and an integer `target`, return *indices of the two numbers such that they add up to `target`*.

You may assume that each input would have ***exactly* one solution**, and you may not use the *same* element twice.

You can return the answer in any order.

### Example 1:
> **Input:** `nums = [2,7,11,15]`, `target = 9`
> **Output:** `[0,1]`
> **Explanation:** Because `nums[0] + nums[1] == 9`, we return `[0, 1]`.

### Example 2:
> **Input:** `nums = [3,2,4]`, `target = 6`
> **Output:** `[1,2]`

### Example 3:
> **Input:** `nums = [3,3]`, `target = 6`
> **Output:** `[0,1]`

### Constraints:
* `2 <= nums.length <= 10^4`
* `-10^9 <= nums[i] <= 10^9`
* `-10^9 <= target <= 10^9`
* **Only one valid answer exists.**

---

# Brute Force Approach:

> **Idea:** Check every pair.
```java
class Solution {

    public int[] twoSum(int[] nums, int target) {

        for(int i=0;i<nums.length;i++){

            for(int j=i+1;j<nums.length;j++){

                if(nums[i]+nums[j]==target){

                    return new int[]{i,j};

                }

            }

        }

        return new int[0];

    }

}
```
### Complexity Analysis
| Complexity | Value |
|------------|-------|
| Time | O(n²) |
| Space | O(1) |
---

# Optimal Approach:
> Use a HashMap.
> Store previously visited numbers.
> For each number,
>check whether
```
target-currentNumber
```
> already exists.

## Algorithm
> Create HashMap.
> Iterate array.
> Calculate complement.
> If complement exists return answer.
> Otherwise store current value.

```java
public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            // Check if the required complement is already tracked
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            
            // Java 8 optimization: putIfAbsent handles duplicate values cleanly
            map.putIfAbsent(nums[i], i);
        }
        
        return new int[] {}; // Return empty array if no match is found
    }
}
```
### Complexity Analysis

| Metric | Value |
|---------|-------|
| Time Complexity | O(n) |
| Space Complexity | O(n) |

#### Why HashMap Works
> HashMap lookup is approximately
```
O(1)
```
> Therefore,instead of checking every element,
> we only perform one lookup per iteration.
---

# Java 8 Approach1 (The Pure Java 8 Streams Solution)

```
public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        return IntStream.range(0, nums.length)
                .filter(i -> {
                    int complement = target - nums[i];
                    if (map.containsKey(complement)) {
                        return true;
                    }
                    map.put(nums[i], i);
                    return false;
                })
                .mapToObj(i -> new int[] { map.get(target - nums[i]), i })
                .findFirst()
                .orElse(new int[] {});
    }
```

## Java 8 Approach 2 (The Parallel Stream Solution (Fast for Massive Datasets))
```
public int[] twoSum(int[] nums, int target) {
        // ConcurrentHashMap prevents race conditions during parallel processing
        ConcurrentHashMap<Integer, Integer> map = new ConcurrentHashMap<>();

        return IntStream.range(0, nums.length)
                .parallel() 
                .filter(i -> {
                    int complement = target - nums[i];
                    // computeIfAbsent acts as an atomic check-and-insert operation
                    Integer foundIdx = map.putIfAbsent(nums[i], i);
                    return map.containsKey(complement) && (foundIdx == null || map.get(complement) != i);
                })
                .mapToObj(i -> new int[] { map.get(target - nums[i]), i })
                .findAny() // findAny is faster than findFirst in parallel streams
                .orElse(new int[] {});
    }
```

## Java 8 Approach 4 (The Functional Collector Solution)
```
public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        Optional<int[]> result = IntStream.range(0, nums.length)
                .mapToObj(i -> {
                    int complement = target - nums[i];
                    if (map.containsKey(complement)) {
                        return new int[] { map.get(complement), i };
                    }
                    // Java 8: Safely computes and caches value if key is missing
                    map.computeIfAbsent(nums[i], key -> i);
                    return null;
                })
                .filter(array -> array != null)
                .findFirst();

        return result.orElse(new int[] {});
    }
```

## Java 8 Approach 3 (Hybrid Stream)
```
public int[] twoSum(int[] nums, int target) {
        // Step 1: Collect indices into a Map using Java 8 Collectors
        Map<Integer, Integer> map = IntStream.range(0, nums.length)
                .boxed()
                .collect(Collectors.toMap(
                    i -> nums[i], 
                    Function.identity(), 
                    (existing, replacement) -> replacement // Keep the latest index for duplicates
                ));

        // Step 2: Use an Optional pipeline to find the matching pair
        return IntStream.range(0, nums.length)
                .filter(i -> map.containsKey(target - nums[i]) && map.get(target - nums[i]) != i)
                .mapToObj(i -> new int[] { i, map.get(target - nums[i]) })
                .findFirst()
                .orElse(new int[] {});
    }
```

### Complexity Analysis
* **Time Complexity:** $O(n)$ because we traverse the array containing $n$ elements exactly once, and lookup time in a hash table is $O(1)$.
* **Space Complexity:** $O(n)$ since the extra space required depends on the number of items stored in the hash table, which stores at most $n$ elements.
