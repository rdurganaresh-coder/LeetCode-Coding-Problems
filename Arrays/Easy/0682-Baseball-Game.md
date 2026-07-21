# 682. Baseball Game

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Stack** &nbsp;&nbsp; 🏷️ **Simulation**

---

## 📝 Problem Description

You are keeping the scores for a baseball game with strange rules. At the beginning of the game, you start with an empty record.

You are given a list of strings `operations`, where `operations[i]` is the $i^{th}$ operation you must apply to the record and is one of the following:

* **An integer `x`:** Record a new score of `x`.
* **`'+'`:** Record a new score that is the sum of the previous two scores.
* **`'D'`:** Record a new score that is the double of the previous score.
* **`'C'`:** Invalidate the previous score, removing it from the record.

Return *the sum of all the scores on the record after applying all the operations*.

The test cases are generated such that the answer and all intermediate calculations fit in a **32-bit integer** and that all operations are valid.

### Examples

**Example 1:**
* **Input:** `ops = ["5","2","C","D","+"]`
* **Output:** `30`
* **Explanation:**
  * `"5"` - Add 5 to the record, record is now `[5]`.
  * `"2"` - Add 2 to the record, record is now `[5, 2]`.
  * `"C"` - Invalidate and remove the previous score, record is now `[5]`.
  * `"D"` - Add $2 \times 5 = 10$ to the record, record is now `[5, 10]`.
  * `"+"` - Add $5 + 10 = 15 to the record, record is now `[5, 10, 15]`.
  * The total sum is $5 + 10 + 15 = 30$.

**Example 2:**
* **Input:** `ops = ["5","-2","4","C","D","9","+","+"]`
* **Output:** `27`
* **Explanation:**
  * `"5"` -> `[5]`
  * `"-2"` -> `[5, -2]`
  * `"4"` -> `[5, -2, 4]`
  * `"C"` -> `[5, -2]`
  * `"D"` -> `[5, -2, -4]`
  * `"9"` -> `[5, -2, -4, 9]`
  * `"+"` -> `[5, -2, -4, 9, 5]`
  * `"+"` -> `[5, -2, -4, 9, 5, 14]`
  * Total sum is $5 + (-2) + (-4) + 9 + 5 + 14 = 27$.

### Constraints
* $1 \le \text{operations.length} \le 1000$
* `operations[i]` is `"C"`, `"D"`, `"+"`, or a string representing an integer in the range $[-3 \times 10^4, 3 \times 10^4]$.
* For operation `"+"`, there will always be at least two previous scores on the record.
* For operations `"C"` and `"D"`, there will always be at least one previous score on the record.

---

## 💡 Core Standard DSA Patterns

### 1. Last-In, First-Out (LIFO) Stack Pattern
* **Concept:** Managing data states where modifications, calculations, and rollbacks strictly target the most recently added historical values.
* **Mechanism:** The operations require looking back at the top element (`'D'`), the top two elements (`'+'`), or dropping the newest value completely (`'C'`). A linear sequential stack tracks these sequential states gracefully.

### 2. Array-Backed Pointer Stack Simulation
* **Concept:** Avoiding dynamic node allocation by tracking element thresholds inside fixed-size local array allocations with a tracking index pointer.
* **Mechanism:** Since the constraints guarantee that the operations count will not exceed $1000$, a flat primitive array `int[]` paired with a tracking size scalar `top` functions as a fast, zero-allocation stack structure.

---

## 💻 Standard DSA Implementations

### 1. Object Stack Framework ($O(N)$ Time & $O(N)$ Space)
The standard and readable framework using the explicit Java Collection structures to separate conditional simulation states clearly.

