# DSA Interview Handbook
### Data Structures & Algorithms — Concepts, Complexity, and Coding Patterns

Code examples are in Java for concreteness, but the concepts and complexity analysis are language-agnostic.

---

## Table of Contents

**Part 1 — Fundamentals**
1. Big O Notation & Complexity Analysis

**Part 2 — Linear Data Structures**
2. Arrays
3. Strings
4. Linked Lists
5. Stacks
6. Queues & Deques

**Part 3 — Non-Linear Data Structures**
7. Hash Tables
8. Trees & Binary Search Trees
9. Balanced Trees & Tries
10. Heaps & Priority Queues
11. Graphs — Representation & Traversal (BFS/DFS)
12. Disjoint Set (Union-Find)
13. Segment Trees & Fenwick Trees (BIT)

**Part 4 — Algorithm Design Paradigms**
14. Sorting Algorithms
15. Searching Algorithms
16. Recursion & Backtracking
17. Divide and Conquer
18. Greedy Algorithms
19. Dynamic Programming
20. Graph Algorithms — Shortest Path, MST, Topological Sort
21. Bit Manipulation
22. Two Pointers & Sliding Window

**Part 5 — Most-Asked Coding Patterns**
23. Pattern Catalog with Solutions

---

# PART 1: FUNDAMENTALS

## 1. Big O Notation & Complexity Analysis

**Q1: What does Big O notation actually describe, and why do we ignore constants and lower-order terms?**

A: Big O describes the **upper-bound growth rate** of an algorithm's time or space requirements as input size (`n`) approaches infinity — it answers "how does this scale," not "how fast is this in absolute milliseconds." We drop constants and lower-order terms (`O(2n + 100)` → `O(n)`) because for large enough `n`, the **dominant term** overwhelmingly determines behavior — an `O(n)` algorithm eventually beats an `O(n²)` one regardless of constant factors, so those constants aren't meaningful for comparing algorithms **asymptotically**. (They still matter in practice for small, fixed-size inputs — a common follow-up interviewers ask.)

**Q2: What's the difference between Big O, Big Ω (Omega), and Big Θ (Theta)?**

A: **Big O** — worst-case **upper bound** ("this algorithm never does worse than..."). **Big Ω** — best-case **lower bound** ("this algorithm never does better than..."). **Big Θ** — a **tight bound**, meaning the upper and lower bounds are the same order — the algorithm's growth rate is *exactly* this, not just bounded by it. In casual interview usage, "Big O" is often used loosely to mean "the typical/expected complexity," but a precise answer distinguishes best/average/worst case explicitly when they differ (e.g., Quicksort: average Θ(n log n), worst-case O(n²)).

**Q3: Rank these complexities from fastest to slowest growth: O(n log n), O(2ⁿ), O(1), O(log n), O(n), O(n²), O(n!).**

A: `O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)`. A useful intuition table:
| Complexity | Example |
|---|---|
| O(1) | Array index access, hash map get/put (average) |
| O(log n) | Binary search, balanced BST operations |
| O(n) | Single loop, linear search |
| O(n log n) | Merge sort, heap sort, efficient sorting |
| O(n²) | Nested loops, bubble/insertion sort |
| O(2ⁿ) | Naive recursive Fibonacci, generating all subsets |
| O(n!) | Generating all permutations, brute-force TSP |

**Q4: How do you analyze the time complexity of recursive algorithms (recurrence relations), and what is the Master Theorem?**

