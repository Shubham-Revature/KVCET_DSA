# Deques and Priority Queues

A standard Queue enforces strict FIFO (First-In, First-Out) fairness. However, real-world systems rarely operate under a single rigid rule. Some workloads require bidirectional access where elements can enter or exit from either side, while other systems must abandon arrival order entirely to handle urgent, high-priority tasks first.

To solve these challenges, we rely on two specialized data structures: the **Double-Ended Queue (Deque)** and the **Priority Queue**.

---

## 1. The Deque (Double-Ended Queue)

**Deque** (pronounced *"deck"*) stands for **Double-Ended Queue**. 

Unlike a standard Queue (which restricts insertion to the back and removal to the front), a Deque allows you to **enqueue and dequeue from BOTH ends in instant $O(1)$ constant time**.

```text
               +-----------------------------------+
  appendleft() |                                   | append()
-------------> |   [Node] <---> [Node] <---> [Node]| <-------------
<------------- |                                   | ------------->
   popleft()   +-----------------------------------+   pop()
```

### Under the Hood: Doubly Linked List Architecture

In Python, `collections.deque` is implemented in CPython as an optimized doubly linked list composed of fixed-size memory blocks (typically 64 elements per block node). 

This design choice creates a crucial contrast with standard Python dynamic arrays:
- **Python `list` (Dynamic Array):** Stored in contiguous memory. Appending to or popping from the right end is $O(1)$ amortized. However, inserting or removing at index 0 (`list.insert(0, val)` or `list.pop(0)`) forces Python to physically shift every remaining element over by one slot in memory—a slow **$O(N)$ linear time** operation.
- **`collections.deque` (Doubly Linked Blocks):** Because data is linked via forward and backward pointers, adding or removing items from either extreme never shifts elements. Python simply updates pointer references or links a new block in instant **$O(1)$ time**.

### Core Methods and Capabilities

To use a `deque`, you import it from the built-in `collections` module:

```python
from collections import deque

d = deque([10, 20, 30])

# Right-end operations (O(1))
d.append(40)          # deque([10, 20, 30, 40])
tail = d.pop()        # tail = 40, deque([10, 20, 30])

# Left-end operations (O(1))
d.appendleft(0)       # deque([0, 10, 20, 30])
head = d.popleft()    # head = 0, deque([10, 20, 30])
```

> [!TIP]
> **Bounded Buffers with `maxlen`:** You can initialize a deque with a fixed maximum size: `d = deque(maxlen=3)`. When the deque reaches capacity and a new item is appended, Python automatically evicts the item from the opposite end in $O(1)$ time. This is ideal for sliding windows and circular activity logs.

### The Trade-Off: Lookups vs. End Mutations

Because a `deque` is a doubly linked structure rather than a single contiguous block of memory, it introduces a classic computer science trade-off:

| Operation | Python `list` (Dynamic Array) | `collections.deque` (Doubly Linked Blocks) |
|---|---|---|
| **Append / Pop (Right End)** | $O(1)$ amortized | $O(1)$ |
| **Insert / Pop (Left End)** | $O(N)$ (forces element shifting) | $O(1)$ (pointer update) |
| **Middle Insertion / Deletion** | $O(N)$ (element shifting) | $O(N)$ (pointer traversal) |
| **Random Access Lookup (`seq[i]`)** | **$O(1)$** (instant address arithmetic) | **$O(N)$** (pointer traversal from nearest end) |
| **Memory Overhead** | Low (single contiguous array buffer) | Higher (node pointers & block overhead) |

**Rule of Thumb:**
- Use a **standard `list`** when you need fast random reads by index (`arr[i]`) or frequent slicing.
- Use a **`deque`** when your primary workload involves adding or removing elements at either or both ends.

### Common Use Cases for a Deque
1. **Work-Stealing Schedulers:** In multithreaded systems, each worker thread processes tasks from the front of its own deque. When an idle worker runs out of tasks, it can "steal" a task from the *back* of a busy worker's deque without locking the front.
2. **Palindrome Checking:** Place characters in a deque and verify symmetry by repeatedly popping simultaneously from the left and right ends.
3. **Sliding Window Processing:** Maintain a rolling window of recent values or monotonic candidate indices for running calculations.

---

## 2. The Priority Queue

Imagine a hospital emergency room where patients wait to see a doctor:
- Patient A arrives at 1:00 PM with a paper cut.
- Patient B arrives at 1:05 PM with acute chest pain.

If the emergency room used a strict FIFO Queue, Patient A would be treated first while Patient B waited in danger. Instead, hospitals use a **Priority Queue**.

In a Priority Queue, FIFO arrival order is completely replaced by priority weighting: **the item with the highest priority is always served first, regardless of when it joined the line.**

```text
Arrival Order:   [Paper Cut (Priority 3)] -> [Chest Pain (Priority 1)] -> [Broken Leg (Priority 2)]
                                  │
                          Priority Queue
                                  ▼
Service Order:   1. Chest Pain (Priority 1)
                 2. Broken Leg (Priority 2)
                 3. Paper Cut  (Priority 3)
```