```java
import java.util.Stack;

public class Solution {
    public int calPoints(String[] operations) {
        Stack<Integer> scoreStack = new Stack<>();

        // Process operations one by one
        for (String op : operations) {
            switch (op) {
                case "+":
                    // Peek the top two elements to calculate their sum
                    int topScore = scoreStack.pop();
                    int nextToTopScore = scoreStack.peek();
                    int newSumScore = topScore + nextToTopScore;
                    
                    // Push the values back in proper sequential order
                    scoreStack.push(topScore);
                    scoreStack.push(newSumScore);
                    break;
                    
                case "D":
                    // Double the most recent score
                    scoreStack.push(2 * scoreStack.peek());
                    break;
                    
                case "C":
                    // Invalidate the last entry entirely
                    scoreStack.pop();
                    break;
                    
                default:
                    // Parse raw numeric string value into an integer
                    scoreStack.push(Integer.parseInt(op));
                    break;
            }
        }

        // Sum all remaining values inside the stack collection
        int totalSum = 0;
        for (int score : scoreStack) {
            totalSum += score;
        }

        return totalSum;
    }
}
```

### 2. Primitive Array Pointer Optimization ($O(N)$ Time & $O(N)$ Space)
A high-performance optimization bypass. It avoids boxed object instantiation wrapper layers by using a pre-allocated fixed-size array to run operations directly.

```java
public class Solution {
    public int calPoints(String[] operations) {
        // Upper bound of elements is bounded by the operations array constraint limit
        int[] scores = new int[operations.length];
        int size = 0;

        for (String op : operations) {
            switch (op) {
                case "+":
                    // Sum the last two valid elements in the array
                    scores[size] = scores[size - 1] + scores[size - 2];
                    size++;
                    break;
                    
                case "D":
                    // Double the previous valid element
                    scores[size] = 2 * scores[size - 1];
                    size++;
                    break;
                    
                case "C":
                    // Decrease size pointer to logically drop the last element
                    size--;
                    break;
                    
                default:
                    // Append the raw numerical value
                    scores[size] = Integer.parseInt(op);
                    size++;
                    break;
            }
        }

        // Aggregate across valid indexes inside the primitive buffer array
        int totalSum = 0;
        for (int i = 0; i < size; i++) {
            totalSum += scores[i];
        }

        return totalSum;
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Object Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Object Stack** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | Generates multiple `Integer` wrappers | Moderate | Highly readable logic code; creates object overhead during runtime push sequences. |
| **2. Array Pointer** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | **Exactly 1 flat array** | Excellent | Optimal execution performance profile; relies on manual pointer indexing logic. |

---

### Detailed Evaluation Breakdown

#### 1. Optimization Through Fixed Allocations
* **Mitigating Boxed Wrappers:** While Java's native `Stack<Integer>` provides clean abstractions, it requires auto-boxing primitive `int` values into heap-allocated `Integer` objects. Under the hood, standard `Stack` structures also rely on a heavy vector layout.
* **Array Memory Layout Benefits:** The array pointer approach creates exactly one primitive matrix layer (`int[]`). This keeps data packed closely in memory, allowing hardware systems to fetch next elements into high-speed CPU caches smoothly without searching through random heap locations.

#### 2. Clean Simulation Branch Processing
Using a `switch` instruction handles fixed character values cleanly. Instead of compiling into repetitive matching chains, Java optimizes fixed `String` switches by mapping operations directly to rapid hash index tables. This matches numbers and operators instantly, ensuring the pipeline finishes cleanly within linear $\mathcal{O}(N)$ limits.

## 💡 Core Standard DSA Patterns

### 1. Functional Stream Pipeline State Aggregation
* **Concept:** Modeling historical simulation transitions sequentially into a single mutable container using functional parameters.
* **Mechanism:** Rather than maintaining a global collection variable in the class scope, Java 8 `Arrays.stream().collect()` handles the process using custom accumulation functions, isolating code tracking neatly within the streaming logic.

### 2. Deque-Backed Stream Reduction
* **Concept:** Combining functional sequence pipelines with a double-ended queue (`Deque`) to keep lookups quick and clean.
* **Mechanism:** Using a standard `Stack` with functional parameters can sometimes cause slow vector modifications. Swapping to a `LinkedList` or `ArrayDeque` inside a stream reduction step makes pushing, popping, and looking up elements lightning-fast.

---

## 💻 Standard DSA Implementations

### 1. Functional Sequential Stream Reduction ($O(N)$ Time & $O(N)$ Space)
A pure Java 8 Streams pipeline solution. It uses a terminal `collect` process to pipe operation string commands through a sequential state engine.

```java
import java.util.Arrays;
import java.util.LinkedList;

