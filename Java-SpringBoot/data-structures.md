# Java — Data Structures Interview Prep

## Table of Contents
- [1. Arrays](#1-arrays)
- [2. Lists — ArrayList vs LinkedList](#2-lists--arraylist-vs-linkedlist)
- [3. Array vs List — Deep Comparison](#3-array-vs-list--deep-comparison)
- [4. Stack](#4-stack)
- [5. Queue & Deque](#5-queue--deque)
- [6. Tree & Binary Search Tree](#6-tree--binary-search-tree)
- [7. HashMap & Map Implementations](#7-hashmap--map-implementations)
- [8. Set Implementations](#8-set-implementations)
- [9. Set vs List — When to Use Which](#9-set-vs-list--when-to-use-which)
- [10. How Java Stores These Structures in Memory](#10-how-java-stores-these-structures-in-memory)
- [11. Java vs Python — Data Structure Comparison](#11-java-vs-python--data-structure-comparison)
- [12. Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. Arrays

**Q: What is an array and how does it work in memory?**

An array is a **fixed-size, contiguous block of memory** holding elements of the same type. Because elements are laid out back-to-back in memory, any element can be reached in O(1) time by computing its address from the base address and the index.

```java
int[] nums = new int[5];              // all initialised to 0
int[] primes = {2, 3, 5, 7, 11};     // literal initialisation

primes[0];          // O(1) — direct address calculation
primes[2] = 99;     // O(1) — direct write
primes.length;      // 5 — fixed, cannot change
```

**Memory layout for `int[] primes`:**

```
Index:  [0]  [1]  [2]  [3]  [4]
Value:    2    3    5    7   11
Memory: 100  104  108  112  116   (each int = 4 bytes, addresses are illustrative)
```

`primes[2]` → base address (100) + 2 × 4 = 108. Direct. No traversal.

**Complexity:**

| Operation | Time |
|---|---|
| Access by index | O(1) |
| Update by index | O(1) |
| Search (unsorted) | O(n) |
| Search (sorted, binary search) | O(log n) |
| Insert / Delete at middle | O(n) — must shift all subsequent elements |
| Insert at end (if space) | O(1) |

**Use an array when:**
- Size is known and fixed upfront.
- You need raw performance with primitive types — no boxing, no object overhead.
- You're working with multi-dimensional data (matrices, grids).
- You're calling a library or legacy API that expects an array.

**Default values:** `int[]` → 0, `boolean[]` → false, `String[]` → null. Forgetting null defaults on reference arrays is a common source of `NullPointerException`.

**Multi-dimensional arrays:**
```java
int[][] matrix = new int[3][4];      // 3 rows, 4 columns
matrix[1][2] = 42;

// Jagged array — rows can have different lengths
int[][] jagged = new int[3][];
jagged[0] = new int[2];
jagged[1] = new int[5];
jagged[2] = new int[1];
```

---

## 2. Lists — ArrayList vs LinkedList

**Q: What is the `List` interface and what are its main implementations?**

`List` is an **ordered, index-accessible collection that allows duplicates**. "Ordered" means insertion order is preserved — element 0 is always the first element added (unless you rearrange). Java provides two main implementations with very different internal structures.

### ArrayList

Backed by a **resizable array**. Elements are stored contiguously. When the array fills up, a new array (1.5× or 2× larger) is allocated and elements are copied over.

```java
List<String> list = new ArrayList<>();   // default capacity: 10
list.add("Alice");
list.add("Bob");
list.add("Carol");

list.get(1);            // "Bob"  — O(1) direct index access
list.add(1, "Dave");    // inserts at index 1, shifts Bob and Carol right — O(n)
list.remove(0);         // removes Alice, shifts everything left — O(n)
list.size();            // current number of elements
list.contains("Bob");   // O(n) linear scan
```

**Pre-size when you know the approximate count:**
```java
List<String> names = new ArrayList<>(500);  // avoids repeated resizing
```

### LinkedList

Backed by a **doubly-linked list**. Each node holds a value and two references: `prev` and `next`.

```
null ← [prev|"Alice"|next] ↔ [prev|"Bob"|next] ↔ [prev|"Carol"|next] → null
         head                                          tail
```

```java
LinkedList<String> ll = new LinkedList<>();
ll.add("Alice");
ll.addFirst("Zara");     // O(1) — prepend
ll.addLast("Bob");       // O(1) — append
ll.get(1);               // O(n) — must traverse from head
ll.removeFirst();        // O(1)
ll.removeLast();         // O(1)
```

**Complexity comparison:**

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(i)` — access by index | O(1) | O(n) — traverse from head/tail |
| `add(e)` — append to end | O(1) amortized | O(1) |
| `add(0, e)` — prepend | O(n) — shifts all | O(1) |
| `add(i, e)` — insert middle | O(n) | O(n) to find pos + O(1) to link |
| `remove(i)` — remove by index | O(n) — shifts | O(n) to find + O(1) to unlink |
| `contains(e)` | O(n) | O(n) |
| Memory per element | Small (just the value) | Larger (value + 2 references) |
| Cache performance | Excellent — contiguous | Poor — nodes scattered on heap |

**The default choice is almost always `ArrayList`.** Cache locality makes iteration significantly faster in practice — even for operations where LinkedList has theoretical O(1) advantage. Prefer `LinkedList` only when you specifically need the `Deque` interface (double-ended queue operations).

---

## 3. Array vs List — Deep Comparison

**Q: When do you actually use an array vs an ArrayList?**

This question has a short answer ("use ArrayList") that hides important tradeoffs.

### Memory: primitives vs boxing

A `int[]` stores raw 4-byte integers packed in memory — zero overhead.  
An `ArrayList<Integer>` stores references to boxed `Integer` objects on the heap. Each `Integer` is a full object (~16 bytes header) plus an 8-byte reference in the list.

```java
int[] primitiveArray = new int[1_000_000];         // ≈ 4 MB
List<Integer> boxedList = new ArrayList<>(1_000_000); // ≈ 20+ MB + GC pressure
```

This matters at scale or in hot loops. Unnecessary boxing/unboxing also adds CPU overhead.

### Resizability

Arrays are fixed at creation. Adding more elements requires allocating a new array and copying — which is exactly what `ArrayList` does internally on resize. If you know the size upfront and it won't change, an array avoids that overhead entirely.

### Type safety: covariance vs invariance

Arrays are **covariant** — `String[]` is a subtype of `Object[]`. This compiles but can throw at runtime:

```java
String[] strings = new String[3];
Object[] objects = strings;    // legal — covariant
objects[0] = 42;               // compiles! but throws ArrayStoreException at runtime
```

Generics are **invariant** — the wrong assignment fails at compile time (safer):

```java
List<String> strList = new ArrayList<>();
List<Object> objList = strList;   // compile error — caught early, as it should be
```

### API richness

`List` integrates with the full Java Collections framework: `Collections.sort()`, `Collections.shuffle()`, `stream()`, `removeIf()`, `subList()`. Arrays need `Arrays.sort()` / `Arrays.stream()` and are otherwise second-class citizens.

```java
List<String> names = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));
Collections.sort(names);
names.removeIf(n -> n.startsWith("C"));
String result = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.joining(", "));
```

### Converting between them

```java
// Array → List (fixed-size view backed by array — no add/remove)
String[] arr = {"a", "b", "c"};
List<String> fixed   = Arrays.asList(arr);
List<String> mutable = new ArrayList<>(Arrays.asList(arr));  // fully mutable copy

// List → Array
String[] back = list.toArray(new String[0]);

// Primitive array → stream
int[] nums = {1, 2, 3, 4, 5};
int sum  = Arrays.stream(nums).sum();
int max  = Arrays.stream(nums).max().getAsInt();
int[] doubled = Arrays.stream(nums).map(x -> x * 2).toArray();
```

### Decision guide

| Situation | Use |
|---|---|
| Size fixed and known | Array — simpler, no overhead |
| Primitive types, large dataset, performance-critical | `int[]` / `long[]` — no boxing |
| Multi-dimensional grid / matrix | `int[][]` — natural, efficient |
| Dynamic size | `ArrayList` |
| Need Collections API (`sort`, `stream`, `removeIf`) | `ArrayList` |
| General-purpose everyday code | `ArrayList` — the default |

---

## 4. Stack

**Q: What is a Stack and how do you use it in Java?**

A Stack is a **Last-In, First-Out (LIFO)** structure — the last element pushed is the first popped. Think of a stack of plates: you always add and remove from the top.

**Do not use the legacy `Stack` class** — it extends `Vector`, which is thread-synchronized on every operation (unnecessary overhead in single-threaded code) and carries outdated API design. Use `ArrayDeque` instead:

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);   // [10]
stack.push(20);   // [20, 10]
stack.push(30);   // [30, 20, 10]

stack.peek();     // 30  — inspect top, no removal
stack.pop();      // 30  — removes and returns → [20, 10]
stack.isEmpty();  // false
stack.size();     // 2
```

| Operation | Time |
|---|---|
| `push` / `pop` / `peek` | O(1) |

**When to use a Stack:**

- **Undo/redo** — push each action; undo pops the last.
- **Balanced brackets validation** — push `(`, `[`, `{`; pop and verify on closing bracket.
- **DFS (Depth-First Search)** — use an explicit stack instead of recursion to avoid stack overflow.
- **Expression evaluation / parsing** — operators and operands pushed onto separate stacks.
- **Backtracking** — save state before exploring a path; pop to restore on backtrack.
- **Call stack simulation** — any algorithm that uses recursion can be rewritten with an explicit stack.

**Example — check balanced brackets:**
```java
boolean isBalanced(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
```

---

## 5. Queue & Deque

**Q: What is a Queue and how do you use it in Java?**

A Queue is a **First-In, First-Out (FIFO)** structure — elements are added at the back (enqueue) and removed from the front (dequeue). Think of a checkout line.

Use `ArrayDeque` (not `LinkedList`) for best performance — it's backed by a resizable circular array:

```java
Queue<String> queue = new ArrayDeque<>();

queue.offer("Alice");    // enqueue → ["Alice"]
queue.offer("Bob");      // ["Alice", "Bob"]
queue.offer("Carol");    // ["Alice", "Bob", "Carol"]

queue.peek();    // "Alice" — front, no removal
queue.poll();    // "Alice" — remove from front → ["Bob", "Carol"]
queue.size();    // 2
```

**`offer` vs `add`, `poll` vs `remove`, `peek` vs `element`:**  
`offer`/`poll`/`peek` return a special value (false/null) on failure.  
`add`/`remove`/`element` throw exceptions on failure.  
Prefer the value-returning versions for queues.

| Operation | Time |
|---|---|
| `offer` (enqueue) / `poll` (dequeue) / `peek` | O(1) |

**When to use a Queue:**
- Processing tasks in arrival order (job queues, request buffers, print queues).
- **BFS (Breadth-First Search)** — visit all neighbours at current depth before going deeper.
- Rate limiting / throttling — buffer incoming requests.
- Producer-consumer patterns — use `BlockingQueue` (`ArrayBlockingQueue`, `LinkedBlockingQueue`) for thread-safe handoff between threads.

### PriorityQueue

A PriorityQueue processes the element with the **highest priority** first (smallest value by default — a min-heap). Order is NOT insertion order.

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
minHeap.poll();   // 1 — always removes the minimum

// Max-heap — reverse the comparator
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5); maxHeap.offer(1); maxHeap.offer(3);
maxHeap.poll();   // 5

// Custom priority — objects sorted by a field
PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority)
);
```

| Operation | Time |
|---|---|
| `offer` (insert) | O(log n) |
| `poll` (remove min/max) | O(log n) |
| `peek` (read min/max) | O(1) |

**When to use PriorityQueue:** scheduling (highest-priority job first), Dijkstra's shortest path, finding the k largest/smallest elements, event-driven simulations.

### Deque (Double-Ended Queue)

`ArrayDeque` implements `Deque` — you can add/remove from **both ends** efficiently. This makes it serve as both a stack and a queue.

```java
Deque<String> deque = new ArrayDeque<>();

deque.addFirst("B");    // [B]
deque.addLast("C");     // [B, C]
deque.addFirst("A");    // [A, B, C]

deque.peekFirst();      // "A"
deque.peekLast();       // "C"
deque.pollFirst();      // "A" → [B, C]
deque.pollLast();       // "C" → [B]
```

**Use Deque when:** sliding window problems (track max/min of a window), palindrome checks, work-stealing algorithms.

---

## 6. Tree & Binary Search Tree

**Q: What is a tree and what makes a Binary Search Tree special?**

A **tree** is a hierarchical structure where each node has zero or more children and exactly one parent (except the root, which has none). No cycles.

Key vocabulary:
- **Root** — the top node with no parent.
- **Leaf** — a node with no children.
- **Height** — longest path from root to a leaf.
- **Depth** — distance from root to a specific node.
- **Subtree** — a node and all its descendants.

A **Binary Tree** has at most 2 children per node (left and right).

A **Binary Search Tree (BST)** adds an ordering invariant:
- Every value in the **left** subtree is **less than** the node's value.
- Every value in the **right** subtree is **greater than** the node's value.
- This holds recursively for every node.

```
       8
      / \
     3   10
    / \    \
   1   6    14
      / \
     4   7

Search for 7:
8 → 7 < 8 → go left → 3 → 7 > 3 → go right → 6 → 7 > 6 → go right → 7 ✓
3 comparisons instead of scanning all 7 nodes.
```

**Node structure in Java:**
```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int val) { this.val = val; }
}
```

**BST complexity:**

| Operation | Average (balanced) | Worst (skewed/linear) |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Min / Max | O(log n) | O(n) |
| In-order traversal | O(n) | O(n) |

A **skewed tree** occurs when you insert already-sorted data — each node only has a right child, and the tree degrades to a linked list. **Self-balancing trees** (AVL, Red-Black) automatically restructure to keep height O(log n).

**Java's `TreeMap` and `TreeSet`** are backed by a Red-Black tree — guaranteed O(log n) for all operations.

**Tree traversals:**
```java
// In-order (Left → Root → Right) — visits BST values in sorted ascending order
void inOrder(TreeNode node) {
    if (node == null) return;
    inOrder(node.left);
    System.out.print(node.val + " ");
    inOrder(node.right);
}

// Pre-order (Root → Left → Right) — useful for copying/serializing a tree
void preOrder(TreeNode node) {
    if (node == null) return;
    System.out.print(node.val + " ");
    preOrder(node.left);
    preOrder(node.right);
}

// Post-order (Left → Right → Root) — useful for deleting a tree, evaluating expression trees
void postOrder(TreeNode node) {
    if (node == null) return;
    postOrder(node.left);
    postOrder(node.right);
    System.out.print(node.val + " ");
}
```

**Common tree types and where they appear:**

| Tree Type | Characteristics | Used in |
|---|---|---|
| BST | Left < root < right | Manual sorted structures |
| AVL Tree | Strictly balanced (height diff ≤ 1) | When reads dominate |
| Red-Black Tree | Loosely balanced | `TreeMap`, `TreeSet` in Java |
| Heap | Parent ≤ children (min-heap) | `PriorityQueue` |
| Trie | Each node is a character | Autocomplete, spell check |
| B-Tree / B+ Tree | Highly branched, stays flat | Database indexes, filesystems |

**When to use a tree:**
- Need **sorted data** with O(log n) insert/search/delete → `TreeMap` / `TreeSet`.
- Need **range queries** (all keys between A and B) → `TreeMap.subMap()`.
- Representing **hierarchical data** (file systems, org charts, JSON/XML).
- Priority processing → heap (`PriorityQueue`).

---

## 7. HashMap & Map Implementations

**Q: What is a HashMap and how does it work internally?**

A `HashMap<K, V>` stores **key-value pairs** with O(1) average get and put. It is the most-used Map implementation.

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);
scores.put("Alice", 99);      // overwrites — same key

scores.get("Alice");                   // 99
scores.getOrDefault("Carol", 0);       // 0 — safe default
scores.containsKey("Bob");             // true
scores.putIfAbsent("Dave", 50);        // only inserts if key not present
scores.merge("Bob", 3, Integer::sum);  // Bob: 87 + 3 = 90

// Iterate
for (Map.Entry<String, Integer> e : scores.entrySet()) {
    System.out.println(e.getKey() + " → " + e.getValue());
}
scores.forEach((k, v) -> System.out.println(k + " → " + v));
```

**How it works internally (Java 8+):**

1. An internal `table` array of **buckets** is maintained (default capacity: 16).
2. On `put(key, value)`: compute `key.hashCode()`, apply a secondary hash, take `hash % capacity` → bucket index.
3. If bucket is **empty**: store the entry directly.
4. If bucket is **occupied (hash collision)**: entries in the same bucket are stored in a **linked list** initially. If a bucket grows beyond 8 entries (and total size > 64), the linked list converts to a **balanced tree** (Red-Black tree) → worst-case O(log n) per bucket instead of O(n).
5. When total entries exceed `capacity × loadFactor` (default 0.75), the table **resizes** (doubles) and all entries are rehashed.

This is why **`hashCode()` and `equals()` must both be correctly overridden for custom key classes**:
- `equals()` determines if two keys are logically the same.
- `hashCode()` must return the same value for equal objects — otherwise two equal keys hash to different buckets and you can never retrieve what you stored.

```java
// If you override equals, you MUST override hashCode
class Point {
    int x, y;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);    // consistent with equals
    }
}
```

**Map implementations compared:**

| | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| Internal structure | Hash table | Hash table + linked list | Red-Black tree |
| Key ordering | None (unpredictable) | Insertion order | Natural order or `Comparator` |
| `get` / `put` / `remove` | O(1) avg | O(1) avg | O(log n) |
| `null` keys | Yes (one) | Yes (one) | No (throws `NullPointerException`) |
| Use when | Fast lookup, order irrelevant | Need insertion order, e.g. LRU cache | Need sorted keys, range queries |

**Thread safety:** `HashMap` is NOT thread-safe. For concurrent access use `ConcurrentHashMap` (not `Hashtable` — it's legacy and uses coarse locking).

---

## 8. Set Implementations

**Q: What is a Set and how are its implementations different?**

A `Set` is a collection of **unique elements** — duplicates are silently ignored. It models the mathematical set.

```java
Set<String> set = new HashSet<>();
set.add("apple");
set.add("banana");
set.add("apple");    // duplicate — ignored, set unchanged
set.size();          // 2
set.contains("banana");  // true — O(1)
set.remove("apple");

// Set operations
Set<Integer> a = new HashSet<>(Set.of(1, 2, 3, 4));
Set<Integer> b = new HashSet<>(Set.of(3, 4, 5, 6));

Set<Integer> union        = new HashSet<>(a); union.addAll(b);      // {1,2,3,4,5,6}
Set<Integer> intersection = new HashSet<>(a); intersection.retainAll(b);  // {3,4}
Set<Integer> difference   = new HashSet<>(a); difference.removeAll(b);    // {1,2}
```

**Set implementations:**

| | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| Backed by | `HashMap` | `LinkedHashMap` | `TreeMap` (Red-Black tree) |
| Iteration order | No guarantee | Insertion order | Sorted ascending (or by `Comparator`) |
| `add` / `remove` / `contains` | O(1) avg | O(1) avg | O(log n) |
| `null` elements | Yes (one) | Yes (one) | No |
| Use when | Fast membership, order irrelevant | Unique + preserve insertion order | Unique + sorted, range queries |

```java
// HashSet — fastest, no order
Set<String> hs = new HashSet<>(List.of("banana", "apple", "cherry"));
// iteration order: unpredictable

// LinkedHashSet — unique + insertion order
Set<String> lhs = new LinkedHashSet<>(List.of("banana", "apple", "cherry"));
// iteration order: banana, apple, cherry

// TreeSet — always sorted
Set<String> ts = new TreeSet<>(List.of("banana", "apple", "cherry"));
// iteration order: apple, banana, cherry

// TreeSet range operations
ts.headSet("cherry");           // [apple, banana] — everything before "cherry"
ts.tailSet("banana");           // [banana, cherry] — "banana" and after
ts.subSet("apple", "cherry");   // [apple, banana]
ts.first();                     // "apple"
ts.last();                      // "cherry"
```

---

## 9. Set vs List — When to Use Which

**Q: How do I decide between a Set and a List?**

| | `List` (`ArrayList`) | `Set` (`HashSet`) |
|---|---|---|
| Duplicates | Allowed | Not allowed |
| Order | Insertion order maintained | No guaranteed order (`HashSet`) |
| Access by index | Yes — `get(i)` O(1) | No |
| `contains()` | O(n) — linear scan | O(1) — hash lookup |
| Iteration | Predictable order | Unpredictable (`HashSet`) |
| Null elements | Yes | Yes (one null in `HashSet`) |
| Use for | Ordered sequences, duplicates valid | Unique elements, fast membership |

**Concrete decision scenarios:**

**Use `List` when:**
- You care about **position** — "the third item", "the last item added".
- **Duplicates are meaningful** — a shopping cart with 3 of the same product.
- You need to **iterate in insertion order** reliably.
- You need **index-based access** — algorithms that jump around by index.

**Use `Set` when:**
- You need to **eliminate duplicates** — unique user IDs, distinct error codes.
- You need **fast membership testing** — "is this value in the allowed set?"
- You're performing **set algebra** — union, intersection, difference.

```java
// Deduplication — fast and idiomatic
List<String> tags = List.of("java", "spring", "java", "rest", "spring");
Set<String> uniqueTags = new HashSet<>(tags);   // {"java", "spring", "rest"}

// Fast membership test — O(1) vs O(n)
Set<String> validStatuses = Set.of("ACTIVE", "PENDING", "SUSPENDED");
if (validStatuses.contains(user.getStatus())) {
    // preferred over: validStatusesList.contains() which is O(n)
}

// When you need unique AND ordered → LinkedHashSet or TreeSet
Set<String> ordered = new LinkedHashSet<>(tags);   // [java, spring, rest] — insertion order, no dupes
```

**A common pattern — deduplicate while preserving order:**
```java
// LinkedHashSet gives you unique elements in the order they first appeared
List<String> input = List.of("c", "a", "b", "a", "c", "d");
List<String> deduped = new ArrayList<>(new LinkedHashSet<>(input));
// ["c", "a", "b", "d"]
```

---

## 10. How Java Stores These Structures in Memory

**Q: How does the JVM actually store each data structure?**

Understanding memory layout explains performance characteristics, helps you reason about GC pressure, and is exactly the kind of depth that stands out in interviews.

### The Java Heap and Object Layout

Every Java object on the heap has:
- **Object header** — 12–16 bytes (JVM metadata: class pointer, identity hashcode, GC/lock flags).
- **Fields** — the actual data.
- **Padding** — JVM aligns objects to 8-byte boundaries.

This means even a `new Integer(5)` takes ~16 bytes instead of 4 bytes for a raw `int`. This overhead is why primitive arrays beat boxed collections for large numeric data.

---

### Array (`int[]`)

```
Stack frame:
  [reference → ]──────────────────────────┐
                                           ▼
Heap:          [header | length=5 | 2 | 3 | 5 | 7 | 11]
                         ^─────── contiguous int values ──────^
```

- One heap allocation for the entire block.
- Elements stored **directly** (for primitives) — no pointers, no indirection.
- Random access: base address + index × element size = O(1) with excellent cache hits.
- Reference array (`String[]`) stores **references** (pointers) in the contiguous block; the actual `String` objects live elsewhere on the heap.

---

### ArrayList

```
Stack/Field:   [ArrayList reference →]──────────────────────────┐
                                                                  ▼
Heap:          [ArrayList object | size=3 | elementData ref →]──►[Object[] | ref0 | ref1 | ref2 | null | ... | null]
                                                                       ↓       ↓       ↓
                                                               [String "Alice"] [String "Bob"] [String "Carol"]
```

- The `ArrayList` object contains a `size` field and a reference to an internal `Object[]` (`elementData`).
- `elementData` is an array of **references**, not values. Each reference points to the actual object elsewhere on the heap.
- Two levels of indirection: `ArrayList` → `Object[]` → each element object.
- This is why primitives cost more — `ArrayList<Integer>` means each `int` is a heap-allocated `Integer` object.
- On resize, a new `Object[]` is allocated and `System.arraycopy` copies the references. The old array is garbage-collected.

---

### LinkedList

```
[LinkedList | size=3 | first → | last → ]
                        │                 │
                        ▼                 │
               [Node | prev=null | "Alice" | next →]
                                              │
                                              ▼
                                   [Node | prev ← | "Bob" | next →]
                                                                  │
                                                                  ▼
                                                       [Node | prev ← | "Carol" | next=null]
                                                         ▲
                                                         └──────────────── last ──────────┘
```

- Each node is a **separate heap object** with three fields: `prev`, `item`, `next`.
- Nodes are **not contiguous** — each lives at an arbitrary address on the heap.
- Accessing element `i` requires traversing from `first` (or `last` if closer) pointer by pointer — O(n) and cache-unfriendly because each pointer dereference likely misses the CPU cache.
- 3 heap objects per logical entry (the `Node` wrapper, plus the value object) — significant memory overhead vs ArrayList.

---

### HashMap

```
[HashMap | size | loadFactor | table ref →]──►[Node[] array — 16 buckets by default]
                                                  [0]: null
                                                  [1]: Node → Node → null  (collision chain)
                                                  [2]: null
                                                  [3]: Node
                                                  ...
                                                 [15]: null

Each Node:  [hash | key ref | value ref | next ref]
                     ↓             ↓
               [Key object]  [Value object]
```

- The internal `table` is an array of `Node` references (buckets).
- Each `Node` holds the key's hash, a reference to the key object, a reference to the value object, and a `next` pointer for the collision chain.
- **No collision:** bucket holds one `Node`, `next = null`.
- **Collision (few entries):** bucket holds a linked list of `Node`s — O(n) worst case per bucket.
- **Collision (8+ entries in one bucket, table size ≥ 64):** linked list converts to a `TreeNode` Red-Black tree — O(log n) per bucket.
- On resize (threshold = capacity × 0.75): new table double the size, all nodes rehashed. Expensive but amortized.

---

### HashSet

`HashSet` is literally a `HashMap` internally — it wraps a `HashMap<E, PRESENT>` where `PRESENT` is a static dummy `Object`. The set elements are the map's keys; the values are all the same dummy sentinel.

This means `HashSet` has the exact same memory layout as `HashMap`, with one dummy object shared across all entries.

---

### TreeMap / TreeSet

```
[TreeMap | size | root →]
              │
              ▼
         [Entry | key | value | left → | right → | parent → | color]
         /          \
   [Entry ...]    [Entry ...]
```

- Backed by a Red-Black tree — every entry is a heap-allocated `Entry` object with 5 fields (key, value, left, right, parent) plus a color bit.
- Significantly more memory per entry than `HashMap` (5 pointer fields vs 3 for HashMap's `Node`).
- Balanced: tree height is guaranteed ≤ 2 × log₂(n+1), so all operations are O(log n).
- No hashing — keys must be `Comparable` or a `Comparator` must be provided.

---

### Stack / Queue (ArrayDeque)

`ArrayDeque` is backed by a **circular resizable array**:

```
Internal array:  [null | "C" | "A" | "B" | null | null | null | null]
                          ▲                   ▲
                         head                tail
```

- `head` and `tail` are indices into the array.
- `addLast`: write at `tail`, advance `tail` (wraps around circularly).
- `addFirst`: move `head` back (wraps), write there.
- Both ends work in O(1) without shifting any elements.
- On resize: new array (2×), elements copied from `head` to `tail`.
- No node objects — just the array and two indices. Very cache-friendly and memory-efficient.

---

### PriorityQueue (Binary Heap)

```
Min-heap logical tree:        Internal array representation:
        1                     index: [0] [1] [2] [3] [4] [5] [6]
       / \                    value:  1   3   2   7   5   4   6
      3   2
     / \ / \
    7  5 4  6

Parent of index i  = (i - 1) / 2
Left child of i    = 2*i + 1
Right child of i   = 2*i + 2
```

- Stored as a plain `Object[]` — the tree is **implicit** in the array layout.
- No node objects, no pointers — just arithmetic to navigate parent/child relationships.
- Very cache-friendly because the array is contiguous.
- `offer`: append at end, then "bubble up" (swap with parent while smaller) — O(log n).
- `poll`: swap root with last element, remove last, then "sift down" — O(log n).
- `peek`: `array[0]` — O(1), always the minimum.

---

## 11. Java vs Python — Data Structure Comparison

**Q: How do Java's data structures map to Python's, and what are the key differences?**

### List: Java `ArrayList` vs Python `list`

Both are backed by a dynamic resizable array under the hood. Python's `list` is essentially the same idea as `ArrayList`.

| | Java `ArrayList` | Python `list` |
|---|---|---|
| Typed | Yes — generics (`List<String>`) | No — can hold mixed types |
| Primitives | Must box (`Integer`, not `int`) | Everything is an object — no boxing concept |
| Access | `list.get(i)` | `list[i]` |
| Append | `list.add(x)` | `list.append(x)` |
| Insert | `list.add(i, x)` — O(n) | `list.insert(i, x)` — O(n) |
| Remove | `list.remove(x)` or `list.remove(index)` | `list.remove(x)` or `del list[i]` |
| Slicing | `list.subList(from, to)` (view) | `list[1:4]` — new list |
| Comprehension | `stream().filter().collect()` | `[x for x in list if x > 0]` |
| Size | `list.size()` | `len(list)` |

Python's `list` is more flexible (mixed types, slicing syntax) but Java's generics provide compile-time type safety.

---

### Array: Java `int[]` vs Python `array` / `numpy`

Python doesn't have a true primitive array built into the language — its `list` always stores object references (like Java's `Object[]`). For true primitive arrays, Python has:
- `array.array('i', [1,2,3])` — typed array module, less common
- `numpy.ndarray` — the standard for numeric work

| | Java `int[]` | Python `list` | Python `numpy.ndarray` |
|---|---|---|---|
| Contiguous memory | Yes | No (stores references) | Yes |
| Typed / primitives | Yes | No | Yes |
| Fixed size | Yes | No | Yes |
| Math operations | Manual loops | Manual loops | Vectorized (`arr * 2`, `np.sum()`) |
| Use case | Fixed data, low-level | General sequences | Scientific computing, ML |

Java's primitive arrays are closer to `numpy` arrays than Python's `list` in terms of memory layout.

---

### HashMap: Java `HashMap` vs Python `dict`

Python's `dict` (since Python 3.7) is ordered by insertion — it's more like Java's `LinkedHashMap` than `HashMap`.

| | Java `HashMap` | Java `LinkedHashMap` | Python `dict` |
|---|---|---|---|
| Ordering | None | Insertion order | Insertion order (Python 3.7+) |
| Get/put | `map.get(k)` / `map.put(k,v)` | same | `d[k]` / `d[k] = v` |
| Default value | `map.getOrDefault(k, def)` | same | `d.get(k, default)` |
| Key check | `map.containsKey(k)` | same | `k in d` |
| Iterate keys | `map.keySet()` | same | `for k in d:` |
| Iterate pairs | `map.entrySet()` | same | `for k,v in d.items():` |
| Merge/update | `map.merge(k, v, fn)` | same | `d.update(other)` |
| Null/None keys | `HashMap` allows one | same | `None` is a valid key |

Python's `dict` is roughly equivalent to `LinkedHashMap` — ordered, flexible, dynamically typed keys and values.

---

### HashSet: Java `HashSet` vs Python `set`

These are very similar in concept and both backed by hash tables.

| | Java `HashSet` | Python `set` |
|---|---|---|
| Add | `set.add(x)` | `s.add(x)` |
| Remove | `set.remove(x)` | `s.remove(x)` (raises if absent) / `s.discard(x)` |
| Contains | `set.contains(x)` | `x in s` |
| Union | `set.addAll(other)` | `s \| other` or `s.union(other)` |
| Intersection | `set.retainAll(other)` | `s & other` or `s.intersection(other)` |
| Difference | `set.removeAll(other)` | `s - other` or `s.difference(other)` |
| Comprehension | No | `{x for x in iterable}` |
| Ordered version | `LinkedHashSet` / `TreeSet` | No built-in — use `dict` keys trick |

Python's `set` syntax is more concise. Java's `TreeSet` has no Python equivalent in the standard library — you'd need `sortedcontainers.SortedSet` from a third-party package.

---

### Stack: Java `ArrayDeque` vs Python `list` / `collections.deque`

Python doesn't have a dedicated stack class — a `list` with `append`/`pop` is idiomatic, or `collections.deque` for more efficiency:

| | Java `ArrayDeque` (as stack) | Python `list` | Python `collections.deque` |
|---|---|---|---|
| Push | `stack.push(x)` | `stack.append(x)` | `dq.appendleft(x)` |
| Pop | `stack.pop()` | `stack.pop()` | `dq.popleft()` |
| Peek | `stack.peek()` | `stack[-1]` | `dq[0]` |
| Thread-safe | No | No | No |

Python's `list.pop()` from the end is O(1) and works fine as a stack. `collections.deque` is better for queue/deque patterns.

---

### Queue: Java `ArrayDeque` vs Python `collections.deque` / `queue.Queue`

| | Java `ArrayDeque` (as queue) | Python `collections.deque` | Python `queue.Queue` |
|---|---|---|---|
| Enqueue | `queue.offer(x)` | `dq.append(x)` | `q.put(x)` |
| Dequeue | `queue.poll()` | `dq.popleft()` | `q.get()` |
| Thread-safe | No | No | **Yes** — built for producer-consumer |
| Blocking operations | `BlockingQueue` variants | No | Yes (`q.put()` blocks when full) |

For multi-threaded producer-consumer in Python, use `queue.Queue`. For Java, use `LinkedBlockingQueue` or `ArrayBlockingQueue`.

---

### PriorityQueue: Java `PriorityQueue` vs Python `heapq`

Both use a min-heap internally. Python's `heapq` is a module of functions that operate on a regular `list`; Java's `PriorityQueue` is an encapsulated class.

| | Java `PriorityQueue` | Python `heapq` |
|---|---|---|
| Push | `pq.offer(x)` | `heapq.heappush(lst, x)` |
| Pop min | `pq.poll()` | `heapq.heappop(lst)` |
| Peek min | `pq.peek()` | `lst[0]` |
| Max-heap | `new PriorityQueue<>(Comparator.reverseOrder())` | Negate values: push `-x`, pop and negate back |
| Build from list | `new PriorityQueue<>(list)` — O(n) | `heapq.heapify(lst)` — O(n) |
| Custom ordering | `Comparator` lambda | Tuple trick: `(priority, item)` |

```python
# Python min-heap
import heapq
heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 1)
heapq.heappush(heap, 3)
heapq.heappop(heap)   # 1

# Python max-heap (negate values)
heapq.heappush(heap, -5)
-heapq.heappop(heap)  # 5
```

---

### Summary: Java ↔ Python Equivalents

| Java | Python | Notes |
|---|---|---|
| `int[]`, `String[]` | `list` (no true primitive array) | Python list stores refs; numpy for numeric arrays |
| `ArrayList<T>` | `list` | Python list is ordered, dynamic, mixed-type |
| `LinkedList<T>` | `collections.deque` | Both doubly-linked |
| `HashMap<K,V>` | `dict` | Python dict preserves insertion order (3.7+) |
| `LinkedHashMap<K,V>` | `dict` | Equivalent since Python 3.7 |
| `TreeMap<K,V>` | `sortedcontainers.SortedDict` (third-party) | No built-in sorted map in Python |
| `HashSet<T>` | `set` | Near-identical semantics |
| `TreeSet<T>` | `sortedcontainers.SortedSet` (third-party) | No built-in |
| `LinkedHashSet<T>` | No exact equivalent | Use `dict` keys for ordered unique |
| `ArrayDeque` (stack) | `list` with `.append()`/`.pop()` | Python list end ops are O(1) |
| `ArrayDeque` (queue) | `collections.deque` | Both O(1) at both ends |
| `PriorityQueue<T>` | `heapq` module on a `list` | Both min-heaps; Python is functional API |
| `BlockingQueue` | `queue.Queue` | Python's Queue is thread-safe built-in |
| `ConcurrentHashMap` | No exact equivalent | Python GIL makes plain dict safer; use `threading.Lock` for composites |

---

## 12. Quick Reference Cheat Sheet

**Q: Given a requirement, what data structure should I reach for?**

| Requirement | Data Structure | Why |
|---|---|---|
| Ordered list, index access, duplicates OK | `ArrayList` | O(1) index, dynamic sizing |
| Ordered list, frequent insert/delete at ends | `ArrayDeque` | O(1) at both ends |
| Fixed size, primitives, max performance | `int[]` / `long[]` etc. | No boxing, contiguous memory |
| Fast lookup by key | `HashMap` | O(1) avg get/put |
| Fast lookup, maintain insertion order | `LinkedHashMap` | O(1) avg + ordered iteration |
| Fast lookup, sorted keys, range queries | `TreeMap` | O(log n), sorted |
| Unique elements, fast membership test | `HashSet` | O(1) contains |
| Unique elements, insertion order | `LinkedHashSet` | O(1) contains + ordered |
| Unique elements, sorted | `TreeSet` | O(log n), sorted iteration |
| LIFO — undo, DFS, bracket matching | `ArrayDeque` as stack | O(1) push/pop |
| FIFO — task queue, BFS | `ArrayDeque` as queue | O(1) offer/poll |
| Priority ordering — always get min/max | `PriorityQueue` | O(log n) insert, O(1) peek |
| Both ends efficiently (sliding window) | `ArrayDeque` as deque | O(1) at head and tail |
| Sorted data, O(log n) ops, range queries | `TreeMap` / `TreeSet` | Red-Black tree |
| Thread-safe key-value store | `ConcurrentHashMap` | Segment locking, safe concurrent access |
| Count frequencies | `HashMap<T, Integer>` | Key = element, value = count |
| Hierarchical / parent-child data | Tree (custom or `TreeMap`) | Natural recursive structure |

**Q: Given a requirement, what data structure should I reach for?**

| Requirement | Data Structure | Why |
|---|---|---|
| Ordered list, index access, duplicates OK | `ArrayList` | O(1) index, dynamic sizing |
| Ordered list, frequent insert/delete at ends | `ArrayDeque` | O(1) at both ends |
| Fixed size, primitives, max performance | `int[]` / `long[]` etc. | No boxing, contiguous memory |
| Fast lookup by key | `HashMap` | O(1) avg get/put |
| Fast lookup, maintain insertion order | `LinkedHashMap` | O(1) avg + ordered iteration |
| Fast lookup, sorted keys, range queries | `TreeMap` | O(log n), sorted |
| Unique elements, fast membership test | `HashSet` | O(1) contains |
| Unique elements, insertion order | `LinkedHashSet` | O(1) contains + ordered |
| Unique elements, sorted | `TreeSet` | O(log n), sorted iteration |
| LIFO — undo, DFS, bracket matching | `ArrayDeque` as stack | O(1) push/pop |
| FIFO — task queue, BFS | `ArrayDeque` as queue | O(1) offer/poll |
| Priority ordering — always get min/max | `PriorityQueue` | O(log n) insert, O(1) peek |
| Both ends efficiently (sliding window) | `ArrayDeque` as deque | O(1) at head and tail |
| Sorted data, O(log n) ops, range queries | `TreeMap` / `TreeSet` | Red-Black tree |
| Thread-safe key-value store | `ConcurrentHashMap` | Segment locking, safe concurrent access |
| Count frequencies | `HashMap<T, Integer>` | Key = element, value = count |
| Hierarchical / parent-child data | Tree (custom or `TreeMap`) | Natural recursive structure |