### Python's `heapq` Module

Priority Queues are typically implemented using a specialized tree structure called a **Binary Heap** (which we explore in detail in Week 8). 

In Python, we do not use lists with repeated sorting or `collections.deque` for priority scheduling. Instead, we use the standard library's **`heapq`** module.

- **Underlying structure:** A binary heap represented inside a standard Python list.
- **Time Complexity:** Inserting an element (`heappush`) and removing the top element (`heappop`) both run in **$O(\log N)$ Logarithmic Time**. This is vastly faster than re-sorting a list ($O(N \log N)$), while providing continuous priority order.
- **Min-Heap Convention:** By default, Python's `heapq` is a **Min-Heap**. The *smallest* number is treated as having the highest priority. If you need a Max-Heap (where larger numbers are served first), multiply priority scores by `-1`.

---

## Try it Yourself

**Scenario:** Implement both data structures to solve real-world problems:
1. Verify if words are palindromes using `collections.deque`.
2. Manage patient triage in a hospital emergency room using `heapq`.

```python
from collections import deque
import heapq

# --- 1. Deque: Palindrome Checker ---
def is_palindrome(text: str) -> bool:
    clean = "".join(ch.lower() for ch in text if ch.isalnum())
    d = deque(clean)
    
    while len(d) > 1:
        if d.popleft() != d.pop():
            return False
    return True

assert is_palindrome("Racecar") is True
assert is_palindrome("Antigravity") is False
print("Palindrome checks passed successfully!")


# --- 2. Priority Queue: Hospital ER Triage ---
er_queue = []

# Patients arrive: (Priority_Score, Patient_Name)
# Lowest number = highest urgency (Min-Heap)
heapq.heappush(er_queue, (3, "Alice (Paper Cut)"))
heapq.heappush(er_queue, (1, "Bob (Heart Attack)"))
heapq.heappush(er_queue, (2, "Charlie (Broken Leg)"))

# The doctor is ready: the patient with the highest urgency is dequeued first
priority, patient = heapq.heappop(er_queue)

assert patient == "Bob (Heart Attack)"
print(f"Doctor is seeing: {patient} [Priority Level {priority}]")
```
```text
Palindrome checks passed successfully!
Doctor is seeing: Bob (Heart Attack) [Priority Level 1]
```

---

## Common Mistakes

- **Using `.pop(0)` on standard lists:** Using `my_list.pop(0)` or `my_list.insert(0, val)` during technical interviews is an immediate red flag. It proves a lack of understanding regarding array memory shifting ($O(N)$ penalty). If you need front-of-sequence modifications, always use `collections.deque`.
- **Randomly indexing into a `deque`:** Because a `deque` is a doubly linked structure under the hood, accessing elements in the middle (e.g., `d[5000]`) requires walking through node pointers and runs in $O(N)$ time. If you need fast random index lookups, use a standard Python list.
- **Sorting a list to simulate a Priority Queue:** Appending items to a standard list and calling `.sort()` runs in $O(N \log N)$ time per insertion. Using `heapq.heappush()` runs in $O(\log N)$ time and keeps the collection organized without full sorting passes.
- **Forgetting that `heapq` is a Min-Heap:** By default, Python pops the smallest number first. If you want the largest number to pop first, invert priority weights by multiplying them by `-1`.
- **Forgetting imports:** Neither `deque` nor `heapq` exist in Python's default global namespace. You must explicitly write `from collections import deque` and `import heapq`.

---

## Key Takeaway

When standard FIFO queues are too restrictive, we turn to specialized queues:
- A **Deque (`collections.deque`)** is an optimized doubly linked structure that provides bidirectional **$O(1)$ insertions and deletions** at both the front (`appendleft` / `popleft`) and back (`append` / `pop`). However, random index lookups in the middle take $O(N)$ time.
- A **Priority Queue (`heapq`)** abandons FIFO order entirely to ensure elements with the highest priority are always processed first. Backed by a binary min-heap, insertions and extractions run in **$O(\log N)$ time**.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Deque** | Double-Ended Queue; an abstract data type allowing $O(1)$ insertions and deletions at both ends. |
| **`appendleft()` / `popleft()`** | Methods specific to `deque` that modify the front of the collection in $O(1)$ constant time. |
| **Doubly Linked Blocks** | The internal CPython implementation of `deque`, storing chunks of elements linked by pointers. |
| **Priority Queue** | An abstract data type where elements are dequeued according to assigned priority rather than arrival order. |
| **Heap** | A specialized tree-based data structure that satisfies the heap property and powers efficient Priority Queues. |
| **`heapq`** | Python's standard library module providing binary min-heap algorithms over standard lists in $O(\log N)$ time. |

---

## References

- [Python Documentation: collections.deque](https://docs.python.org/3/library/collections.html#collections.deque)
- [Python Documentation: heapq module](https://docs.python.org/3/library/heapq.html)