A: A recursive algorithm's complexity is expressed as a **recurrence relation** — e.g., merge sort: `T(n) = 2T(n/2) + O(n)` (two recursive calls on half the input, plus O(n) work to merge). The **Master Theorem** gives a shortcut for recurrences of the form `T(n) = aT(n/b) + f(n)`:
- If `f(n) = O(n^(log_b(a) - ε))` → `T(n) = Θ(n^log_b(a))` (recursive work dominates).
- If `f(n) = Θ(n^log_b(a))` → `T(n) = Θ(n^log_b(a) · log n)` (balanced — this is merge sort's case: a=2, b=2, f(n)=n, giving Θ(n log n)).
- If `f(n) = Ω(n^(log_b(a) + ε))` → `T(n) = Θ(f(n))` (the "combine" work dominates).

**Q5: What's the difference between time complexity and space complexity, and what does "auxiliary space" mean?**

A: Time complexity measures how runtime scales with input size; space complexity measures how **memory usage** scales with input size. **Auxiliary space** specifically refers to **extra** space used by the algorithm **beyond the input itself** (e.g., a temporary array, a recursion call stack) — this distinction matters because an "in-place" sort like quicksort still technically uses O(n) total space (the input array), but its **auxiliary** space is O(log n) (just the recursion stack), which is the number usually quoted when comparing in-place vs non-in-place algorithms.

---

# PART 2: LINEAR DATA STRUCTURES

## 2. Arrays

**Q1: What are the time complexities of array operations, and why is insertion/deletion O(n) in the middle but O(1) at the end?**

A:
| Operation | Complexity | Why |
|---|---|---|
| Access by index | O(1) | Direct memory address calculation: `base + index × elementSize` |
| Search (unsorted) | O(n) | Must check elements one by one |
| Search (sorted) | O(log n) | Binary search applicable |
| Insert/delete at end | O(1) amortized | No shifting needed (assuming capacity available) |
| Insert/delete at start/middle | O(n) | All subsequent elements must shift to preserve contiguity |

Arrays store elements in **contiguous memory**, which is what makes O(1) indexed access possible (direct address math) — but that same contiguity requirement means inserting/removing anywhere except the end forces shifting every subsequent element to maintain that contiguous layout.

**Q2: What is "amortized O(1)" for dynamic array (e.g., `ArrayList`/`Vector`) appends, and how does the doubling strategy achieve it?**

A: A dynamic array starts with some capacity; when full, it allocates a **new, larger array** (typically **double** the size) and copies all existing elements over — that single resize operation is O(n). But because resizes happen at exponentially increasing intervals (after 1, 2, 4, 8, 16... appends), the **total** cost of all resizes across `n` appends sums to O(n) (a geometric series), which averaged (amortized) over `n` individual append operations gives O(1) **per** append, even though any single append might occasionally trigger an O(n) resize.

**Q3: How do you rotate an array by k positions in-place with O(1) extra space?**
```java
public static void rotate(int[] nums, int k) {
    k %= nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}
private static void reverse(int[] arr, int i, int j) {
    while (i < j) { int t = arr[i]; arr[i++] = arr[j]; arr[j--] = t; }
}
```
The "reverse three times" trick: reversing the whole array, then reversing each of the two resulting segments individually, produces a rotation — O(n) time, O(1) space, versus a naive O(n·k) shift-one-at-a-time approach.

**Q4: What's the difference between a static array and a dynamic array, and what languages/structures represent each?**

A: A **static array** has a **fixed size** determined at creation (Java's `int[]`, C's `int arr[10]`) — cannot grow/shrink; attempting to exceed its bounds is undefined behavior or an exception. A **dynamic array** (Java's `ArrayList`, C++'s `std::vector`, Python's `list`, JS arrays) automatically resizes as needed, built internally **on top of** a static array plus the doubling-resize logic described above.

---

## 3. Strings

**Q1: Why are strings often described as "immutable arrays of characters," and what's the practical performance implication?**

A: In many languages (Java, Python, JavaScript), strings are **immutable** — every "modification" (concatenation, replacement) actually creates a **new** string object rather than changing the existing one in place. The practical implication: building a string via repeated concatenation in a loop (`result += nextChunk`) is **O(n²)** overall (each concatenation copies the growing string so far), whereas using a mutable builder (`StringBuilder` in Java, list-and-`join` in Python) accumulates pieces first and joins once, achieving O(n).
```java
// BAD — O(n^2)
String result = "";
for (String s : parts) result += s;

// GOOD — O(n)
StringBuilder sb = new StringBuilder();
for (String s : parts) sb.append(s);
String result = sb.toString();
```

**Q2: How do you check if two strings are anagrams, and what's the optimal complexity?**
```java
public static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] counts = new int[26];
    for (int i = 0; i < a.length(); i++) {
        counts[a.charAt(i) - 'a']++;
        counts[b.charAt(i) - 'a']--;
    }
    for (int c : counts) if (c != 0) return false;
    return true;
}
```
O(n) time, O(1) space (the count array is fixed-size, independent of input length) — better than the naive O(n log n) approach of sorting both strings and comparing.

**Q3: Explain the Knuth-Morris-Pratt (KMP) algorithm's core idea for substring search, and why it beats naive O(n·m) search.**

A: Naive substring search re-checks characters redundantly after a partial match fails — e.g., matching "AAAAB" against "AAAAAAAAAB" restarts from scratch at each failed position, wasting work re-comparing characters already known to match. **KMP** precomputes a **"failure function" / LPS array** (Longest Proper Prefix which is also Suffix) for the pattern, which tells the algorithm exactly how far it can **safely skip ahead** without re-checking characters known to still match, based on the pattern's own internal structure — achieving O(n + m) time (linear in text length + pattern length) instead of naive search's O(n·m) worst case.

**Q4: What's the sliding window technique, and how does it apply to a "longest substring without repeating characters" problem?**
```java
public static int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        while (window.contains(s.charAt(right))) {
            window.remove(s.charAt(left++));
        }
        window.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```
The sliding window maintains a **contiguous, valid range** `[left, right]`, expanding `right` to include new elements and contracting `left` only when the window becomes invalid (here, a duplicate character) — achieving O(n) since each character is added and removed from the window at most once, versus a brute-force O(n²)/O(n³) check of every substring.

---

## 4. Linked Lists

**Q1: Compare singly linked lists, doubly linked lists, and circular linked lists.**

A: **Singly linked** — each node points only to the **next** node; traversal is one-directional; O(1) insertion at the head, O(n) to reach/insert at an arbitrary position (no backward traversal possible). **Doubly linked** — each node has both `next` and `prev` pointers, enabling **bidirectional traversal** and O(1) removal of a **known** node (no need to traverse from the head to find its predecessor) — at the cost of extra memory per node for the second pointer. **Circular linked list** — the last node points back to the **first** (instead of `null`), useful for round-robin scheduling or any cyclic-traversal use case; can be singly or doubly linked.

**Q2: Array vs Linked List — summarize the core tradeoff.**

A: Arrays: O(1) random access, poor insertion/deletion in the middle (O(n) shifting), better cache locality (contiguous memory improves CPU cache hit rates in practice, even though this doesn't show up in Big-O). Linked Lists: O(n) access (must traverse from the head), O(1) insertion/deletion **once positioned** at the right node, no need to pre-allocate/resize (grows one node at a time), but each node has memory overhead (the pointer(s)) and poor cache locality (nodes scattered across memory).

**Q3: Reverse a singly linked list — iterative and recursive.**
```java
// Iterative — O(n) time, O(1) space
public static Node reverseIterative(Node head) {
    Node prev = null;
    while (head != null) {
        Node next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}

// Recursive — O(n) time, O(n) space (call stack)
public static Node reverseRecursive(Node head) {
    if (head == null || head.next == null) return head;
    Node newHead = reverseRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

**Q4: Detect and find the start of a cycle in a linked list (Floyd's Cycle Detection).**
```java
public static Node detectCycleStart(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            // cycle detected — find the entry point
            Node ptr = head;
            while (ptr != slow) { ptr = ptr.next; slow = slow.next; }
            return ptr;
        }
    }
    return null; // no cycle
}
```
The mathematical proof: once slow and fast meet inside the cycle, resetting one pointer to `head` and advancing both at the same speed causes them to meet **exactly at the cycle's start** — a classic, frequently-asked derivation interviewers may ask you to explain, not just implement.

---

## 5. Stacks

**Q1: What is a stack, and what are its core operations with their complexities?**

A: A **LIFO** (Last-In-First-Out) structure — the most recently added element is the first removed. Core operations, all **O(1)**: `push` (add to top), `pop` (remove from top), `peek`/`top` (view top without removing), `isEmpty`. Typically implemented via a dynamic array (with amortized O(1) push, as covered in Section 2) or a linked list (O(1) push/pop at the head).

**Q2: What are the classic use cases for a stack, and why is each a natural fit?**

A: **Function call management** (the "call stack" — each function call pushes a new frame, returns pop it, naturally LIFO since the most recently called function must return first). **Undo/redo functionality** (each action pushed; undo pops the most recent). **Expression evaluation/parsing** (matching parentheses/brackets, converting infix to postfix, evaluating postfix expressions). **Backtracking algorithms** (explicitly or via the recursive call stack — see Section 16) — exploring a path and being able to "pop back" to the last decision point when a path fails.

**Q3: Check for balanced parentheses using a stack.**
```java
public static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) return false;
        }
    }
    return stack.isEmpty();
}
```

**Q4: Implement a stack that supports `getMin()` in O(1), not just O(n).**
```java
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>(); // tracks running minimum

    public void push(int val) {
        stack.push(val);
        minStack.push(minStack.isEmpty() ? val : Math.min(val, minStack.peek()));
    }
    public void pop() { stack.pop(); minStack.pop(); }
    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); } // O(1) — a very common interview question
}
```
The trick: maintain a **parallel stack** that tracks the minimum-so-far at each push, so popping automatically "restores" the correct previous minimum too — trading O(n) extra space for O(1) `getMin()`.

---

## 6. Queues & Deques

**Q1: What is a queue, and how does it differ from a stack?**

A: A **FIFO** (First-In-First-Out) structure — elements are removed in the same order they were added, unlike a stack's LIFO order. Core operations, all O(1) with a proper implementation: `enqueue` (add to the back), `dequeue` (remove from the front), `peek` (view front).

**Q2: Why is a naive array-based queue (shifting elements on dequeue) inefficient, and how does a circular buffer fix it?**

A: A naive implementation that removes from index 0 of a plain array requires **shifting every remaining element** left by one — O(n) per dequeue. A **circular buffer** (circular queue) uses a fixed-size array with `front` and `rear` indices that **wrap around** (`(index + 1) % capacity`) instead of shifting elements — both enqueue and dequeue become true O(1), at the cost of a fixed maximum capacity (or needing resize logic similar to dynamic arrays if unbounded growth is required).

**Q3: What is a Deque (double-ended queue), and what's a common algorithmic use case?**

A: A **Deque** supports O(1) insertion/removal at **both ends** (front and back) — effectively a generalization that can act as either a stack or a queue. A classic use case: the **sliding window maximum** problem — maintaining a deque of indices in **decreasing order of value**, so the front always holds the current window's maximum:
```java
public static int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // stores INDICES, values decreasing
    int[] result = new int[nums.length - k + 1];
    for (int i = 0; i < nums.length; i++) {
        if (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // out of window
        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) deque.pollLast(); // maintain decreasing order
        deque.offerLast(i);
        if (i >= k - 1) result[i - k + 1] = nums[deque.peekFirst()];
    }
    return result;
}
```
O(n) overall — each index is added and removed from the deque at most once, despite the nested-looking `while` loop.

**Q4: How does a Priority Queue differ from a regular queue, and what's it typically implemented with?**

A: A regular queue dequeues strictly in **insertion order** (FIFO). A **Priority Queue** dequeues the element with the **highest (or lowest) priority** first, regardless of insertion order — typically implemented with a **heap** (Section 10), giving O(log n) insertion and O(log n) removal-of-highest-priority, versus a naive sorted-array approach's O(n) insertion. Used for algorithms like Dijkstra's shortest path and task scheduling by priority.

---

# PART 3: NON-LINEAR DATA STRUCTURES

## 7. Hash Tables

**Q1: How does a hash table achieve average O(1) lookup, and what causes collisions?**

A: A **hash function** converts a key into an array index (bucket location) — `index = hash(key) % capacity`. Since the space of possible keys is typically far larger than the number of buckets, **collisions** (two different keys hashing to the same bucket) are mathematically inevitable (pigeonhole principle) — a good hash function minimizes their **frequency** and distributes keys **uniformly**, but can't eliminate them entirely.

**Q2: Compare the two main collision resolution strategies: chaining vs open addressing.**

A: **Separate chaining** — each bucket holds a **linked list** (or, as in Java 8+ `HashMap`, a tree once a bucket gets large enough) of all entries hashing to that index; lookups within a bucket degrade to O(k) for k colliding entries, but the table itself never "fills up" in the same sense. **Open addressing** — on collision, probe for the **next available slot** within the same array using a defined sequence (linear probing: `index+1, index+2...`; quadratic probing; double hashing) — no extra memory for links, better cache locality, but requires careful handling of deletions (naive removal can break probe chains, typically solved with "tombstone" markers) and performance degrades sharply as the table fills up (load factor approaching 1).

**Q3: What is load factor, and why does resizing matter for maintaining O(1) average performance?**

A: Load factor = `number of entries / number of buckets`. As it grows, collision chains get longer (chaining) or probe sequences get longer (open addressing), degrading average-case performance toward O(n). Hash tables **resize** (typically doubling capacity) once load factor crosses a threshold (Java's `HashMap` default is 0.75), rehashing all existing entries into the larger table — an O(n) operation, but infrequent enough (like dynamic array resizing) that it only contributes **amortized O(1)** to the average per-operation cost.

**Q4: What properties must a good hash function have?**

A: **Deterministic** (same key always produces the same hash), **uniform distribution** (spreads keys evenly across buckets to minimize collisions), **fast to compute** (shouldn't itself become the bottleneck), and ideally exhibits the **avalanche effect** (a tiny change in the key produces a drastically different hash, avoiding clustering for similar keys). It does **not** need to be cryptographically secure/irreversible (that's a different, stricter requirement for cryptographic hash functions like SHA-256) — a hash table's hash function just needs good distribution properties.

**Q5: Two Sum — the canonical hash table interview question.**
```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value -> index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) return new int[]{seen.get(complement), i};
        seen.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution");
}
```
O(n) time, O(n) space — the canonical example of trading space for time: a naive nested-loop check is O(n²) with O(1) space; the hash-table version does a single pass by remembering what it's seen.

---

## 8. Trees & Binary Search Trees

**Q1: Define the core tree terminology: root, leaf, height, depth, and the difference between a full/complete/perfect binary tree.**

A: **Root** — the topmost node (no parent). **Leaf** — a node with no children. **Height** of a node — the longest path from it down to a leaf; **depth** of a node — the distance from the root down to it (root has depth 0). **Full binary tree** — every node has either 0 or 2 children (never exactly 1). **Complete binary tree** — all levels are fully filled except possibly the last, which is filled left-to-right (this is the shape a heap must maintain, Section 10). **Perfect binary tree** — all internal nodes have 2 children **and** all leaves are at the same depth (a stricter special case of both full and complete).

**Q2: What is a Binary Search Tree (BST), and what invariant must every node satisfy?**

A: A BST is a binary tree where, for **every** node: all values in its **left** subtree are **less than** the node's value, and all values in its **right** subtree are **greater than** it (assuming no duplicates; conventions vary on where equal values go). This invariant is what enables O(log n) search/insert/delete **on a balanced tree** — at each node, you can discard an entire half of the remaining subtree based on a single comparison, exactly like binary search on a sorted array.

**Q3: Implement BST search, insert, and explain why worst-case complexity is O(n), not O(log n).**
```java
class Node { int val; Node left, right; Node(int val) { this.val = val; } }

