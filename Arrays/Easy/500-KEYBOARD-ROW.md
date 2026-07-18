# LeetCode 500: Keyboard Row

> 🟢 **Easy** &nbsp;&nbsp; 🏷️ **Arrays** &nbsp;&nbsp; 🏷️ **Hash Table** &nbsp;&nbsp; 🏷️ **String**

---

## 📝 Problem Description

Given an array of strings `words`, return *the words that can be typed using letters of the alphabet on only **one row** of the American keyboard*.

Note that the strings are case-insensitive; both lowercased and uppercased versions of the same letter are treated as if they are in the same row.

In the American keyboard:
* The first row consists of the characters `"qwertyuiop"`
* The second row consists of the characters `"asdfghjkl"`
* The third row consists of the characters `"zxcvbnm"`

### Examples

**Example 1:**
* **Input:** `words = ["Hello","Alaska","Dad","Peace"]`
* **Output:** `["Alaska","Dad"]`
* **Explanation:** Both "Alaska" and "Dad" can be spelled completely using only the second row of characters.

**Example 2:**
* **Input:** `words = ["omk"]`
* **Output:** `[]`

**Example 3:**
* **Input:** `words = ["adsdf","sfd"]`
* **Output:** `["adsdf","sfd"]`

### Constraints
* $1 \le \text{words.length} \le 20$
* $1 \le \text{words}[i].\text{length} \le 100$
* `words[i]` consists of English letters (both lowercase and uppercase).

---

## 💡 Core Patterns Used

### 1. Direct-Addressing Array Cache Pattern (Primitive Map Lookup)
* **Concept:** Mapping character sets to integer row identities using a small, bounded primitive array instead of a heavy object hash table.
* **Mechanism:** Since characters are restricted to uppercase and lowercase English letters, we can create a fixed-size `int[26]` or `int[128]` tracking cache. Each alphabet index directly stores its respective row ID (1, 2, or 3). Checking a word reduces to extracting the row ID of the first character and ensuring every subsequent character shares the identical row value.

### 2. Java 8 Stream Short-Circuit Predicate Pattern
* **Concept:** Filtering a string stream via functional pipeline matching boundaries.
* **Mechanism:** Filters the input array using regex patterns (`.matches()`) combined with clean functional collections to compile matching entries seamlessly into a final array layout.

---

## 💻 Java & Java 8 Implementations

### 1. High-Performance Direct Array Cache (Standard DSA Style)
The definitive production benchmark standard. It avoids heap allocations entirely during processing by validating row IDs via single-cycle primitive lookup hits.

```java
import java.util.ArrayList;
import java.util.List;

public class KeyboardRowClassic {
    // Shared primitive lookup mapping alphabet characters directly to their Keyboard Row IDs
    private static final int[] ROW_MAP = new int[26];

    static {
        // Map Row 1: qwertyuiop
        for (char c : "qwertyuiop".toCharArray()) ROW_MAP[c - 'a'] = 1;
        // Map Row 2: asdfghjkl
        for (char c : "asdfghjkl".toCharArray())  ROW_MAP[c - 'a'] = 2;
        // Map Row 3: zxcvbnm
        for (char c : "zxcvbnm".toCharArray())    ROW_MAP[c - 'a'] = 3;
    }

    public static String[] findWords(String[] words) {
        if (words == null || words.length == 0) return new String[0];

        List<String> validWords = new ArrayList<>();

        for (String word : words) {
            if (isValidSingleRow(word)) {
                validWords.add(word);
            }
        }

        return validWords.toArray(new String[0]);
    }

    private static boolean isValidSingleRow(String word) {
        // Extract base target row matching character index 0 (normalized to lowercase)
        int targetRow = getRowId(word.charAt(0));

        for (int i = 1; i < word.length(); i++) {
            if (getRowId(word.charAt(i)) != targetRow) {
                return false; // Mismatch found: Word spans multiple rows
            }
        }
        return true;
    }

    private static int getRowId(char c) {
        // Handle case-insensitivity cleanly via bitwise masking or subtraction
        if (c >= 'A' && c <= 'Z') {
            return ROW_MAP[c - 'A'];
        }
        return ROW_MAP[c - 'a'];
    }
}
```

### 2. Java 8 Stream Regex Filtering (Declarative Style)
An exceptionally expressive solution that abstracts loop states completely by evaluating characters via short-circuiting regex matching pipelines.

```java
import java.util.Arrays;

public class KeyboardRowRegexStream {
    public static String[] findWords(String[] words) {
        if (words == null || words.length == 0) return new String[0];

        // Regex explanation: Match strings containing strictly row-isolated characters case-insensitively
        String row1Regex = "(?i)[qwertyuiop]+";
        String row2Regex = "(?i)[asdfghjkl]+";
        String row3Regex = "(?i)[zxcvbnm]+";

        return Arrays.stream(words)
                .filter(word -> word.matches(row1Regex) || 
                                word.matches(row2Regex) || 
                                word.matches(row3Regex))
                .toArray(String[]::new);
    }
}
```

---

## 📊 Performance & Constraint Evaluation

### Complexity & Constraint Matrix

| Implementation | Time Complexity | Space Complexity (Auxiliary) | Heap Allocations | Early Exit Capability? | Primary Engineering Trade-off |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Direct Array Cache** | $O(N \times L)$ | $O(1)$ (Capped bounds) | **Zero (Immune)** | **Yes (Instant)** | Maximum execution speed; requires minor manual character indexing logic. |
| **2. Regex Stream** | $O(N \times L)$ | $O(1)$ | High | No (Full match evaluation) | Outstanding declarative brevity; regex state machine recompilation adds runtime tax. |

*Where **$N$** is the number of words in the array and **$L$** is the maximum length of a word string.*

---

### Detailed Evaluation Breakdown

#### 1. Optimal Direct Array Cache Strategy
* **Zero GC Overhead:** Completely avoiding stream allocations or string copies during the matching loops ensures zero objects are allocated on the heap. This prevents Garbage Collection (GC) pauses entirely, creating an ideal architecture for embedded processing pipelines.
* **High-Speed Short-Circuiting:** The helper method returns `false` the exact millisecond a mismatching row character is evaluated. For a word of length 100, if character index 1 breaks the row boundary, we bypass the remaining 98 lookups entirely, saving valuable CPU cycles.

#### 2. Java 8 Stream Regex Optimization
* **Strengths:** Exceptional visual readability. The code layout clearly logs the exact functional requirements (Filter stream elements matching target keyboard rows $\rightarrow$ pack into fresh output array).
* **Performance Impact:** Regex compilation costs. Evaluating string patterns via `.matches()` compiles a temporary internal state machine engine behind the scenes on every invocation. For small constraint bounds ($N \le 20$), this is completely negligible, but it can become a bottleneck when dealing with millions of log messages.