public class Solution {
    public int calPoints(String[] operations) {
        if (operations == null || operations.length == 0) {
            return 0;
        }

        // Process operations sequentially into a structured historical tracking list
        LinkedList<Integer> scoreHistory = Arrays.stream(operations)
            .collect(
                LinkedList::new,
                (history, op) -> {
                    switch (op) {
                        case "+":
                            int last = history.peekLast();
                            int secondLast = history.get(history.size() - 2);
                            history.addLast(last + secondLast);
                            break;
                        case "D":
                            history.addLast(2 * history.peekLast());
                            break;
                        case "C":
                            history.removeLast();
                            break;
                        default:
                            history.addLast(Integer.parseInt(op));
                            break;
                    }
                },
                LinkedList::addAll // Parallel combiner logic fallback
            );

        // Map containing entries to primitive ints and summarize
        return scoreHistory.stream().mapToInt(Integer::intValue).sum();
    }
}
```

### 2. Stream-Wrapped External Pointer Accumulator ($O(N)$ Time & $O(N)$ Space)
An implementation using Java 8 functional iterations to process values over a high-performance external state object canvas.

```java
import java.util.Arrays;

public class Solution {
    // Structural state container wrapper to hold scores smoothly inside lambda closures
    private static class GameState {
        int[] scores;
        int size = 0;

        GameState(int maxCapacity) {
            this.scores = new int[maxCapacity];
        }

        void apply(String op) {
            switch (op) {
                case "+":
                    scores[size] = scores[size - 1] + scores[size - 2];
                    size++;
                    break;
                case "D":
                    scores[size] = 2 * scores[size - 1];
                    size++;
                    break;
                case "C":
                    size--;
                    break;
                default:
                    scores[size] = Integer.parseInt(op);
                    size++;
                    break;
            }
        }
    }

    public int calPoints(String[] operations) {
        if (operations == null || operations.length == 0) {
            return 0;
        }

        GameState state = new GameState(operations.length);

        // Process elements using forEach sequence execution
        Arrays.stream(operations).forEach(state::apply);

        // Stream across processed valid indices to calculate aggregate sum values
        return Arrays.stream(state.scores, 0, state.size).sum();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Object Allocations | Hardware Cache Optimization | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Stream Reduction** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | Creates list nodes and integer boxes | Moderate | Pure, declarative codebase layout; introduces wrapper footprint overheads. |
| **2. Pointer Accumulator** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | **Exactly 2 object structures** | Excellent | Blazing fast array throughput operations; demands custom structural helper code wrappers. |

---

### Detailed Evaluation Breakdown

#### 1. Lambda State Scoping Invariants
* **Effect of Effectively Final Constraints:** Java 8 lambda definitions cannot mutate loose external local variables. Wrapping states within mutable containers like a `LinkedList` or a custom `GameState` class bypasses this limitation cleanly, ensuring thread-safe encapsulation.
* **Primitive IntStream Gains:** The second approach optimizes performance by switching from a boxed object stream to a primitive `Arrays.stream(int[], start, end)`. This completely skips the wrapper layer, processing calculations cleanly at the hardware level.

#### 2. Thread Safety and Reduction Operations
Java 8 streams are concise but built with specific constraints. Because each baseball score depends heavily on previous rounds, processing events concurrently across an unordered parallel configuration can miscalculate results. Running the operations sequentially ensures scores are processed in the correct historical order.