public static Node insert(Node root, int val) {
    if (root == null) return new Node(val);
    if (val < root.val) root.left = insert(root.left, val);
    else if (val > root.val) root.right = insert(root.right, val);
    return root;
}
public static boolean search(Node root, int val) {
    if (root == null) return false;
    if (val == root.val) return true;
    return val < root.val ? search(root.left, val) : search(root.right, val);
}
```
O(log n) is the **balanced-tree** average/best case — but a BST built from **already-sorted input** (inserting 1, 2, 3, 4, 5 in order) degenerates into essentially a linked list (every node has only a right child), giving O(n) worst-case search/insert/delete. This is precisely the motivation for **self-balancing trees** (Section 9).

**Q4: Explain the three depth-first traversal orders (in-order, pre-order, post-order) and one breadth-first (level-order), with what each is typically used for.**
```java
void inorder(Node n)   { if (n==null) return; inorder(n.left); visit(n); inorder(n.right); }   // Left, Root, Right
void preorder(Node n)  { if (n==null) return; visit(n); preorder(n.left); preorder(n.right); } // Root, Left, Right
void postorder(Node n) { if (n==null) return; postorder(n.left); postorder(n.right); visit(n); } // Left, Right, Root
```
- **In-order** on a BST visits nodes in **sorted ascending order** — the most distinguishing, frequently-tested fact about BST in-order traversal.
- **Pre-order** is useful for **copying/serializing** a tree (root written first, so you can reconstruct top-down).
- **Post-order** is useful when children must be processed **before** the parent (e.g., deleting a tree bottom-up, or computing a value that depends on subtree results, like folder sizes).
- **Level-order** (BFS, using a queue) visits nodes level by level — used whenever "shortest path"/"minimum steps" thinking applies to a tree, or when you specifically need level-by-level output.

**Q5: Find the Lowest Common Ancestor (LCA) of two nodes in a BST.**
```java
public static Node lcaBST(Node root, int p, int q) {
    if (p < root.val && q < root.val) return lcaBST(root.left, p, q);
    if (p > root.val && q > root.val) return lcaBST(root.right, p, q);
    return root; // p and q diverge here (or one equals root) — this IS the LCA
}
```
Exploits the BST property directly: O(h) where h is tree height (O(log n) balanced, O(n) worst case) — much simpler than the general-binary-tree LCA algorithm (which needs to search both subtrees explicitly since there's no ordering to exploit).

---

## 9. Balanced Trees & Tries

**Q1: What problem do self-balancing trees (AVL, Red-Black) solve, and what's the core mechanism?**

A: They solve the BST worst-case-degeneration problem (Section 8, Q3) by **automatically restructuring** the tree after insertions/deletions to keep it roughly balanced, guaranteeing O(log n) height (and thus O(log n) operations) **even in the worst case** — not just on average. The core mechanism in both is **rotations** (local restructuring operations that rebalance a subtree in O(1) while preserving the BST property) triggered when an imbalance is detected.

**Q2: AVL trees vs Red-Black trees — what's the practical tradeoff?**

A: **AVL trees** enforce a **stricter** balance invariant (heights of left/right subtrees of any node differ by at most 1), making them **more rigidly balanced** — faster lookups on average, but more frequent/expensive rebalancing rotations on insert/delete. **Red-Black trees** use a looser balance invariant (via node coloring rules ensuring no path is more than roughly 2x longer than any other), tolerating slightly less-balanced trees in exchange for **fewer rotations** on average during insertion/deletion. Practical rule of thumb: AVL trees are preferred for **lookup-heavy** workloads (e.g., a read-mostly index); Red-Black trees are preferred for **insertion/deletion-heavy** workloads — which is why Java's `TreeMap`/`TreeSet` and C++'s `std::map` use Red-Black trees internally.

**Q3: What is a Trie (prefix tree), and what problem is it specifically good at solving?**

A: A Trie is a tree where each node represents a **single character**, and paths from the root spell out strings/prefixes — nodes are shared among words with common prefixes.
```java
class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isEndOfWord;
}
class Trie {
    private TrieNode root = new TrieNode();
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) node = node.children.computeIfAbsent(c, k -> new TrieNode());
        node.isEndOfWord = true;
    }
    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEndOfWord;
    }
    public boolean startsWith(String prefix) { return find(prefix) != null; }
    private TrieNode find(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            node = node.children.get(c);
            if (node == null) return null;
        }
        return node;
    }
}
```
Tries excel at **prefix-based operations** — autocomplete, spell-checkers, IP routing tables — with search/insert complexity O(L) where L is the string's length, **independent of how many words are stored** (unlike a hash set of strings, which is O(L) average too, but doesn't natively support efficient "give me all words with this prefix" queries — a Trie does, in O(prefix length + number of matches)).

---

## 10. Heaps & Priority Queues

**Q1: What is a heap, and what invariant does a min-heap vs max-heap maintain?**

A: A heap is a **complete binary tree** (Section 8, Q1) satisfying the **heap property**: in a **min-heap**, every parent is **≤** its children (so the minimum element is always at the root); in a **max-heap**, every parent is **≥** its children (maximum at the root). Note this is a **weaker** ordering than a BST — siblings and cousins have no defined relationship, only the parent-child relationship is constrained, which is what makes heap operations simpler/faster than full BST rebalancing.

**Q2: How is a heap typically implemented using a plain array, and how do you compute a node's children/parent indices?**

A: Because a heap is always a **complete** tree, it can be stored compactly in an array with no pointers needed at all:
```
For a node at index i (0-indexed):
  left child  = 2*i + 1
  right child = 2*i + 2
  parent      = (i - 1) / 2
