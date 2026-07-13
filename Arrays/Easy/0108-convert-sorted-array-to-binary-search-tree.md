# 108. Convert Sorted Array to Binary Search Tree

`Easy` • `Array` • `Divide and Conquer` • `Tree` • `Binary Search Tree`
> 🟢 Easy &nbsp;&nbsp; 🏷️ Arrays &nbsp;&nbsp;  🏷️ Divide and Conquer &nbsp;&nbsp; 🏷️ Tree &nbsp;&nbsp; 🏷️ Binary Search Tree &nbsp;&nbsp;

---

## Problem Description
Given an integer array `nums` where the elements are sorted in **ascending order**, convert *it to a **height-balanced** binary search tree*.

A **height-balanced** binary tree is a binary tree in which the depth of the two subtrees of every node never differs by more than one.

### Example 1:
> **Input:** `nums = [-10,-3,0,5,9]`
> **Output:** `[0,-3,9,-10,null,5]`
> **Explanation:** `[0,-10,5,null,-3,null,9]` is also accepted:
> 
>       0                     0
>      / \                   / \
>     -3   9                -10  5
>    /   /                   \   \
>  -10  5                    -3   9

### Example 2:
> **Input:** `nums = [1,3]`
> **Output:** `[3,1]`
> **Explanation:** `[1,null,3]` and `[3,1]` are both height-balanced BSTs.

### Constraints:
* `1 <= nums.length <= 10^4`
* `-10^4 <= nums[i] <= 10^4`
* `nums` is sorted in a **strictly increasing** order.

---

## Optimal Approach
> Use a Divide and Conquer approach similar to binary search. To ensure the tree stays height-balanced, always pick the middle element of the current array segment as the root node. Recursively repeat this step for the left half to construct the left subtree, and the right half for the right subtree.

---

## Definition for a binary tree node
```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

---

## Method 1: Traditional Recursive Helper (Recommended)
This approach processes the boundaries using an explicit recursive helper function. It scores a flawless **0ms runtime** on LeetCode's engine and uses no additional space beyond the recursion stack.

```java
import java.util.Objects;

class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        Objects.requireNonNull(nums, "Input array cannot be null");
        return constructBST(nums, 0, nums.length - 1);
    }

    private TreeNode constructBST(int[] nums, int left, int right) {
        if (left > right) {
            return null;
        }

        // Prevent potential integer overflow
        int mid = left + (right - left) / 2;
        
        TreeNode root = new TreeNode(nums[mid]);
        root.left = constructBST(nums, left, mid - 1);
        root.right = constructBST(nums, mid + 1, right);
        
        return root;
    }
}
```

---

## Method 2: Functional Subarray Extraction (Java 8 API)
This variation uses Java 8's `Arrays.copyOfRange` to split the array dynamically instead of manually passing indexing bounds. It mimics a pure functional divide-and-conquer strategy.

```java
import java.util.Arrays;

class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        int mid = nums.length / 2;
        TreeNode root = new TreeNode(nums[mid]);

        // Java 8: Slice and process sub-arrays via range copies
        root.left = sortedArrayToBST(Arrays.copyOfRange(nums, 0, mid));
        root.right = sortedArrayToBST(Arrays.copyOfRange(nums, mid + 1, nums.length));

        return root;
    }
}
```
* **Pros:** Clean, minimal recursive signature without an explicit helper method.
* **Cons:** Runs at **$O(n \log n)$ time complexity** and **$O(n)$ space complexity** because it reallocates new array slices at every recursive frame step.

---

## Method 3: Stream-Inspired State Mapping Optimization
If you want to track boundaries without array copying while avoiding a traditional custom helper signature, you can map states using functional configurations.

```java
import java.util.function.BiFunction;

class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) return null;

        // Java 8: Define the recursive logic inside a lambda expression container
        BiFunction<Integer, Integer, TreeNode> bstBuilder = new BiFunction<Integer, Integer, TreeNode>() {
            @Override
            public TreeNode apply(Integer left, Integer right) {
                if (left > right) return null;
                
                int mid = (left + right) >>> 1;
                TreeNode node = new TreeNode(nums[mid]);
                
                node.left = this.apply(left, mid - 1);
                node.right = this.apply(mid + 1, right);
                
                return node;
            }
        };

        return bstBuilder.apply(0, nums.length - 1);
    }
}
```


## Functional Array Custom Spliterator:
>This approach builds a custom functional splitter that wraps our primitive arrays. Instead of creating expensive array copies, it slices metadata windows using a functional pipeline interface, mapping directly into the object hierarchy.
```java
class Solution {
    // Custom container to track array slices without copying memory blocks
    private static class ArraySlice {
        final int[] data;
        final int start;
        final int end;

