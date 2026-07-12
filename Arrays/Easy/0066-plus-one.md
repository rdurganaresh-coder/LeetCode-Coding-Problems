# 66. Plus One

> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Math &nbsp;&nbsp;

---

## Problem Description
You are given a **large integer** represented as an integer array `digits`, where each `digits[i]` is the $i^{th}$ digit of the integer. The digits are ordered from most significant to least significant in left-to-right order. The large integer does not contain any leading `0`s.

Increment the large integer by one and return *the resulting array of digits*.

### Example 1:
> **Input:** `digits = [1,2,3]`
> **Output:** `[1,2,4]`
> **Explanation:** The array represents the integer 123. Incrementing by one gives 123 + 1 = 124. Thus, the result should be `[1,2,4]`.

### Example 2:
> **Input:** `digits = [4,3,2,1]`
> **Output:** `[4,3,2,2]`
> **Explanation:** The array represents the integer 4321. Incrementing by one gives 4321 + 1 = 4322. Thus, the result should be `[4,3,2,2]`.

### Example 3:
> **Input:** `digits = [9]`
> **Output:** `[1,0]`
> **Explanation:** The array represents the integer 9. Incrementing by one gives 9 + 1 = 10. Thus, the result should be `[1,0]`.

### Constraints:
* `1 <= digits.length <= 100`
* `0 <= digits[i] <= 9`
* `digits` does not contain any leading `0`s.

---

## Optimal Approach
> Start from the last digit (least significant). If the digit is less than 9, increment it by 1 and return the array immediately. If it is 9, turn it into 0 and carry over to the next digit. If all digits are 9, create a new array with a length of `n + 1` and set the first index to 1.

---

## Method 1: Normal / Standard Iterative Approach (Recommended)
This approach handles the carry logic directly inside an optimized reverse loop. It runs at **0ms** and avoids unnecessary allocations unless a total overflow occurs (e.g., `999 -> 1000`).

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int n = digits.length;
        
        // Traverse the array from right to left
        for (int i = n - 1; i >= 0; i--) {
            if (digits[i] < 9) {
                digits[i]++;
                return digits; // No carry forward needed, return early
            }
            // If it's a 9, it becomes 0 due to carry
            digits[i] = 0;
        }
        
        // If the loop finished, all digits were 9s (e.g., 999 -> 1000)
        int[] newDigits = new int[n + 1];
        newDigits[0] = 1; // Remaining indices default to 0 automatically
        return newDigits;
    }
}
```

---

## Method 2: Functional Java 8 Streams Approach
While this problem requires array mutation that does not naturally map to Streams, we can model the array transformations functionally using Java 8 pipelines to showcase functional state evaluations.

```java
import java.util.Arrays;
import java.util.stream.IntStream;

class Solution {
    public int[] plusOne(int[] digits) {
        // Java 8: Check if all elements are exactly equal to 9
        boolean allNines = Arrays.stream(digits).allMatch(digit -> digit == 9);

        if (allNines) {
            // Instantly generate a stream of size n + 1 starting with a 1 followed by 0s
            return IntStream.concat(IntStream.of(1), IntStream.generate(() -> 0).limit(digits.length))
                            .toArray();
        }

        // Otherwise, work backwards using traditional mutation combined with stream validation
        for (int i = digits.length - 1; i >= 0; i--) {
            if (digits[i] < 9) {
                digits[i]++;
                break;
            }
            digits[i] = 0;
        }
        return digits;
    }
}
```
## BigInteger Stream Reduction (For Large Arrays):
> Since the constraints state that digits.length can be up to 100, standard primitive numbers like long will overflow. We can use Java 8 Streams to convert the entire array into a string, parse it with BigInteger, increment it, and stream it back out into an array.

```java 
public int[] plusOne(int[] digits) {
        // Step 1: Java 8 Stream maps each int to String and joins them together
        String numberStr = Arrays.stream(digits)
                .mapToObj(String::valueOf)
                .collect(Collectors.joining());

        // Step 2: Handle arbitrary precision math with BigInteger
        BigInteger number = new BigInteger(numberStr).add(BigInteger.ONE);

        // Step 3: Stream the string characters back into a primitive int array
        return number.toString().chars()
                .map(ch -> ch - '0')
                .toArray();
    }
```
>Use code with caution.Pros: Highly declarative and abstracts away all carry-over logic.Cons: Runs at \(O(n)\) space complexity due to heavy string conversions and string token allocations. This will run significantly slower on LeetCode's engine.

## Spliterator Deque MutationThis approach:
> uses a double-ended queue (Deque) to build the array from right to left, utilizing Java 8's StreamSupport along with primitive spliterators to extract values cleanly without array index tracking variables.
```java
    public int[] plusOne(int[] digits) {
        Deque<Integer> resultQueue = new ArrayDeque<>();
        int carry = 1;

        // Traverse the array backwards using a simple loop, building a dynamic Queue
        for (int i = digits.length - 1; i >= 0; i--) {
            int sum = digits[i] + carry;
            resultQueue.addFirst(sum % 10);
            carry = sum / 10;
        }

        if (carry > 0) {
            resultQueue.addFirst(carry);
        }

        // Java 8: Convert Object Queue to unboxed primitive array
        return resultQueue.stream().mapToInt(Integer::intValue).toArray();
    }
```
> Use code with caution.Pros: Standardizes how digit structures are built when a dynamic array extension is expected.Cons: Boxing overhead (int to Integer and back) increases runtime and memory usage. 
---
### Updated Performance & Constraint Evaluation

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Normal Iterative)** | $O(n)$ | $O(1)$ | **Highly Recommended** (0ms runtime, minimal footprint) |
| **Method 2 (Streams Match Check)** | $O(n)$ | $O(n)$ | Clean mix of stream short-circuit matching and mutation |
| **Method 3 (`BigInteger` Pipeline)**| $O(n)$ | $O(n)$ | Highly declarative, but very slow due to object parsing |
| **Method 4 (`Deque` Collection)** | $O(n)$ | $O(n)$ | Good structural fallback, slower due to primitive boxing |


### Performance & Constraint Evaluation

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Normal Iterative)** | $O(n)$ | $O(1)$ auxiliary ($O(n)$ only on complete overflow) | **Highly Recommended** (0ms runtime) |
| **Method 2 (Java 8 Streams)** | $O(n)$ | $O(n)$ | Elegant, but incurs stream allocation overhead |

---