```
This array-based implicit representation is why heaps are memory-efficient compared to pointer-based trees, and why `java.util.PriorityQueue` is backed by a plain array internally.

**Q3: Explain `heapify` (sift-down) and how it achieves O(log n) insertion/extraction.**

A: **Insertion**: add the new element at the **end** of the array (maintaining completeness), then **sift up** — repeatedly swap it with its parent while it violates the heap property, which takes at most O(log n) swaps (the tree's height). **Extraction (of the min/max)**: remove the root, move the **last** element into the root position (maintaining completeness), then **sift down** — repeatedly swap it with its smaller (min-heap) child while it violates the heap property, again O(log n) swaps max. Building a heap from an unsorted array of n elements is actually O(n), not O(n log n) — a classic "trick" fact: calling sift-down from the middle of the array upward has a tighter combined bound than naively inserting n elements one at a time.

**Q4: Find the K largest elements in an array — classic heap use case.**
```java
public static List<Integer> kLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap by default in Java
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) minHeap.poll(); // evict the smallest, keeping only the top k so far
    }
    return new ArrayList<>(minHeap);
}
```
O(n log k) — better than sorting the whole array (O(n log n)) when k is small relative to n. The counterintuitive trick many candidates miss at first: use a **min-heap** (not a max-heap) of size k to track the k **largest** elements, since you always want to evict the current smallest of your "top k so far" candidates.

---

## 11. Graphs — Representation & Traversal

**Q1: Compare adjacency matrix vs adjacency list graph representations.**

A: **Adjacency matrix** — a `V × V` 2D array where `matrix[i][j] = 1` (or a weight) if an edge exists — O(1) edge-existence check, but O(V²) space regardless of how sparse the graph is, and O(V) to enumerate a single node's neighbors. **Adjacency list** — each vertex stores a list of its neighbors — O(V + E) space (efficient for sparse graphs, the common case in practice), O(degree) to enumerate a node's neighbors, but O(degree) (not O(1)) to check if a specific edge exists. Adjacency lists are the default choice for most graph algorithms/interview problems unless the graph is known to be dense or you specifically need O(1) edge lookups.

**Q2: Explain BFS (Breadth-First Search) — algorithm, complexity, and canonical use case.**
```java
public static List<Integer> bfs(Map<Integer, List<Integer>> graph, int start) {
    List<Integer> order = new ArrayList<>();
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(start);
    visited.add(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
    return order;
}
```
BFS explores **level by level** using a queue — O(V + E) time. Canonical use case: finding the **shortest path in an unweighted graph** (BFS is guaranteed to reach any node via the fewest possible edges first, since it exhausts all nodes at distance `d` before moving to distance `d+1`).

**Q3: Explain DFS (Depth-First Search) — algorithm, complexity, and canonical use cases.**
```java
public static void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited, List<Integer> order) {
    visited.add(node);
    order.add(node);
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (!visited.contains(neighbor)) dfs(graph, neighbor, visited, order);
    }
}
```
DFS explores **as deep as possible** along one branch before backtracking — O(V + E) time, using either explicit recursion (implicitly using the call stack) or an explicit stack. Canonical use cases: **cycle detection**, **topological sorting** (Section 20), **connected components**, and **backtracking-style exploration** (maze solving, generating all paths) where you need to fully commit to a path before trying alternatives.

**Q4: How do you detect a cycle in a directed graph vs an undirected graph — why does the approach differ?**

A: In an **undirected** graph, a cycle exists if, during DFS, you reach an already-visited node that **isn't the immediate parent** you came from (since undirected edges are bidirectional, seeing the parent again is expected, not a cycle). In a **directed** graph, you need to track nodes currently **on the current recursion path** (a separate "in-progress" set, distinct from "fully visited") — a cycle exists if you reach a node that's still on the current path (a "back edge" to an ancestor in the DFS tree), whereas reaching a node that was fully visited **and finished** via a different branch entirely is fine (that's just a shared descendant, not a cycle) — this distinction (visited-and-done vs currently-in-progress) is exactly what topological sort's cycle-detection relies on too.

---

## 12. Disjoint Set (Union-Find)

**Q1: What problem does the Union-Find (Disjoint Set Union, DSU) data structure solve?**

A: It efficiently tracks a partition of elements into **disjoint (non-overlapping) sets**, supporting two core operations: `find(x)` (which set does x belong to — represented by a "representative"/root element) and `union(x, y)` (merge the sets containing x and y). Classic uses: detecting cycles in an undirected graph (union-find can detect "these two nodes are already in the same connected component" before adding an edge, in near-O(1)), Kruskal's MST algorithm (Section 20), and network connectivity queries.

**Q2: Implement Union-Find with the two key optimizations: path compression and union by rank/size.**
```java
class UnionFind {
    int[] parent, rank;
    UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i; // each element starts as its own set
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // PATH COMPRESSION — flatten the tree on the way up
        return parent[x];
    }
    void union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return;
        // UNION BY RANK — attach the smaller/shallower tree under the larger/deeper one
        if (rank[rootX] < rank[rootY]) { parent[rootX] = rootY; }
        else if (rank[rootX] > rank[rootY]) { parent[rootY] = rootX; }
        else { parent[rootY] = rootX; rank[rootX]++; }
    }
}
```
**Path compression** — during `find`, make every visited node point **directly** to the root, flattening the tree for all future lookups. **Union by rank** — always attach the shorter tree under the taller one's root, avoiding unnecessarily deep chains. **Together**, these two optimizations give a near-constant amortized complexity per operation — **O(α(n))**, where α is the **inverse Ackermann function**, which grows so slowly it's effectively ≤ 4-5 for any input size that could ever exist in practice — a fact interviewers sometimes ask about specifically ("is Union-Find really O(1)?").

---

## 13. Segment Trees & Fenwick Trees (BIT)

**Q1: What problem do Segment Trees and Fenwick Trees both solve, and why isn't a simple prefix-sum array always enough?**

A: Both support efficient **range queries** (sum, min, max over a range `[l, r]`) **and** efficient **point updates** (changing a single element) on an array. A simple **prefix-sum array** gives O(1) range-sum queries, but a single element update requires recomputing the entire prefix-sum array from that point forward — O(n) per update. Segment Trees and Fenwick Trees both achieve **O(log n) for both** range queries and point updates, trading a bit of query speed (vs prefix sum's O(1)) for dramatically better update speed (vs prefix sum's O(n)).

**Q2: What's the structural difference between a Segment Tree and a Fenwick Tree (Binary Indexed Tree)?**

A: A **Segment Tree** is an explicit **binary tree** (usually array-backed) where each node represents a **range/segment** of the original array, with leaves as individual elements and internal nodes storing the aggregate (sum/min/max) of their children's ranges — more general-purpose, supporting a wider variety of range operations (range updates with lazy propagation, range min/max, etc.), at the cost of more memory (~4n) and more complex implementation. A **Fenwick Tree (BIT)** is a cleverer, more compact structure exploiting the **binary representation of indices** to implicitly encode partial sums in a single array of size n+1, using bit manipulation (`index & (-index)`) to navigate — simpler to implement, less memory, but fundamentally more limited (primarily designed for prefix-sum-style queries; range-min/max isn't naturally supported the way it is with a segment tree).

**Q3: Implement a basic Fenwick Tree for prefix sums.**
```java
class FenwickTree {
    int[] tree;
    int n;
    FenwickTree(int n) { this.n = n; tree = new int[n + 1]; }