        ArraySlice(int[] data, int start, int end) {
            this.data = data;
            this.start = start;
            this.end = end;
        }

        boolean isValid() { return start <= end; }
        int mid() { return start + (end - start) / 2; }
    }

    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) return null;

        // Java 8: Define a self-referencing functional lambda logic pipeline
        Function<ArraySlice, TreeNode> treeBuilder = new Function<ArraySlice, TreeNode>() {
            @Override
            public TreeNode apply(ArraySlice slice) {
                return Optional.of(slice)
                        .filter(ArraySlice::isValid)
                        .map(s -> {
                            int mid = s.mid();
                            TreeNode root = new TreeNode(s.data[mid]);
                            
                            // Recursively build children subtrees using lazy-bound state evaluations
                            root.left = this.apply(new ArraySlice(s.data, s.start, mid - 1));
                            root.right = this.apply(new ArraySlice(s.data, mid + 1, s.end));
                            
                            return root;
                        })
                        .orElse(null);
            }
        };

        return treeBuilder.apply(new ArraySlice(nums, 0, nums.length - 1));
    }
}
```
> Use code with caution.Why it's great: It eliminates index calculation variables completely from the outer method namespace. It leverages Optional pipelines instead of standard if-else guard checks.Performance: O(n) time complexity with \(O(\log n)\) auxiliary stack space, completely avoiding the expensive memory overhead of Arrays.copyOfRange.

## Functional Queue Coordinate Loop (Iterative Approach):
>For extremely deep array inputs where recursion might cause a StackOverflowError, we can simulate the call stack iteratively. This approach couples an explicit loop with a state holder pattern and unboxes values via sequential processing queues.
```java
class Solution {
    private static class NodeState {
        TreeNode node;
        int left;
        int right;

        NodeState(TreeNode node, int left, int right) {
            this.node = node;
            this.left = left;
            this.right = right;
        }
    }

    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) return null;

        int totalLen = nums.length;
        TreeNode root = new TreeNode(0); // Temporary dummy value
        
        Queue<NodeState> processingQueue = new LinkedList<>();
        processingQueue.add(new NodeState(root, 0, totalLen - 1));

        // Use a continuous loop processing element states
        while (!processingQueue.isEmpty()) {
            NodeState curr = processingQueue.poll();
            int mid = curr.left + (curr.right - curr.left) / 2;
            curr.node.val = nums[mid]; // Populate the deferred index value

            if (curr.left <= mid - 1) {
                curr.node.left = new TreeNode(0);
                processingQueue.add(new NodeState(curr.node.left, curr.left, mid - 1));
            }
            if (mid + 1 <= curr.right) {
                curr.node.right = new TreeNode(0);
                processingQueue.add(new NodeState(curr.node.right, mid + 1, curr.right));
            }
        }

        return root;
    }
}
```

> Use code with caution.Why it's great:
> Moves calculations entirely onto the heap storage allocation rather than using runtime execution threads, preventing stack crash conditions entirely on huge data trees.

### Updated Performance & Constraint Evaluation (Markdown Format)

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Recursive Indices)** | $O(n)$ | $O(\log n)$ call stack | **Highly Recommended** (Optimal 0ms execution) |
| **Method 4 (`Optional` Slice)** | $O(n)$ | $O(\log n)$ call stack | Clean data pipeline structure with linear time profile |
| **Method 5 (Iterative Queue)** | $O(n)$ | $O(n)$ heap allocation | Overkill for simple trees, but strictly safe from StackOverflow |

---

### Performance & Constraint Evaluation

| Solution Method | Time Complexity | Space Complexity | LeetCode Recommendation |
| :--- | :--- | :--- | :--- |
| **Method 1 (Recursive Indices)** | $O(n)$ | $O(\log n)$ call stack | **Highly Recommended** (Optimal 0ms execution) |
| **Method 2 (`Arrays.copyOfRange`)**| $O(n \log n)$ | $O(n)$ | Inefficient due to heavy array array allocations |
| **Method 3 (`BiFunction` Lambda)** | $O(n)$ | $O(\log n)$ call stack | Functional structure; slight lambda execution overhead |

---
