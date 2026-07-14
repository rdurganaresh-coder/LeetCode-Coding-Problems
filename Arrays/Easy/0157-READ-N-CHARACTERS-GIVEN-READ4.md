# LeetCode 157: Read N Characters Given Read4

> 🟢 Easy &nbsp;&nbsp; 🏷️Arrays &nbsp;&nbsp; 🏷️ Simulation

---

## 📝 Problem Description

Given a file and an API `read4`, divide the file read stream to fetch up to `n` characters.

The API `int read4(char[] buf4)` reads 4 consecutive characters from a file and writes those characters into the buffer array `buf4`. The return value is the actual number of characters read.

Note that `read4()` has its own file pointer, much like `FILE *fp` in C/C++.

### API Specification
* **Definition:** `int read4(char[] buf4);`
* **Input:** `char[] buf4` (an allocated destination buffer array of size 4)
* **Output:** Returns the count of characters read ($0 \le \text{count} \le 4$). It populates `buf4`.

### Method to Implement
You need to implement the method `int read(char[] buf, int n)` which reads `n` total characters from the file and stores them into the destination buffer array `buf`. You are **not** allowed to call the file system directly; you must interact with it solely through the `read4` API.

### Examples

**Example 1:**
* **Input:** `file = "abc", n = 4`
* **Output:** `3`
* **Explanation:** The file has 3 characters total: 'a', 'b', and 'c'. Calling `read4` reads all 3 characters into an internal staging buffer and returns 3. Since 3 is less than the requested `n` (4), the simulation stops and returns 3.

**Example 2:**
* **Input:** `file = "abcde", n = 3`
* **Output:** `3`
* **Explanation:** The file has 5 characters: "abcde". The first call to `read4` returns 4 characters ("abcd"). However, since `n = 3`, we only copy 3 characters ('a', 'b', 'c') into our final buffer `buf` and stop.

**Example 3:**
* **Input:** `file = "abcdabcdefg", n = 8`
* **Output:** `8`
* **Explanation:** The first call to `read4` returns 4 characters ("abcd"). The second call to `read4` returns another 4 characters ("abcd"). We have copied a total of 8 characters, satisfying `n`.

### Constraints
* $1 \le \text{file.length} \le 500$
* `file` consists of English letters and digits.
* `buf` destination size is guaranteed to be large enough to hold `n` characters.

---

## 💡 Core Patterns Used

### 1. Fixed-Window Buffer Simulation
* **Concept:** Interfacing with fixed-size structural hardware or file APIs using a local static array window cache.
* **Mechanism:** Maintains a standard chunking sequence where we continuously fetch a fixed number of bytes (4 bytes at a time) and selectively drain elements into the final dynamic consumer buffer `buf`.

### 2. Double-Bounded Convergence
* **Concept:** Processing streams under two strict termination thresholds simultaneously.
* **Mechanism:** The loop exits as soon as *either* of the following conditions is met:
  1. The total copied characters reach the upper limit `n`.
  2. The `read4` API returns less than 4 characters, signaling that the End Of File (EOF) has been reached.

---

## 💻 Java & Java 8 Implementations

*Note: For the code compilation to look valid, assume the class extends a base class containing the signature `int read4(char[] buf4);`.*

### 1. Classic Pointer Simulation (Optimal $O(N)$ Time & $O(1)$ Space)
This standard procedural format uses a single loop to manage stream buffers with zero overhead. It reads data into a temporary 4-character array and transfers it to the final array element by element.

```java
public class SolutionRead4Classic extends ReaderBase {
    /**
     * @param buf Destination buffer
     * @param n   Number of characters to read
     * @return    The number of actual characters read
     */
    public int read(char[] buf, int n) {
        char[] buf4 = new char[4];
        int totalCharsCopied = 0;
        
        while (totalCharsCopied < n) {
            // Read next chunk from API
            int charsReadFromAPI = read4(buf4);
            
            // If EOF is reached on an empty read boundary, break early
            if (charsReadFromAPI == 0) {
                break;
            }
            
            // Copy data to main buffer element-by-element
            for (int i = 0; i < charsReadFromAPI && totalCharsCopied < n; i++) {
                buf[totalCharsCopied++] = buf4[i];
            }
            
            // If the API read less than 4 characters, we hit the end of the file
            if (charsReadFromAPI < 4) {
                break;
            }
        }
        
        return totalCharsCopied;
    }
}
```