    void update(int i, int delta) { // add `delta` to element at index i (1-indexed)
        for (; i <= n; i += i & (-i)) tree[i] += delta;
    }
    int prefixSum(int i) { // sum of elements [1..i]
        int sum = 0;
        for (; i > 0; i -= i & (-i)) sum += tree[i];
        return sum;
    }
    int rangeSum(int l, int r) { return prefixSum(r) - prefixSum(l - 1); }
}
```
`i & (-i)` isolates the **lowest set bit** of `i`, which is the mechanism that lets each `update`/`prefixSum` call touch only O(log n) array positions rather than needing an explicit tree structure with pointers.

---

# PART 4: ALGORITHM DESIGN PARADIGMS

## 14. Sorting Algorithms

**Q1: Compare the major sorting algorithms by time complexity, space complexity, and stability.**

| Algorithm | Best | Average | Worst | Space | Stable? |
|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(n+k) | Yes |

**Q2: What does "stable" mean for a sorting algorithm, and when does it actually matter?**

A: A sort is **stable** if elements with **equal keys retain their original relative order** after sorting. It matters when you sort by one key but want a **secondary, previously-established order preserved** for ties — e.g., sorting a list of employees by department, where you'd previously sorted them by name: a stable sort keeps each department's employees in alphabetical order; an unstable sort might scramble them.

**Q3: Implement Quicksort and explain its worst-case scenario.**
```java
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIdx = partition(arr, low, high);
        quickSort(arr, low, pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);
    }
}
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) { i++; swap(arr, i, j); }
    }
    swap(arr, i + 1, high);
    return i + 1;
}
```
Worst case O(n²) occurs when the pivot is **consistently the smallest or largest** element (e.g., always picking the last element as pivot on an already-sorted or reverse-sorted array) — each partition only removes **one** element instead of roughly halving the problem, degenerating into n nested partitions each O(n). Mitigation: **randomized pivot selection** or **median-of-three** (median of first/middle/last elements), which makes the worst case extremely unlikely in practice, though not impossible in theory.

**Q4: When would you choose Merge Sort over Quicksort, and vice versa?**

A: **Merge Sort** — guaranteed O(n log n) **worst case** (no pathological input degrades it), and is **stable** — preferred when worst-case guarantees matter (real-time systems, adversarial input possibility) or stability is required, and when sorting **linked lists** (merge sort doesn't need random access, unlike quicksort's partitioning which benefits from it) or **external sorting** (data too large for memory — merge sort's sequential access pattern suits disk-based merging well). **Quicksort** — typically **faster in practice** due to better cache locality and lower constant factors, and sorts **in-place** (O(log n) auxiliary space vs merge sort's O(n)) — preferred for general-purpose in-memory array sorting where average-case performance and memory efficiency matter more than worst-case guarantees.

**Q5: What is Counting Sort, and why can it beat the O(n log n) comparison-sort lower bound?**

A: Counting Sort works by counting the **occurrences of each distinct value** (assuming a known, limited range of integer keys, size k) and using those counts to place elements directly into their sorted position — O(n + k) time. It beats O(n log n) because it's **not a comparison-based sort** — the Ω(n log n) lower bound applies specifically to algorithms that determine order purely via pairwise comparisons; Counting Sort instead uses the **actual values** as array indices, sidestepping that theoretical limit entirely. The tradeoff: it only works efficiently when the key range `k` is reasonably small relative to `n` (a large or sparse range makes it worse than a comparison sort).

---

## 15. Searching Algorithms

**Q1: Implement Binary Search and explain its precondition.**
```java
public static int binarySearch(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```
O(log n) — but **requires the array to already be sorted**; a very common interview trap is applying binary search to unsorted data (or forgetting to check the precondition when it's implied but not explicit in the problem).

**Q2: What is the "search in rotated sorted array" pattern, and how does it modify standard binary search?**
```java
public static int searchRotated(int[] nums, int target) {
    int low = 0, high = nums.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] == target) return mid;
        if (nums[low] <= nums[mid]) { // left half is sorted
            if (nums[low] <= target && target < nums[mid]) high = mid - 1;
            else low = mid + 1;
        } else { // right half is sorted
            if (nums[mid] < target && target <= nums[high]) low = mid + 1;
            else high = mid - 1;
        }
    }
    return -1;
}
```
Key insight: even though the whole array isn't sorted, **at least one half** (relative to `mid`) always **is** sorted — determine which half is sorted first, then check if the target falls within that sorted half's range to decide which direction to continue searching, still achieving O(log n).

**Q3: What is "binary search on the answer," and when do you reach for this pattern?**

A: A technique for problems that aren't obviously a search-in-a-sorted-array problem, but where the **answer space itself has a monotonic property** — e.g., "find the minimum capacity needed to ship packages within D days": as capacity increases, the number of days needed to ship monotonically decreases (or stays the same). You binary search over the **range of possible answers** (here, capacity values), using a feasibility check (`canShipWithinDays(capacity) → boolean`) at each midpoint instead of a direct array-element comparison, converging on the minimum/maximum feasible value in O(log(range) × cost-of-feasibility-check). Recognizing "the answer is monotonic, and I can cheaply check if a candidate answer works" is the key trigger for this pattern.

---

## 16. Recursion & Backtracking

**Q1: What are the essential components every correct recursive function needs?**

A: A **base case** (the condition that stops recursion, preventing infinite calls/stack overflow), and a **recursive case** that makes **measurable progress toward the base case** with each call (a smaller/simpler version of the same problem). Missing or incorrect base cases, or recursive calls that don't actually shrink the problem, are the two most common sources of infinite recursion bugs.

**Q2: What is backtracking, and how does it differ from plain recursion/brute force?**

A: Backtracking is a refinement of brute-force recursive search that **prunes** invalid paths **as early as possible**, rather than fully exploring them and checking validity only at the end — at each step, you make a choice, recurse, and if that choice leads to a dead end (violates a constraint), you **undo it ("backtrack")** and try the next option. This early pruning is what makes backtracking practical for problems where the full brute-force search space is exponential but most branches can be eliminated quickly.

**Q3: Generate all permutations of an array — the canonical backtracking template.**
```java
public static List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
    return result;
}
private static void backtrack(int[] nums, List<Integer> current, boolean[] used, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current)); // must COPY — current is mutated after this
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.add(nums[i]);
        backtrack(nums, current, used, result);  // choose then explore
        current.remove(current.size() - 1);      // un-choose (backtrack)
        used[i] = false;
    }
}
```
This "**choose then explore then un-choose**" structure is the universal template for nearly all backtracking problems (permutations, combinations, subsets, N-Queens, Sudoku solvers) — recognizing this pattern is more valuable than memorizing any single problem's solution.

**Q4: Solve N-Queens (or explain the constraint-checking approach) to demonstrate backtracking with pruning.**
```java
public static int totalNQueens(int n) {
    return solve(n, 0, new HashSet<>(), new HashSet<>(), new HashSet<>());
}
private static int solve(int n, int row, Set<Integer> cols, Set<Integer> diag1, Set<Integer> diag2) {
    if (row == n) return 1;
    int count = 0;
    for (int col = 0; col < n; col++) {
        int d1 = row - col, d2 = row + col; // diagonal identifiers
        if (cols.contains(col) || diag1.contains(d1) || diag2.contains(d2)) continue; // PRUNE
        cols.add(col); diag1.add(d1); diag2.add(d2);
        count += solve(n, row + 1, cols, diag1, diag2);
        cols.remove(col); diag1.remove(d1); diag2.remove(d2); // backtrack
    }
    return count;
}
```
The `continue` on constraint violation is the pruning — instead of placing a queen and only discovering the conflict at the very end (after fully placing all n queens), the check happens **immediately**, avoiding wasted exploration of the entire remaining subtree beneath an already-invalid placement.

---

## 17. Divide and Conquer

**Q1: What are the three steps of the divide-and-conquer paradigm, and how does it differ from dynamic programming?**

A: **Divide** (split the problem into smaller independent subproblems), **Conquer** (solve each subproblem recursively), **Combine** (merge the subproblem solutions into the overall answer). The key distinguishing feature from **Dynamic Programming** (Section 19): divide-and-conquer subproblems are **independent** — they don't overlap, so there's no benefit to memoization/caching (merge sort never re-solves the same sub-array twice). DP specifically applies when subproblems **do overlap**, making caching the previously-computed results the entire point.

**Q2: Give the classic examples of divide-and-conquer algorithms.**

A: **Merge Sort** and **Quicksort** (Section 14), **Binary Search** (dividing the search space in half each step), the **Fast Fourier Transform**, **Karatsuba multiplication** (multiplying large numbers faster than the naive O(n²) grade-school method), and **finding the maximum subarray sum via divide-and-conquer** (O(n log n) — split into left half, right half, and a "crossing the midpoint" case — though Kadane's algorithm, Section 19, solves the same problem in O(n) via DP, a common "which approach is actually better" follow-up).

**Q3: Implement "find the maximum element in a rotated sorted array" using divide and conquer.**
```java
public static int findMax(int[] nums, int low, int high) {
    if (low == high) return nums[low];
    int mid = low + (high - low) / 2;
    if (nums[mid] > nums[high]) return findMax(nums, mid + 1, high); // max is in right half
    else return findMax(nums, low, mid); // max is in left half (including mid)
}
```
O(log n) — same "discard half the search space each step" divide-and-conquer spirit as binary search, adapted to a slightly different decision rule based on the rotation.

---

## 18. Greedy Algorithms

**Q1: What defines a greedy algorithm, and what property must a problem have for a greedy approach to actually produce the optimal solution?**

A: A greedy algorithm makes the **locally optimal choice at each step**, without reconsidering previous choices, hoping (and in valid cases, provably guaranteeing) this leads to a globally optimal solution. This only works correctly when the problem exhibits the **greedy-choice property** (a global optimum can always be reached by making a series of locally optimal choices) **and optimal substructure** (an optimal solution to the problem contains optimal solutions to its subproblems) — critically, **not all problems have these properties**, and applying a greedy approach to one that doesn't produces a plausible-looking but genuinely **wrong** answer (a very common interview trap: "does a greedy approach actually work here, or do you need DP?").

**Q2: Activity Selection Problem — classic greedy example.**
```java
public static int maxActivities(int[][] activities) { // each activity = [start, end]
    Arrays.sort(activities, (a, b) -> a[1] - b[1]); // sort by END time — the key greedy insight
    int count = 1, lastEnd = activities[0][1];
    for (int i = 1; i < activities.length; i++) {
        if (activities[i][0] >= lastEnd) { // no overlap with the last selected activity
            count++;
            lastEnd = activities[i][1];
        }
    }
    return count;
}
```
Sorting by **end time** (not start time — a common mistake) is what makes the greedy choice provably optimal: always picking the activity that finishes earliest leaves the **maximum possible remaining time** for future activities, which is exactly the greedy-choice property this problem satisfies.

**Q3: When does a greedy approach FAIL, using the classic coin-change counterexample?**

A: With coin denominations `{1, 3, 4}` and a target of `6`: greedy (always take the largest coin that fits) picks `4 + 1 + 1 = 3 coins`, but the actual optimal solution is `3 + 3 = 2 coins`. This demonstrates that greedy **doesn't always find the optimal answer** — it happens to work for "nice" denomination systems (like standard currency: 1, 5, 10, 25...) but fails for arbitrary ones, which is exactly why the general Coin Change problem requires **Dynamic Programming** (Section 19), not a greedy approach, despite superficially "feeling" greedy.

**Q4: Huffman Coding — how does greedy apply to optimal prefix-free encoding?**

A: Huffman coding builds an optimal (minimum expected code length) prefix-free binary encoding by **greedily merging the two lowest-frequency nodes** repeatedly (using a min-heap) into a new combined node, until only one tree remains — the resulting tree's leaf depths determine each character's code length, with more frequent characters ending up with shorter codes. This greedy merge strategy is **provably optimal** for this specific problem (a classical result), making it a commonly-cited "greedy actually works, and here's the heap-based implementation" interview topic, often paired with the coin-change counterexample to illustrate that greedy correctness must be proven per-problem, not assumed.

---

## 19. Dynamic Programming

**Q1: What are the two defining properties a problem must have for Dynamic Programming to apply, and what do they mean concretely?**

A: **Overlapping subproblems** — the same smaller subproblem is solved **repeatedly** across the recursive call tree (e.g., naive recursive Fibonacci recomputes `fib(2)` many times) — this is what makes **caching/memoization** valuable (if subproblems never repeated, as in Section 17's divide-and-conquer, caching would be pointless). **Optimal substructure** — an optimal solution to the overall problem can be constructed from optimal solutions to its subproblems (e.g., the shortest path from A to C through B is the shortest A→B path plus the shortest B→C path) — without this, even correctly cached subproblem answers wouldn't combine into a correct overall answer.

**Q2: What's the difference between top-down (memoization) and bottom-up (tabulation) DP approaches?**

A: **Top-down/memoization** — write the natural recursive solution, add a cache (`Map`/array) to store and reuse previously computed results, checking the cache before recursing further:
```java
public static int fibMemo(int n, Map<Integer, Integer> memo) {
    if (n <= 1) return n;
    if (memo.containsKey(n)) return memo.get(n);
    int result = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    memo.put(n, result);
    return result;
}
```
**Bottom-up/tabulation** — build the solution **iteratively** from the smallest subproblems up to the final answer, typically using an array/table, with no recursion (and thus no call-stack overhead):
```java
public static int fibTab(int n) {
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[1] = 1;
    for (int i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}
```
Top-down is often more **intuitive to derive** (closer to the natural recursive problem definition) and only computes subproblems that are actually needed (useful if not all subproblems are relevant for a given input); bottom-up avoids recursion overhead/stack depth limits entirely and is often easier to further optimize for **space** (see Q4).

**Q3: Solve the 0/1 Knapsack problem, and explain the DP state definition.**
```java
public static int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];
    // dp[i][w] = max value achievable using the first i items, with capacity w
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            dp[i][w] = dp[i - 1][w]; // don't take item i-1
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]); // take it
            }
        }
    }
    return dp[n][capacity];
}
```
The critical skill DP interview questions actually test: correctly **defining the state** (`dp[i][w]` = "best value using the first i items with capacity w") and the **recurrence relating a state to smaller states** (either skip the current item, or take it and reduce remaining capacity) — once those two things are right, the implementation is largely mechanical.

**Q4: How would you optimize the Knapsack solution's space from O(n × capacity) to O(capacity)?**
```java
public static int knapsackOptimized(int[] weights, int[] values, int capacity) {
    int[] dp = new int[capacity + 1];
    for (int i = 0; i < weights.length; i++) {
        for (int w = capacity; w >= weights[i]; w--) { // iterate BACKWARDS — critical detail
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    return dp[capacity];
}
```
Since `dp[i][w]` only ever depends on the **previous row** (`dp[i-1][...]`), you don't need to keep the entire 2D table — a single 1D array, updated **in place**, suffices. The backward iteration is essential: iterating forward would let an item's `dp[w - weights[i]]` read a value **already updated in the current pass**, effectively allowing the same item to be "reused" multiple times (which is correct for the *unbounded* knapsack variant, but wrong for 0/1 knapsack where each item is usable at most once) — a frequently-tested subtlety.

**Q5: Solve "Longest Common Subsequence" (LCS) — a very common DP string problem.**
```java
public static int lcs(String a, String b) {
    int[][] dp = new int[a.length() + 1][b.length() + 1];
    for (int i = 1; i <= a.length(); i++) {
        for (int j = 1; j <= b.length(); j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[a.length()][b.length()];
}
```
O(m·n) time and space — the recurrence pattern here (matching characters extend a diagonal predecessor; mismatches take the best of "drop from A" or "drop from B") is the direct foundation for many related problems (edit distance, longest palindromic subsequence, diff algorithms).

---

## 20. Graph Algorithms — Shortest Path, MST, Topological Sort

**Q1: Explain Dijkstra's algorithm — what it computes, its complexity, and its key limitation.**

A: Dijkstra's finds the **shortest path from a single source to all other vertices** in a weighted graph with **non-negative edge weights**. It repeatedly selects the unvisited vertex with the currently-smallest known distance (using a min-heap/priority queue), and **relaxes** its outgoing edges (updates neighbors' distances if a shorter path is found through this vertex) — O((V + E) log V) with a binary heap. **Key limitation**: it produces **incorrect** results if the graph has **negative edge weights** — the greedy "once finalized, never revisit" assumption breaks down, since a later negative edge could retroactively create a shorter path to an already-finalized vertex.

**Q2: What is the Bellman-Ford algorithm, and why/when would you use it over Dijkstra?**

A: Bellman-Ford also computes single-source shortest paths, but works correctly with **negative edge weights** (unlike Dijkstra) by relaxing **every edge, V-1 times** — O(V·E), notably slower than Dijkstra. It can also **detect negative cycles** (a cycle whose total weight is negative, making "shortest path" ill-defined/unbounded) — after V-1 relaxation rounds, if a **V-th round still finds an improvement**, a negative cycle must exist somewhere reachable from the source. Use Bellman-Ford specifically when negative weights are possible (Dijkstra's speed advantage doesn't matter if it gives a wrong answer).

**Q3: What does Floyd-Warshall compute, and how does its use case differ from Dijkstra/Bellman-Ford?**

A: Floyd-Warshall computes **all-pairs shortest paths** (shortest distance between **every** pair of vertices, not just from one source) in O(V³) using dynamic programming (`dp[k][i][j]` = shortest path from i to j using only intermediate vertices up to k). It handles negative edges (but not negative cycles). Use it when you genuinely need all-pairs distances on a **relatively small** graph (O(V³) is prohibitive for large V) — versus running Dijkstra from every single vertex individually (O(V·(E log V))), which can actually be **faster** than Floyd-Warshall for sparse graphs despite computing the same all-pairs result, a common "which is actually better here" follow-up.

**Q4: Explain Minimum Spanning Tree (MST), and compare Kruskal's vs Prim's algorithms.**

A: An MST is a subset of edges connecting **all vertices** with **minimum total edge weight**, containing **no cycles** (exactly V-1 edges for V vertices). **Kruskal's** — sort **all edges** by weight ascending, then greedily add each edge **if it doesn't create a cycle** (checked via Union-Find, Section 12) — O(E log E), and naturally suited to **sparse** graphs (works directly on the edge list). **Prim's** — start from an arbitrary vertex and greedily grow the tree by always adding the **cheapest edge connecting the current tree to a new vertex** (using a min-heap) — O(E log V), and naturally suited to **dense** graphs (works vertex-by-vertex, similar structurally to Dijkstra). Both are greedy and both are provably optimal for this specific problem (unlike the general coin-change counterexample from Section 18) — a good example to contrast against when discussing when greedy actually works.

**Q5: Explain Topological Sort — what it requires, and both the DFS-based and Kahn's-algorithm (BFS-based) approaches.**

A: Topological sort produces a **linear ordering of vertices** such that for every directed edge u→v, u comes **before** v in the ordering — only possible on a **Directed Acyclic Graph (DAG)**; a cycle makes it impossible (there'd be no valid "before" for at least one pair). Common use case: task scheduling with dependencies (build systems, course prerequisites).
```java
// Kahn's algorithm (BFS-based) — also naturally detects cycles
public static List<Integer> topoSort(int n, Map<Integer, List<Integer>> graph) {
    int[] inDegree = new int[n];
    for (List<Integer> neighbors : graph.values())
        for (int neighbor : neighbors) inDegree[neighbor]++;

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < n; i++) if (inDegree[i] == 0) queue.offer(i);

    List<Integer> order = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (--inDegree[neighbor] == 0) queue.offer(neighbor);
        }
    }
    // if order.size() < n, a cycle exists — not all nodes could be processed
    return order.size() == n ? order : new ArrayList<>(); // empty = cycle detected
}
```
The DFS-based approach instead does a post-order DFS traversal and **reverses** the finish order (a node finishes after all its dependents are fully explored, so reversing gives the correct dependency order) — both are O(V + E); Kahn's is often preferred when you also want **explicit, easy cycle detection** as a natural byproduct (if the final order has fewer than V nodes, a cycle exists).

---

## 21. Bit Manipulation

**Q1: What are the core bitwise operators, and what does each do?**

A: `&` (AND — 1 only if both bits are 1), `|` (OR — 1 if either bit is 1), `^` (XOR — 1 if bits differ), `~` (NOT — flips all bits), `<<` (left shift — multiply by 2 per shift), `>>` (right shift — divide by 2 per shift, sign-extending for negative numbers), `>>>` (unsigned right shift — fills with 0 regardless of sign, Java-specific).

**Q2: What are the classic bit-manipulation tricks every candidate should know?**
```java
// Check if a number is a power of 2
boolean isPowerOfTwo(int n) { return n > 0 && (n & (n - 1)) == 0; }
// Why: a power of 2 has exactly ONE set bit; n-1 flips that bit and sets all lower bits to 1,
// so ANDing them together yields 0 only for powers of 2.

// Count set bits (Brian Kernighan's algorithm)
int countSetBits(int n) {
    int count = 0;
    while (n != 0) { n &= (n - 1); count++; } // clears the LOWEST set bit each iteration
    return count;
}

// Find the single non-duplicate number in an array where every other number appears twice
int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) result ^= num; // XOR cancels out pairs: a ^ a = 0, a ^ 0 = a
    return result;
}

