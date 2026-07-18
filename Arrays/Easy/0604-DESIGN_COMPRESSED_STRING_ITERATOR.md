# LeetCode 604: Design Compressed String Iterator

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Design** &nbsp;&nbsp; 🏷️ **String** &nbsp;&nbsp; 🏷️ **Design Patterns** &nbsp;&nbsp; 🏷️ **Iterator**

---

## 📝 Problem Description

Design and implement a data structure for a compressed string iterator. The given compressed string will be in the form of each letter followed by a positive integer representing the number of times that letter repeats in the original uncompressed string.

Implement the `StringIterator` class:
* `StringIterator(String compressedString)` Initializes the object with the compressed string.
* `char next()` Returns the next character if the original uncompressed string still has characters remaining, otherwise returns a space `' '`.
* `boolean hasNext()` Returns `true` if there is any character remaining in the uncompressed string, otherwise returns `false`.

### Examples

**Example 1:**
```java
StringIterator stringIterator = new StringIterator("L1e2t1c1o1d1e1");
stringIterator.next();    // return 'L' (Uncompressed form is "Leetcode")
stringIterator.next();    // return 'e'
stringIterator.next();    // return 'e'
stringIterator.next();    // return 't'
stringIterator.next();    // return 'c'
stringIterator.next();    // return 'o'
stringIterator.hasNext(); // return true
stringIterator.next();    // return 'd'
stringIterator.next();    // return 'e'
stringIterator.hasNext(); // return false
stringIterator.next();    // return ' ' (No characters left)
```

### Constraints
* $1 \le \text{compressedString.length} \le 1000$
* `compressedString` consists of lowercase and uppercase English letters followed by digits.
* The number of a single character repeats in the uncompressed string will be in the range $[1, 10^9]$.
* At most $5000$ calls total will be made to `next` and `hasNext`.

---

## 💡 Core Patterns Used

### 1. On-The-Fly On-Demand Decompression Pattern
* **Concept:** Avoiding an explicit complete string decompression up-front in the constructor (which would trigger fatal `OutOfMemoryError` failures given character repeat frequencies up to $10^9$).
* **Mechanism:** Maintains global structural pointer offsets (`index`) over the source compressed string along with active token state tracking fields (`currentChar` and `currentCount`). The string is parsed conditionally only when `currentCount` hits zero, matching the low-overhead, memory-stationary iterator pattern.

### 2. Multi-Digit Number Parsing Pattern
* **Concept:** Correctly interpreting consecutive integer numeric strings without dropping intermediate radix positions.
* **Mechanism:** When a fresh token boundary is crossed, the character is captured, and the pointer advances to read the continuous trailing digit characters. It accumulates the true integer count smoothly across values via:
  $$\text{count} = \text{count} \times 10 + (\text{compressedString.charAt(index)} - \text'0\text')$$

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Pointer State Iterator (Standard DSA Style)
The textbook optimal architecture. It initializes instantly in $O(1)$ constant time and extracts token parts on demand with absolute minimal auxiliary memory footprint constraints.

```java
public class StringIteratorClassic {
    private final String src;
    private int index;
    private char currentChar;
    private int currentCount;

    public StringIteratorClassic(String compressedString) {
        this.src = compressedString;
        this.index = 0;
        this.currentChar = ' ';
        this.currentCount = 0;
    }

    // O(1) Amortized Time Complexity
    public char next() {
        if (!hasNext()) {
            return ' ';
        }
        
        // If the current character inventory is exhausted, parse the next chunk
        if (currentCount == 0) {
            currentChar = src.charAt(index++);
            
            // Accumulate multi-digit numbers (e.g., "a123")
            while (index < src.length() && Character.isDigit(src.charAt(index))) {
                currentCount = currentCount * 10 + (src.charAt(index++) - '0');
            }
        }
        
        currentCount--;
        return currentChar;
    }

    // O(1) Amortized Time Complexity
    public boolean hasNext() {
        // Safe check: True if there are characters left in the active pool or more chunks in the string
        return currentCount > 0 || index < src.length();
    }
}
```

### 2. Java 8 Functional Queue Buffer Iterator
This variant pre-splits the text layout blocks up-front into custom token records using functional stream mappings, storing them within a clean FIFO list structure to abstract tracking loops away.

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class StringIteratorStreamQueue {
    private static class TokenNode {
        char character;
        int count;
        
        TokenNode(char character, int count) {
            this.character = character;
            this.count = count;
        }
    }

    private final Queue<TokenNode> tokenBuffer;

    public StringIteratorStreamQueue(String compressedString) {
        this.tokenBuffer = new LinkedList<>();
        if (compressedString == null || compressedString.isEmpty()) return;

        // Use standard regex matching to separate individual character-count tokens cleanly
        Pattern pattern = Pattern.compile("([A-Za-z])(\\d+)");
        Matcher matcher = pattern.matcher(compressedString);

        // Populate the data queue container with separate token nodes
        while (matcher.find()) {
            char ch = matcher.group(1).charAt(0);
            int count = Integer.parseInt(matcher.group(2));
            tokenBuffer.offer(new TokenNode(ch, count));
        }
    }

    // O(1) Constant evaluation time complexity
    public char next() {
        if (!hasNext()) {
            return ' ';
        }

        TokenNode headNode = tokenBuffer.peek();
        char outputChar = headNode.character;
        headNode.count--;

        // If the node count hits zero, evict it from the active tracking queue entirely
        if (headNode.count == 0) {
            tokenBuffer.poll();
        }

        return outputChar;
    }

    // O(1) Constant execution time complexity
    public boolean hasNext() {
        return !tokenBuffer.isEmpty();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Constructor Complexity | `next()` Complexity | `hasNext()` Complexity | Space Complexity | GC Memory Management Pressure | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Pointer State** | $O(1)$ | $O(1)$ Amortized | $O(1)$ | $O(1)$ | **Zero (Immune)** | Fastest option by far; utilizes zero intermediate heap objects. |
| **2. Queue Buffer** | $O(K)$ | $O(1)$ Constant | $O(1)$ Constant | $O(K)$ | Medium | Exceptional encapsulation clarity; constructs temporary node objects up-front. |

*Where **$K$** is the absolute length of the `compressedString` input parameter ($1 \le K \le 1000$).*

---

### Detailed Evaluation Breakdown

#### 1. Why Full Uncompression is a System Anti-Pattern
A common trap for this problem is decompressing the string into a single large buffer in the constructor (e.g., expanding `"a1000000000"` into an array containing one billion `'a'` elements). Since a single repeat count can reach $10^9$, this approach will instantly crash the application thread with an `OutOfMemoryError`. 

Both provided patterns eliminate this risk completely by decoupling the representation from the raw physical size, ensuring memory stays strictly bound by the *compressed text scale*.

#### 2. Amortized vs. Absolute Constant-Time Performance
* **Pointer State Optimization:** Implementation 1 is exceptionally lean. The constructor performs zero text traversal ($O(1)$ setup cost). Calls to `next()` run in $O(1)$ amortized time because the parsing `while` loop only triggers intermittently when an existing character pool is completely exhausted, processing the string in-place with zero object generation churn.
* **Queue Buffer Mechanics:** Implementation 2 pays an up-front linear parsing cost in the constructor ($O(K)$) to pre-populate a `Queue` buffer. However, this grants it an absolute, deterministic $O(1)$ constant runtime profile for every subsequent `next()` call, completely isolating lookup tasks from text parsing loops.

***

If you want to step up to the next core structural architecture query, or want me to output a full **JUnit automated verification test matrix script** to check custom character combinations, let me know!
