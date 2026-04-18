# Java — Algorithms Interview Prep

## Table of Contents
- [1. Searching Algorithms](#1-searching-algorithms)
- [2. Sorting Algorithms](#2-sorting-algorithms)
- [3. Recursion & Divide and Conquer](#3-recursion--divide-and-conquer)
- [4. Two Pointers & Sliding Window](#4-two-pointers--sliding-window)
- [5. Hashing Patterns](#5-hashing-patterns)
- [6. Tree & Graph Traversal](#6-tree--graph-traversal)
- [7. Dynamic Programming Intro](#7-dynamic-programming-intro)

---

## 1. Searching Algorithms

**Q: What is linear search and when is it appropriate?**

Linear search iterates through every element until it finds the target. It works on any collection — sorted or unsorted.

```java
int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;   // found — return index
    }
    return -1;  // not found
}
```

| | Value |
|---|---|
| Time complexity | O(n) |
| Space complexity | O(1) |
| Requires sorted input | No |

**Use when:** the array is small, unsorted, or you're searching only once (sorting first + binary search only pays off for repeated searches).

---

**Q: What is binary search and how does it work?**

Binary search works on a **sorted** array. It repeatedly halves the search space by comparing the target to the middle element.

```
Sorted array: [2, 5, 8, 12, 16, 23, 38, 42, 56, 72]
Target: 23

Step 1: mid = index 4 → value 16. 23 > 16 → search right half
Step 2: mid = index 7 → value 42. 23 < 42 → search left half
Step 3: mid = index 5 → value 23. Found!
3 steps instead of 9 for linear search.
```

```java
int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;   // avoids integer overflow vs (left+right)/2

        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;   // target is in right half
        else right = mid - 1;                          // target is in left half
    }
    return -1;  // not found
}
```

| | Value |
|---|---|
| Time complexity | O(log n) |
| Space complexity | O(1) iterative / O(log n) recursive (call stack) |
| Requires sorted input | Yes |

**Why `left + (right - left) / 2` instead of `(left + right) / 2`?**  
If `left` and `right` are both large integers, `left + right` can overflow `int`. Subtracting first keeps the arithmetic safe.

Java provides `Arrays.binarySearch(arr, target)` — returns the index if found, or a negative value if not.

---

**Q: What are common variations of binary search?**

Binary search isn't just "find exact value" — it's a template for any problem where you can eliminate half the search space at each step.

**Find first occurrence of a value (duplicates in array):**
```java
int firstOccurrence(int[] arr, int target) {
    int left = 0, right = arr.length - 1, result = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) {
            result = mid;        // record, but keep searching left
            right = mid - 1;
        } else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return result;
}
```

**Find the smallest element greater than target (upper bound):**
```java
int upperBound(int[] arr, int target) {
    int left = 0, right = arr.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) left = mid + 1;
        else right = mid;
    }
    return left;  // index of first element > target
}
```

**Binary search on answer:** Some problems ask "find the minimum X such that condition(X) is true." If condition is monotonic (once true, stays true), binary search on the answer space directly.

---

## 2. Sorting Algorithms

**Q: How does Bubble Sort work? When would you use it?**

Bubble Sort repeatedly steps through the array, comparing adjacent elements and swapping if they're out of order. Larger elements "bubble up" to the end.

```java
void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;   // already sorted — early exit
    }
}
```

| | Value |
|---|---|
| Time (best) | O(n) — already sorted, with early exit |
| Time (average/worst) | O(n²) |
| Space | O(1) |
| Stable | Yes |

**Honest assessment:** Never use Bubble Sort in production. It exists primarily to teach sorting concepts. The early-exit optimization makes it O(n) for nearly-sorted data, which is its only redeeming use case — but Insertion Sort is better even there.

---

**Q: How does Insertion Sort work?**

Builds a sorted portion of the array one element at a time — pick the next element and insert it into its correct position in the already-sorted left portion.

```java
void insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];   // shift right to make room
            j--;
        }
        arr[j + 1] = key;          // insert in correct position
    }
}
```

| | Value |
|---|---|
| Time (best) | O(n) — nearly sorted |
| Time (average/worst) | O(n²) |
| Space | O(1) |
| Stable | Yes |

**Actually useful for:** small arrays (< ~20 elements) and nearly-sorted data. This is why Timsort (Java's built-in sort) uses Insertion Sort for small subarrays — it's faster than Merge Sort for tiny n due to low overhead and cache efficiency.

---

**Q: How does Merge Sort work?**

Merge Sort is a **divide and conquer** algorithm. It recursively splits the array in half, sorts each half, then merges the two sorted halves.

```java
void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;                     // base case: single element

    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);                     // sort left half
    mergeSort(arr, mid + 1, right);                // sort right half
    merge(arr, left, mid, right);                  // merge both halves
}

void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;

    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];

    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

| | Value |
|---|---|
| Time (all cases) | O(n log n) — guaranteed |
| Space | O(n) — needs auxiliary array |
| Stable | Yes |

**Use when:** guaranteed O(n log n) is required, stability matters, or you're sorting linked lists (where Quick Sort performs poorly).

---

**Q: How does Quick Sort work?**

Quick Sort picks a **pivot**, partitions the array so all elements less than the pivot come before it and all greater come after, then recursively sorts both partitions.

```java
void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}

int partition(int[] arr, int low, int high) {
    int pivot = arr[high];   // choose last element as pivot
    int i = low - 1;         // index of smaller element

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;  // swap
        }
    }
    int temp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = temp;  // place pivot
    return i + 1;
}
```

| | Value |
|---|---|
| Time (average) | O(n log n) |
| Time (worst) | O(n²) — when pivot is always min/max (sorted input with naive pivot) |
| Space | O(log n) — call stack |
| Stable | No |

**In practice, Quick Sort is often faster than Merge Sort** despite the same average complexity — it has better cache locality (in-place) and lower constant factors. Java uses Dual-Pivot Quicksort for primitive arrays for this reason.

---

## 3. Recursion & Divide and Conquer

**Q: What is recursion and what are the requirements for a correct recursive function?**

Recursion is when a function calls itself to solve a smaller version of the same problem.

Every recursive function needs:
1. **Base case** — a condition that stops the recursion (no more self-calls).
2. **Recursive case** — calls itself with a *smaller/simpler* input that moves toward the base case.
3. **Progress** — each call must get closer to the base case, otherwise you get infinite recursion and a `StackOverflowError`.

```java
// Factorial: n! = n * (n-1)!
int factorial(int n) {
    if (n <= 1) return 1;               // base case
    return n * factorial(n - 1);        // recursive case — n decreases by 1 each call
}

// Sum of array elements
int sum(int[] arr, int i) {
    if (i == arr.length) return 0;      // base case
    return arr[i] + sum(arr, i + 1);    // recursive case
}
```

**When to prefer recursion over iteration:**
- Problems with naturally recursive structure: trees, graphs, divide-and-conquer.
- When the recursive solution is significantly cleaner and n is bounded (no stack overflow risk).

**When to be careful:** Deep recursion (n > ~10,000 in Java) can overflow the call stack. Convert to iteration with an explicit stack if needed.

---

**Q: What is divide and conquer?**

Divide and conquer breaks a problem into smaller subproblems of the same type, solves them recursively, then combines the results.

Three steps:
1. **Divide** — split the problem into smaller subproblems.
2. **Conquer** — solve each subproblem recursively (base case: solve directly when small enough).
3. **Combine** — merge the subproblem solutions into the overall answer.

**Examples:** Merge Sort (divide array, sort halves, merge), Binary Search (divide in half, search one side), Quick Sort (partition, sort partitions).

```java
// Classic: find maximum in array using divide and conquer
int findMax(int[] arr, int left, int right) {
    if (left == right) return arr[left];         // base case: one element

    int mid = left + (right - left) / 2;
    int leftMax  = findMax(arr, left, mid);      // conquer left
    int rightMax = findMax(arr, mid + 1, right); // conquer right
    return Math.max(leftMax, rightMax);           // combine
}
```

---

## 4. Two Pointers & Sliding Window

**Q: What is the two-pointer technique?**

Two pointers is a pattern where you maintain two indices into an array (or string) and move them based on conditions — often starting from opposite ends or both from the left. Turns O(n²) brute force solutions into O(n).

**Example: Check if a sorted array has two numbers that sum to a target**
```java
// Brute force: O(n²) — try every pair
// Two pointers: O(n)

boolean hasPairWithSum(int[] sortedArr, int target) {
    int left = 0, right = sortedArr.length - 1;

    while (left < right) {
        int sum = sortedArr[left] + sortedArr[right];
        if (sum == target) return true;
        else if (sum < target) left++;    // need larger sum → move left pointer right
        else right--;                     // need smaller sum → move right pointer left
    }
    return false;
}
```

**Example: Reverse an array in place**
```java
void reverse(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left++] = arr[right];
        arr[right--] = temp;
    }
}
```

**Recognise two-pointer problems:** sorted array + pair/triplet sum, palindrome check, removing duplicates in-place, merging two sorted arrays.

---

**Q: What is the sliding window technique?**

Sliding window maintains a contiguous subarray (window) of variable or fixed size, sliding it across the array without recomputing the whole window from scratch each time. Reduces O(n²) brute force to O(n).

**Example: Maximum sum of any subarray of size k**
```java
// Brute force: O(n*k) — recompute sum of k elements at every position
// Sliding window: O(n) — add next element, remove first element

int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];   // initial window

    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];   // slide: add new, remove old
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

**Variable-size window example: Longest substring without repeating characters**
```java
int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        while (window.contains(s.charAt(right))) {
            window.remove(s.charAt(left++));   // shrink window from left
        }
        window.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## 5. Hashing Patterns

**Q: What is the frequency map pattern?**

Count occurrences of elements using a HashMap, then use the map for O(1) lookups instead of O(n) scans.

```java
// Check if two strings are anagrams
boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;

    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    for (char c : t.toCharArray()) {
        if (!freq.containsKey(c)) return false;
        freq.merge(c, -1, Integer::sum);
        if (freq.get(c) == 0) freq.remove(c);
    }
    return freq.isEmpty();
}