// Swap two numbers without a temp variable
void swap(int[] arr, int i, int j) {
    if (i == j) return; // IMPORTANT: skip if same index, or XOR-swap zeroes the value out
    arr[i] ^= arr[j];
    arr[j] ^= arr[i];
    arr[i] ^= arr[j];
}
```

**Q3: Why is `n & (n - 1)` such a useful building block, beyond just the power-of-2 check?**

A: `n & (n - 1)` always **clears the lowest set bit** of `n` — this single fact underlies several algorithms: counting set bits (repeat until n becomes 0, counting iterations — Brian Kernighan's algorithm above, which is O(number of set bits) rather than O(total bits), faster than naively checking every bit position), and detecting powers of 2 (a power of 2 has exactly one set bit, so clearing it yields exactly 0). Recognizing this single bit-trick building block explains a disproportionate number of "clever" bit manipulation interview answers.

---

## 22. Two Pointers & Sliding Window

**Q1: What is the two-pointer technique, and what problem characteristics make it applicable?**

A: Two pointers traverse a data structure (usually a sorted array or a string) from **different starting positions** (often both ends, or one fixed and one moving), moving them based on some comparison logic, to avoid the O(n²) cost of checking all pairs individually. It's applicable when the data has some **ordering/monotonicity property** that lets you make a directional decision at each step (e.g., in a sorted array, if the current pair sum is too small, moving the left pointer right can only increase the sum — a decision you can trust without checking all alternatives).

**Q2: Two Sum on a SORTED array using two pointers (contrast with the hash-map version from Section 7) — O(1) space.**
```java
public static int[] twoSumSorted(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return new int[]{left, right};
        else if (sum < target) left++;  // need a bigger sum — move left pointer up
        else right--;                    // need a smaller sum — move right pointer down
    }
    return new int[]{-1, -1};
}
```
O(n) time, O(1) space — better than the hash-map version's O(n) **space** when the array is already sorted (or sorting it first is acceptable, adding O(n log n) but still avoiding extra space beyond that).

**Q3: What's the difference between a fixed-size and variable-size sliding window, with an example of each?**

A: **Fixed-size** — the window size `k` is given/constant; you slide it one position at a time, typically maintaining a running aggregate (sum, max) incrementally rather than recomputing from scratch each time (e.g., "maximum sum of any k consecutive elements" — add the new element entering the window, subtract the one leaving). **Variable-size** — the window **grows and shrinks** dynamically based on a condition (as seen in Section 3's "longest substring without repeating characters") — expand the right edge to include more elements, and contract the left edge whenever the window becomes invalid, tracking the best window size seen along the way. Recognizing "I need a contiguous range that satisfies some condition, and I want the longest/shortest/optimal one" is the trigger for reaching for sliding window over a brute-force check of every possible sub-range.

---

# PART 5: MOST-ASKED CODING PATTERNS

## 23. Pattern Catalog with Solutions

**Q1: Maximum Subarray Sum (Kadane's Algorithm) — the canonical DP-meets-greedy pattern.**
```java
public static int maxSubArray(int[] nums) {
    int maxSoFar = nums[0], maxEndingHere = nums[0];
    for (int i = 1; i < nums.length; i++) {
        maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]); // extend or restart
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
```
O(n) — at each position, decide whether extending the previous subarray is still beneficial, or whether starting fresh from the current element alone is better (this "extend or restart" decision, based only on the immediately preceding state, is what qualifies this as DP despite the deceptively simple implementation).

**Q2: Merge Intervals — a very common "sort first, then greedily merge" pattern.**
```java
public static int[][] mergeIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start time
    List<int[]> merged = new ArrayList<>();
    for (int[] interval : intervals) {
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < interval[0]) {
            merged.add(interval); // no overlap with the last merged interval
        } else {
            merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], interval[1]); // extend
        }
    }
    return merged.toArray(new int[0][]);
}
```
O(n log n), dominated by the sort — once sorted by start time, a single linear pass suffices since any overlap can only occur with the **most recently merged** interval.

**Q3: Number of Islands — the canonical "flood fill via BFS/DFS on a grid" pattern.**
```java
public static int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                count++;
                sink(grid, i, j); // flood-fill to mark this whole island as visited
            }
        }
    }
    return count;
}
private static void sink(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] != '1') return;
    grid[i][j] = '0'; // mark visited
    sink(grid, i + 1, j); sink(grid, i - 1, j); sink(grid, i, j + 1); sink(grid, i, j - 1);
}
```
O(rows × cols) — every cell is visited a constant number of times total. This grid-traversal-via-DFS/BFS template generalizes to a huge family of problems (surrounded regions, max area of island, rotting oranges via BFS for the "minimum time" variant).

**Q4: Course Schedule (can all courses be finished?) — cycle detection via topological sort, applied.**
```java
public static boolean canFinish(int numCourses, int[][] prerequisites) {
    Map<Integer, List<Integer>> graph = new HashMap<>();
    int[] inDegree = new int[numCourses];
    for (int[] p : prerequisites) {
        graph.computeIfAbsent(p[1], k -> new ArrayList<>()).add(p[0]);
        inDegree[p[0]]++;
    }
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) queue.offer(i);
    int processed = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        processed++;
        for (int next : graph.getOrDefault(course, List.of())) {
            if (--inDegree[next] == 0) queue.offer(next);
        }
    }
    return processed == numCourses; // if fewer were processed, a cycle blocked some courses
}
```
Directly reuses the Kahn's-algorithm topological sort template from Section 20 — recognizing "this is secretly a cycle-detection-in-a-DAG problem" is the actual interview skill being tested, not the algorithm implementation itself.

**Q5: Climbing Stairs — the "disguised Fibonacci" DP pattern, useful for recognizing when a problem reduces to a known recurrence.**
```java
public static int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```
O(n) time, O(1) space — the recurrence `ways(n) = ways(n-1) + ways(n-2)` (you reach step n either from step n-1 with a single step, or step n-2 with a double step) is structurally identical to Fibonacci, a very common "this new-sounding problem is actually a classic recurrence in disguise" pattern-recognition test.

---

## Quick-Reference Complexity Cheat Sheet

| Data Structure | Access | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(1) at the end (amortized) |
| Linked List | O(n) | O(n) | O(1)* | O(1)* | *O(1) once positioned at the node |
| Stack/Queue | O(n) | O(n) | O(1) | O(1) | Operations restricted to one/both ends |
| Hash Table | — | O(1) avg | O(1) avg | O(1) avg | O(n) worst case (heavy collisions) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) worst case if unbalanced |
| Heap | — | O(n) | O(log n) | O(log n) root | O(1) to peek min/max |
| Trie | — | O(L) | O(L) | O(L) | L = length of the string, independent of # words stored |

| Algorithm | Time | Space | Key Use Case |
|---|---|---|---|
| Binary Search | O(log n) | O(1) | Sorted array lookup, "binary search on the answer" |
| Merge Sort | O(n log n) | O(n) | Stable, guaranteed worst-case, linked lists |
| Quick Sort | O(n log n) avg | O(log n) | Fastest in practice, in-place |
| BFS | O(V+E) | O(V) | Shortest path (unweighted), level-order |
| DFS | O(V+E) | O(V) | Cycle detection, topological sort, backtracking |
| Dijkstra | O((V+E) log V) | O(V) | Shortest path, non-negative weights |
| Bellman-Ford | O(V·E) | O(V) | Shortest path with negative weights, cycle detection |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs shortest path, small graphs |
| Kruskal's / Prim's | O(E log E) / O(E log V) | O(V) | Minimum Spanning Tree |
| Union-Find | ~O(α(n)) | O(n) | Cycle detection, connectivity, Kruskal's |

---

*Tip: the single highest-leverage DSA interview skill isn't memorizing solutions — it's **pattern recognition**. Before coding, explicitly ask "is this two pointers, sliding window, BFS/DFS on a graph/grid, backtracking, or DP?" out loud, state which signals led you there (sorted input → two pointers; "contiguous subarray/substring" → sliding window; "count ways" or "min/max" with overlapping subproblems → DP), and confirm your approach with the interviewer before writing code. That verbal reasoning is usually worth more than a perfect first-try implementation.*
