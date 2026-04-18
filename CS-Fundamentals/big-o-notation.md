# Big O Notation — Interview Prep

## Table of Contents
- [1. What is Big O?](#1-what-is-big-o)
- [2. Common Complexity Classes](#2-common-complexity-classes)
- [3. Rules for Calculating Big O](#3-rules-for-calculating-big-o)
- [4. Space Complexity](#4-space-complexity)
- [5. Data Structure Complexity Reference](#5-data-structure-complexity-reference)
- [6. Sorting Algorithm Complexity Reference](#6-sorting-algorithm-complexity-reference)
- [7. Recognizing Complexity in Code](#7-recognizing-complexity-in-code)
- [8. Thought-Provoking Questions](#8-thought-provoking-questions)

---

## 1. What is Big O?

**Q: What is Big O notation and what does it actually measure?**

Big O notation describes how the **runtime or memory usage of an algorithm grows relative to the size of the input** as that input approaches infinity. It gives an upper bound — the worst-case growth rate — so you can compare algorithms independently of hardware, language, or constant factors.

The variable `n` typically represents the size of the input (number of elements in an array, nodes in a tree, characters in a string, etc.).

**What Big O is NOT:**
- It is not the exact runtime in milliseconds.
- It ignores constant factors and lower-order terms — O(2n) simplifies to O(n), O(n² + n) simplifies to O(n²).
- It describes *growth rate*, not absolute speed. An O(n²) algorithm can be faster than O(n log n) for very small n.

**Three types of analysis:**
- **Best case (Ω Omega)** — ideal input (e.g., already sorted). Rarely useful.
- **Average case (Θ Theta)** — expected performance over all inputs.
- **Worst case (O Big O)** — most commonly discussed. Guarantees an upper bound.

---

## 2. Common Complexity Classes

**Q: What are the common Big O complexities, from fastest to slowest?**

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

### O(1) — Constant Time

Runtime does not change regardless of input size.

```java
int first = array[0];          // array index access
map.get("key");                // HashMap lookup
stack.push(x);                 // push to stack
```

No matter if the array has 10 or 10 million elements — these take the same time.

---

### O(log n) — Logarithmic Time

Input is divided (usually halved) at each step. Extremely efficient for large inputs.

```
n = 1,000,000 → log₂(n) ≈ 20 steps
n = 1,000,000,000 → log₂(n) ≈ 30 steps
```

```java
// Binary search — halves the search space each step
// TreeMap/TreeSet operations — balanced BST height is log n
```

Every time you see "divide in half and recurse/iterate," think O(log n).

---

### O(n) — Linear Time

Runtime grows directly proportional to input size. Must touch every element at least once.

```java
for (int x : array) {          // single loop over n elements
    System.out.println(x);
}

// Finding max in unsorted array, counting occurrences, building a HashMap from a list
```

---

### O(n log n) — Linearithmic Time

Common in efficient sorting algorithms. Think "sort n items, and sorting involves log n levels of work per item."

```java
Arrays.sort(array);            // Timsort — O(n log n)
Collections.sort(list);        // O(n log n)
// Merge sort, heap sort
```

This is the best achievable for comparison-based sorting.

---

### O(n²) — Quadratic Time

A nested loop where both loops depend on n. Performance degrades fast — 10x more data = 100x more work.

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {   // nested loop → O(n²)
        // ...
    }
}
// Bubble sort, insertion sort, selection sort
// Comparing every pair of elements
```

Acceptable for small n (< ~1000). Avoid for large inputs.

---

### O(2ⁿ) — Exponential Time

Common in brute-force recursive algorithms — each call branches into two more. Adding one element to input doubles the work.

```java
// Naive recursive Fibonacci
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);  // two branches every call
}
// fib(50) makes ~2^50 ≈ 1 quadrillion calls
```

Also appears in: generating all subsets of a set (2ⁿ subsets), brute-force password cracking.

---

### O(n!) — Factorial Time

Generating all permutations of n items. Becomes unusable beyond n ≈ 12.

```
n=10 → 3,628,800 operations
n=15 → 1,307,674,368,000 operations
```

Appears in: brute-force Travelling Salesman Problem.

---

**Q: How do these complexities compare at scale?**

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) |
|---|---|---|---|---|---|---|
| 10 | 1 | 3 | 10 | 33 | 100 | 1,024 |
| 100 | 1 | 7 | 100 | 664 | 10,000 | 10³⁰ |
| 1,000 | 1 | 10 | 1,000 | 9,965 | 1,000,000 | ∞ |
| 1,000,000 | 1 | 20 | 1,000,000 | 19,900,000 | 10¹² | ∞ |

The gap between O(n log n) and O(n²) becomes catastrophic at scale — this is why the choice of algorithm matters enormously for large datasets.

---

## 3. Rules for Calculating Big O

**Q: How do you calculate the Big O of a piece of code?**

### Rule 1: Drop constants

O(2n) → O(n). O(500) → O(1). Constants are hardware/implementation details, not growth characteristics.

```java
for (int x : arr) { process(x); }  // O(n)
for (int x : arr) { log(x); }      // O(n)
// Together: O(2n) → O(n)
```

### Rule 2: Drop non-dominant terms

O(n² + n) → O(n²). For large n, the n² term overwhelms the n term completely.

O(n³ + n² + n + 1) → O(n³)

### Rule 3: Sequential steps add

Two separate loops over the same array: O(n) + O(n) = O(2n) → O(n).

```java
for (int x : arr) { /* O(n) */ }
for (int x : arr) { /* O(n) */ }
// Total: O(n) — not O(n²)
```

### Rule 4: Nested loops multiply

A loop inside a loop: O(n) × O(n) = O(n²).

```java
for (int i = 0; i < n; i++) {        // O(n)
    for (int j = 0; j < n; j++) {    // O(n)
        // O(n²)
    }
}
```

### Rule 5: Different variables stay separate

If a loop iterates over array A (size `n`) and a nested loop iterates over array B (size `m`), the complexity is O(n × m), NOT O(n²) — unless n and m are equal.

```java
for (String a : listA) {          // O(n)
    for (String b : listB) {      // O(m)
        if (a.equals(b)) ...
    }
}
// O(n * m) — only O(n²) if both lists are the same size
```

### Rule 6: Recursive complexity — use the recurrence relation

For a recursive function, multiply: (number of recursive calls) ^ (depth of recursion).

```java
// Binary search — 1 call per level, log n levels → O(log n)
// Merge sort — 2 calls per level, log n levels, O(n) merge work → O(n log n)
// Naive fibonacci — 2 calls, n levels → O(2ⁿ)
```

---

## 4. Space Complexity

**Q: What is space complexity and how is it different from time complexity?**

Space complexity measures how much **extra memory** an algorithm uses relative to input size. It counts auxiliary space (the memory your algorithm allocates), not the input itself.

```java
// O(1) space — uses a fixed number of variables regardless of n
int sum(int[] arr) {
    int total = 0;            // one variable
    for (int x : arr) total += x;
    return total;
}

// O(n) space — allocates a new structure proportional to input
int[] doubled(int[] arr) {
    int[] result = new int[arr.length];  // n extra elements
    for (int i = 0; i < arr.length; i++) result[i] = arr[i] * 2;
    return result;
}

// O(n) space — call stack depth is n for n recursive calls
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);   // n stack frames
}

// O(log n) space — binary search recursion stack depth is log n
```

**Time vs space tradeoffs** are common — memoization (caching) trades O(n) space to reduce O(2ⁿ) time to O(n). Hash tables trade O(n) space for O(1) lookup instead of O(n) linear search.

---

## 5. Data Structure Complexity Reference

**Q: What are the time complexities of common data structure operations?**

### Arrays / ArrayList

| Operation | Array | ArrayList |
|---|---|---|
| Access by index | O(1) | O(1) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted, binary) | O(log n) | O(log n) |
| Insert at end | O(1)* | O(1) amortized* |
| Insert at middle/start | O(n) | O(n) |
| Delete at middle/start | O(n) | O(n) |

*Amortized — occasional resizing is O(n) but averaged over all inserts is O(1).

### LinkedList

| Operation | Time |
|---|---|
| Access by index | O(n) |
| Insert/Delete at head or tail | O(1) |
| Insert/Delete at middle (given node ref) | O(1) |
| Search | O(n) |

### HashMap / HashSet

| Operation | Average | Worst Case |
|---|---|---|
| get / put / remove / contains | O(1) | O(n)* |

*Worst case is O(n) with all keys in the same bucket (pathological hash collisions). In practice, treated as O(1).

### TreeMap / TreeSet (Red-Black Tree)

| Operation | Time |
|---|---|
| get / put / remove / contains | O(log n) |
| First / Last key | O(log n) |
| Range query (subMap, headMap) | O(log n + k) |

### Stack / Queue (ArrayDeque)

| Operation | Time |
|---|---|
| push / pop / peek (stack) | O(1) |
| offer / poll / peek (queue) | O(1) |

### PriorityQueue (Binary Heap)

| Operation | Time |
|---|---|
| offer (insert) | O(log n) |
| poll (remove min/max) | O(log n) |
| peek (read min/max) | O(1) |
| Building a heap from n elements | O(n) |

---

## 6. Sorting Algorithm Complexity Reference

**Q: What are the complexities of common sorting algorithms?**

| Algorithm | Best | Average | Worst | Space | Stable? |
|---|---|---|---|---|---|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | No |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | Yes |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²)* | O(log n) | No |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| **Counting Sort** | O(n + k) | O(n + k) | O(n + k) | O(k) | Yes |
| **Timsort** (Java default) | O(n) | O(n log n) | O(n log n) | O(n) | Yes |

*Quick Sort worst case (O(n²)) occurs when the pivot is always the smallest or largest element (already-sorted input with naive pivot). Mitigated with randomized pivot selection.

**A stable sort** preserves the relative order of equal elements. Matters when you sort by one field and want to maintain order from a previous sort.

**Java's `Arrays.sort()`:**
- Primitive arrays → **Dual-Pivot Quicksort** — O(n log n) average, not stable (primitives don't need stability).
- Object arrays → **Timsort** — O(n log n), stable.

---

## 7. Recognizing Complexity in Code

**Q: How do I quickly identify the complexity of a given piece of code?**

**Pattern → Complexity cheat sheet:**

| Code Pattern | Complexity |
|---|---|
| Single statement, array index, map get | O(1) |
| One loop from 0 to n | O(n) |
| Two separate loops from 0 to n | O(n) |
| Loop from 0 to n, inside loop from 0 to n | O(n²) |
| Loop that halves i each iteration (`i /= 2`) | O(log n) |
| Loop 0 to n, inner loop 0 to log n | O(n log n) |
| Recursive call on half the input | O(log n) |
| Recursive with 2 calls on halved input + O(n) work | O(n log n) |
| Recursive with 2 calls on n-1 input | O(2ⁿ) |

```java
// O(log n) — i halves each iteration
for (int i = n; i > 1; i /= 2) {
    System.out.println(i);
}

// O(n log n) — outer O(n), inner O(log n)
for (int i = 0; i < n; i++) {            // O(n)
    for (int j = 1; j < n; j *= 2) {     // O(log n)
        System.out.println(i + j);
    }
}

// O(n²) even though inner loop doesn't always run n times
// Sum of 1 + 2 + 3 + ... + n = n(n+1)/2 → O(n²)
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {   // inner starts at i, not 0
        // still O(n²) — triangular number
    }
}
```

---

## 8. Thought-Provoking Questions

**Q: If an algorithm is O(n log n) and another is O(n²), is the first always faster?**

Not necessarily — for small n, O(n²) can be faster due to lower constant factors. Insertion sort (O(n²)) is often faster than merge sort (O(n log n)) for arrays of size < 10–20 because it has almost no overhead and benefits from cache locality. This is why Timsort uses insertion sort for small subarrays and merge sort for the rest.

Big O describes *asymptotic* growth — it tells you which algorithm wins as n gets large, but constants matter for small inputs.

---

**Q: What does "amortized O(1)" mean for ArrayList?**

Most `ArrayList.add()` calls are O(1) — just writing to the next slot. But occasionally the backing array is full and must be resized: a new array is allocated (usually 2×) and all elements are copied over — that single operation is O(n).

However, if you add n elements total, the resizes happen at sizes 1, 2, 4, 8, ..., n — total copy work is `1 + 2 + 4 + ... + n = 2n`. Spread across n insertions, that's O(2n / n) = O(1) per insertion on average. This is amortized O(1) — expensive operations are rare enough that the average cost stays constant.

---

**Q: Can you have O(1) space AND O(1) time? What does O(n) space actually mean in practice?**

Yes — simple operations like `a + b` are both O(1) time and O(1) space. O(n) space means your algorithm allocates memory that grows with n — a new array of size n, a recursive call stack n levels deep, a HashMap with n entries. It doesn't count the input itself, only the *extra* memory you allocate.

The practical implication: an O(n) space algorithm can run out of memory on very large inputs even if its time complexity is acceptable. For example, recursion that's O(n) deep can hit a stack overflow for n = 100,000.

---

**Q: Why can't comparison-based sorting do better than O(n log n)?**

Any comparison-based sort must be able to distinguish between all n! possible orderings of n elements. A binary decision tree of comparisons must have at least n! leaves, which requires a tree of height at least log₂(n!) ≈ n log n (by Stirling's approximation). Therefore, at least O(n log n) comparisons are needed in the worst case — this is a mathematical lower bound, not a limitation of known algorithms.

Sorting algorithms like Counting Sort and Radix Sort beat this bound by NOT using comparisons — they exploit the structure of the keys (e.g., integers in a known range).