### 2. Math-Bounded Array Copy ($O(N)$ Time Optimization)
This approach replaces individual loop assignments with optimized bulk memory methods, calculating precise segment bounds using `Math.min`.

```java
public class SolutionRead4BulkCopy extends ReaderBase {
    public int read(char[] buf, int n) {
        char[] buf4 = new char[4];
        int totalCharsCopied = 0;
        boolean isEndOfFile = false;
        
        while (totalCharsCopied < n && !isEndOfFile) {
            int charsReadFromAPI = read4(buf4);
            
            if (charsReadFromAPI < 4) {
                isEndOfFile = true;
            }
            
            // Determine the maximum safe length to copy without exceeding 'n'
            int uniqueChunkSize = Math.min(charsReadFromAPI, n - totalCharsCopied);
            
            // System.arraycopy compiles to highly optimized native assembly instructions
            System.arraycopy(buf4, 0, buf, totalCharsCopied, uniqueChunkSize);
            totalCharsCopied += uniqueChunkSize;
        }
        
        return totalCharsCopied;
    }
}
```

### 3. Java 8 Functional Supplier Stream (Stateful Stream Simulation)
Since streaming requires clean abstractions, this pattern encapsulates the file reader state within a functional framework using a custom data mapper pipeline.

```java
import java.util.Arrays;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Stream;

public class SolutionRead4Stream extends ReaderBase {
    private static class ChunkWrapper {
        char[] data = new char[4];
        int length = 0;
    }

    public int read(char[] buf, int n) {
        AtomicInteger totalCopied = new AtomicInteger(0);
        ChunkWrapper wrapper = new ChunkWrapper();

        // Generate a stateful functional supplier sequence stream
        Stream.generate(() -> {
            wrapper.length = read4(wrapper.data);
            return wrapper;
        })
        .takeWhile(chunk -> chunk.length > 0 && totalCopied.get() < n)
        .forEach(chunk -> {
            int limit = Math.min(chunk.length, n - totalCopied.get());
            for (int i = 0; i < limit; i++) {
                buf[totalCopied.get() + i] = chunk.data[i];
            }
            totalCopied.addAndGet(limit);
        });

        return totalCopied.get();
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Thread Safety | Primitive Safety Array Profiles | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Classic Simulation**| $O(N)$ | $O(1)$ | Safe if unshared | Yes (Optimal) | Highly readable iteration block; manual character shifting |
| **2. Math Bulk Copy** | $O(N)$ | $O(1)$ | Safe if unshared | Yes (Optimal) | Maximum speed using memory-aligned system loops |
| **3. Functional Stream** | $O(N)$ | $O(1)$ | Unsafe | No (`Atomic` Wrapper) | High functional expression; object generation churn |

---

### Detailed Evaluation Breakdown

#### 1. Classic Pointer Simulation
* **Strengths:** Utterly basic memory allocation layout. The 4-character array `buf4` is allocated just once outside the loop on the stack frame, guaranteeing deterministic performance profiles.
* **Weaknesses:** Modifying values using element-by-element loops introduces branch instructions inside deep call graphs.

#### 2. Math Bulk Copy
* **Strengths:** Maximum performance index. `System.arraycopy` circumvents standard Java Virtual Machine bytecode indexing checks by utilizing direct native memory blocks. This is ideal for handling larger streams or network payloads.
* **Constraints:** Relies on structural pointer subtraction code logic patterns, which requires careful index bound checking to prevent off-by-one errors.

#### 3. Java 8 Functional Supplier Stream
* **Strengths:** Modern declarative syntax profile. It cleanly wraps data termination boundaries using functional stream pipelines like Java 9's `.takeWhile()`.
* **Weaknesses:** Memory allocation overhead. Initializing atomic state wrappers (`AtomicInteger`) introduces minor locking costs and object generation churn inside the local heap arena.
