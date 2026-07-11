# 1. Two Sum

> 🟢 Easy &nbsp;&nbsp; 🏷️ Topics &nbsp;&nbsp; 🏢 Companies &nbsp;&nbsp; 💡 Hint

---

## Problem

Given an array of integers `nums` and an integer `target`, return **indices** of the two numbers such that they add up to `target`.

You may assume that each input would have **exactly one solution**, and you **may not use the same element twice**.

You can return the answer in **any order**.

---

## Example 1

**Input**

```text
nums = [2,7,11,15]
target = 9
```

**Output**

```text
[0,1]
```

**Explanation**

Because `nums[0] + nums[1] == 9`, we return `[0,1]`.

---

## Example 2

**Input**

```text
nums = [3,2,4]
target = 6
```

**Output**

```text
[1,2]
```

**Explanation**

Because `nums[1] + nums[2] == 6`, we return `[1,2]`.

---

## Example 3

**Input**

```text
nums = [3,3]
target = 6
```

**Output**

```text
[0,1]
```

**Explanation**

Because `nums[0] + nums[1] == 6`, we return `[0,1]`.

---

## Constraints

```text
2 <= nums.length <= 10^4
-10^9 <= nums[i] <= 10^9
-10^9 <= target <= 10^9
Only one valid answer exists.
```

---

## Follow-up

Can you come up with an algorithm that is **better than O(n²)**?

---

## Tags

- Array
- Hash Table

---

## Companies

- Amazon
- Google
- Microsoft
- Meta
- Apple
- Bloomberg
- Adobe

---

## Hints

1. Try using a **HashMap** to store numbers you've already visited.
2. For every element, check whether `target - nums[i]` already exists.
3. This reduces the time complexity to **O(n)**.

---

## Notes

- Exactly one solution exists.
- Cannot use the same index twice.
- Return indices, **not** values.
- HashMap is the optimal approach.



## Solution

## What is the question asking?

Explain the problem in simple English.

Example:

We need to find two numbers whose sum equals the target and return their indices.

---

## Input

| Parameter | Type | Description |
|-----------|------|-------------|
| nums | int[] | Input array |
| target | int | Required sum |

---

## Output

| Return Type | Description |
|-------------|-------------|
| int[] | Indices of the answer |

---

# Observations

- Only one answer exists.
- Cannot reuse same index.
- Order doesn't matter.

---

# Brute Force Approach

> **Idea:** Check every pair.

---

## Algorithm

1. Pick first element.
2. Compare with remaining elements.
3. If sum equals target return indices.

---

## Dry Run

Array

```text
[2,7,11,15]
```

Target

```text
9
```

```
i=0

2+7 = 9 ✅

Return [0,1]
```

---

## Java Code

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

---

## Complexity

| Complexity | Value |
|------------|-------|
| Time | O(n²) |
| Space | O(1) |

---

# Better Approach

Explain if one exists.

Example

Sorting

Binary Search

Prefix Sum

etc.

---

# Optimal Approach

## Intuition

Use a HashMap.

Store previously visited numbers.

For each number,

check whether

```
target-currentNumber
```

already exists.

---

## Algorithm

1. Create HashMap.
2. Iterate array.
3. Calculate complement.
4. If complement exists return answer.
5. Otherwise store current value.

---

## Dry Run

Input

```
nums=[2,7,11,15]

target=9
```

| Index | Number | Complement | Map | Answer |
|------:|--------|------------|-----|--------|
|0|2|7|{2=0}| |
|1|7|2|Found|[0,1]|

---

## Java Solution

```java
import java.util.*;

class Solution {

    public int[] twoSum(int[] nums, int target) {

        Map<Integer,Integer> map = new HashMap<>();

        for(int i=0;i<nums.length;i++){

            int complement = target - nums[i];

            if(map.containsKey(complement)){

                return new int[]{map.get(complement),i};

            }

            map.put(nums[i],i);

        }

        return new int[0];

    }

}
```

---

# Complexity Analysis

| Metric | Value |
|---------|-------|
| Time Complexity | O(n) |
| Space Complexity | O(n) |

---

# Why HashMap Works

HashMap lookup is approximately

```
O(1)
```

Therefore,

instead of checking every element,

we only perform one lookup per iteration.

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

# Edge Cases

✅ Minimum input

```text
[1,2]
```

---

✅ Duplicate numbers

```text
[3,3]
```

---

✅ Negative numbers

```text
[-3,4,3,90]
```

---

✅ Large values

```text
10^9
```

---

# Common Mistakes

❌ Returning numbers instead of indices

❌ Using same element twice

❌ Forgetting duplicate values

❌ Sorting and losing indices

---

# Pattern Recognition

| Pattern | Used |
|----------|------|
| HashMap | ✅ |
| Array | ✅ |
| Lookup | ✅ |

---

# Alternative Solutions

| Approach | Time | Space |
|----------|------|-------|
| Brute Force | O(n²) | O(1) |
| HashMap | O(n) | O(n) |

---

# Interview Follow-up Questions

1. Can you solve it without extra space?

2. What if the array is sorted?

3. What if there are multiple answers?

4. Can duplicates exist?

5. Why is HashMap O(1)?

6. What is the worst-case complexity of HashMap?

7. Can TreeMap be used?

8. What happens if integers overflow?

---

# Related Problems

- Two Sum II
- Three Sum
- Four Sum
- Two Sum Less Than K
- Target Sum

---

# Revision Notes

✔ Learn the HashMap pattern.

✔ Store value → index.

✔ Check complement first.

✔ Don't reuse same index.

✔ Time Complexity: O(n)

✔ Space Complexity: O(n)

---

# Key Takeaways

- Recognize the HashMap pattern.
- Always think about complements.
- Understand trade-offs between time and space.
- Be ready to explain why the optimal approach works.

---