// Find the first non-repeating character
char firstUnique(String s) {
    Map<Character, Integer> freq = new LinkedHashMap<>();  // insertion order preserved
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    for (Map.Entry<Character, Integer> e : freq.entrySet()) {
        if (e.getValue() == 1) return e.getKey();
    }
    return '\0';
}
```

**Two-sum using HashMap — O(n) instead of O(n²):**
```java
int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();   // value → index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{};
}
```

---

## 6. Tree & Graph Traversal

**Q: What is DFS and BFS? When do you use each?**

Both are ways to visit all nodes in a tree or graph.

**DFS (Depth-First Search)** — go as deep as possible down one path before backtracking.

```java
// DFS on a binary tree — recursive (uses call stack implicitly)
void dfs(TreeNode node) {
    if (node == null) return;
    System.out.print(node.val + " ");   // pre-order: process before children
    dfs(node.left);
    dfs(node.right);
}

// DFS iterative — explicit stack
void dfsIterative(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        System.out.print(node.val + " ");
        if (node.right != null) stack.push(node.right);
        if (node.left != null) stack.push(node.left);
    }
}
```

**BFS (Breadth-First Search)** — visit all nodes at the current depth before going deeper. Uses a queue.

```java
void bfs(TreeNode root) {
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size();        // number of nodes at this level
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            System.out.print(node.val + " ");
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        System.out.println();   // new line after each level
    }
}
```

| | DFS | BFS |
|---|---|---|
| Data structure | Stack (or recursion) | Queue |
| Memory | O(h) — tree height | O(w) — max width of tree |
| Use for | Path finding, cycle detection, topological sort, exploring all solutions | Shortest path (unweighted), level-order traversal, "closest" problems |
| Best when | Tree is deep, answer likely deep | Tree is wide, answer likely near root |

**Three DFS orderings for binary trees:**
- **Pre-order** (root → left → right) — copy/serialize a tree.
- **In-order** (left → root → right) — visits BST nodes in sorted order.
- **Post-order** (left → right → root) — delete a tree, evaluate expression trees.

---

## 7. Dynamic Programming Intro

**Q: What is dynamic programming?**

Dynamic Programming (DP) solves problems by breaking them into **overlapping subproblems**, solving each subproblem only once, and storing results to avoid recomputation. It applies when a problem has **optimal substructure** (optimal solution contains optimal solutions to subproblems) and **overlapping subproblems** (same subproblems recur).

Two approaches:
- **Top-down (memoization)** — recursive solution + cache results.
- **Bottom-up (tabulation)** — iteratively build up solutions from base cases.

**Classic example: Fibonacci**
```java
// Naive recursive — O(2ⁿ), recomputes the same values millions of times
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

// Top-down DP (memoization) — O(n) time, O(n) space
int fib(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != 0) return memo[n];      // already computed — return cached
    memo[n] = fib(n-1, memo) + fib(n-2, memo);
    return memo[n];
}

// Bottom-up DP (tabulation) — O(n) time, O(1) space
int fib(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

**Classic example: Longest Common Subsequence (LCS)**
```java
int lcs(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];   // dp[i][j] = LCS of s1[0..i-1] and s2[0..j-1]

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1))
                dp[i][j] = dp[i-1][j-1] + 1;   // characters match — extend LCS
            else
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);  // take best of skipping either
        }
    }
    return dp[m][n];
}
```

**How to recognise a DP problem:**
- "Find the maximum/minimum..."
- "How many ways to..."
- "Is it possible to..."
- The problem has choices at each step with consequences that affect future steps.
- Brute force is exponential but has repeated subproblems.

**DP vs Divide and Conquer:** Both break problems into subproblems. The key difference — DP subproblems **overlap** (same sub-problem appears multiple times). Divide and Conquer subproblems are **independent** (no overlap, no caching needed). Merge Sort = Divide and Conquer. Fibonacci = DP.
