# The LRU Cache: The Ultimate Hybrid Data Structure

In production distributed systems, databases, web browsers, and CPU architectures, **caching** is the single most effective performance optimization. 

A cache keeps frequently accessed data in ultra-fast RAM so the system does not need to perform expensive disk reads or database queries. However, RAM is strictly finite. When your cache runs out of memory, **which item do you throw away?**

The most popular, battle-tested eviction policy in computer science is **LRU: Least Recently Used**. It dictates:
> *"When the cache reaches capacity, evict the item that has not been accessed for the longest period of time."*

Designing an LRU Cache is one of the most famous technical interview problems in the world (LeetCode 146). The challenge? Both reading data (`get`) and writing data (`put`) must execute in **instant $O(1)$ Constant Time**!

In this capstone lesson of Week 10, we will build an LRU Cache by uniting two fundamental data structures into a seamless hybrid: the **Hash Map** and the **Doubly Linked List**.

---

## 1. The Operational Requirements

An LRU Cache with capacity $C$ must support two primary operations in strict **$O(1)$ time**:

1. **`get(key)`:**
   - If `key` exists in the cache, return its value and **mark it as Most Recently Used (MRU)**.
   - If `key` does not exist, return `-1`.
2. **`put(key, value)`:**
   - If `key` already exists, update its value and **mark it as MRU**.
   - If `key` is new, insert it as MRU.
   - If inserting causes total items to exceed capacity $C$, **evict the Least Recently Used (LRU) item**!

```text
Capacity = 3
State:    [MRU] "Alice" <---> "Bob" <---> "Charlie" [LRU]

1. User calls: get("Charlie")
   "Charlie" is accessed! Move to front:
   [MRU] "Charlie" <---> "Alice" <---> "Bob" [LRU]

2. User calls: put("Dave", 100)
   Capacity full! Evict LRU ("Bob") and insert "Dave" at front:
   [MRU] "Dave" <---> "Charlie" <---> "Alice" [LRU]
```

---

## 2. The Data Structure Dilemma: Why Neither Structure Works Alone

To achieve $O(1)$ for both operations, let's test our existing data structures:

| Data Structure | Lookup Speed | Move-to-Front / Eviction Speed | Verdict |
|---|---|---|---|
| **Array / Python List** | $O(1)$ by index, but $O(N)$ by key | $O(N)$ (Shifting elements takes linear time) | **Fails** |
| **Singly Linked List** | $O(N)$ (Must traverse from head) | $O(N)$ (Cannot unlink node in $O(1)$ without `prev`) | **Fails** |
| **Hash Map Alone** | **$O(1)$ instant key lookup** | $O(N)$ (No concept of chronological recency) | **Fails** |
| **Doubly Linked List Alone** | $O(N)$ (Must search sequentially) | **$O(1)$ instant unlinking and head insertion** | **Fails** |

### The Grand Marriage: Hash Map + Doubly Linked List
Notice the symmetry:
- The **Hash Map** excels at what the Linked List fails: **$O(1)$ search**.
- The **Doubly Linked List** excels at what the Hash Map fails: **$O(1)$ node removal and reordering**.

By combining them, we create the ultimate hybrid:
1. The **Hash Map** maps `key ──► Node pointer`.
2. The **Doubly Linked List** stores the actual `Node` objects ordered by recency.

```text
Hash Map:
{
  "A": Node_A,
  "B": Node_B,
  "C": Node_C
}
        │           │           │
        ▼           ▼           ▼
Doubly Linked List:
[head] <---> [Node_A] <---> [Node_B] <---> [Node_C] <---> [tail]
 (MRU)                                                     (LRU)
```

---

## 3. The Power of Sentinel (Dummy) Nodes

Managing head and tail edge cases in a Doubly Linked List (empty list, single element, removing the head, removing the tail) usually requires dozens of messy `if node == self.head:` checks.

We eliminate 100% of these edge cases by using **Sentinel Nodes** (also called Dummy Nodes):
- `self.head`: A permanent dummy node at the front (representing Most Recently Used).
- `self.tail`: A permanent dummy node at the back (representing Least Recently Used).

```text
Permanent Structure:
[dummy_head] <---------------------------------------> [dummy_tail]
                 (Real data nodes inserted between)
```

- The **Most Recently Used (MRU)** item is always: `self.head.next`.
- The **Least Recently Used (LRU)** item is always: `self.tail.prev`.

### Two Fundamental DLL Helper Methods
With sentinel nodes, our pointer mutations require only two tiny, bulletproof helper methods:

#### 1. `_remove(node)`: Unlink a node in $O(1)$
```python
def _remove(self, node: Node) -> None:
    node.prev.next = node.next
    node.next.prev = node.prev
```

#### 2. `_add_to_front(node)`: Insert a node right after `self.head` in $O(1)$
```python
def _add_to_front(self, node: Node) -> None:
    node.next = self.head.next
    node.prev = self.head
    self.head.next.prev = node
    self.head.next = node
```

Whenever any node is accessed or updated, we simply call:
```python
self._remove(node)
self._add_to_front(node)
```
This moves the node to the front in **4 pointer reassignments ($O(1)$ time)**!

---

## 4. Complete Implementation: LRU Cache from Scratch

```python
class Node:
    def __init__(self, key: int = 0, value: int = 0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}  # Maps key -> Node pointer

        # Permanent sentinel boundary nodes
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: Node) -> None:
        """Unlinks a node from its current position in O(1)."""
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_front(self, node: Node) -> None:
        """Inserts a node immediately after the dummy head in O(1)."""
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1

        node = self.cache[key]
        # Mark as Most Recently Used
        self._remove(node)
        self._add_to_front(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            node.value = value  # Update value
            # Mark as Most Recently Used
            self._remove(node)
            self._add_to_front(node)
        else:
            # If at capacity, evict the LRU element (node before dummy tail)
            if len(self.cache) >= self.capacity:
                lru_node = self.tail.prev
                self._remove(lru_node)
                del self.cache[lru_node.key]  # Remove from hash map!

            # Insert new node as MRU
            new_node = Node(key, value)
            self.cache[key] = new_node
            self._add_to_front(new_node)
```

---

## 5. Python's Standard Library: `OrderedDict` and `@lru_cache`

While building an LRU Cache from scratch is mandatory for technical interviews, Python provides two pre-built production solutions in its standard library:

### 1. `collections.OrderedDict`
Python's `OrderedDict` is internally implemented using this exact Hash Map + Doubly Linked List hybrid! It exposes two dedicated methods for LRU operations:
- `od.move_to_end(key)`: Moves a key to the end (MRU) in $O(1)$.
- `od.popitem(last=False)`: Pops the first item (LRU) in $O(1)$.

```python
from collections import OrderedDict

class LRUCacheOrderedDict:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)  # Mark as MRU
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # Evict LRU (first item)
```

### 2. `@functools.lru_cache`
In Week 7, we used `@lru_cache` to memoize recursive functions (like Fibonacci). Now you know what it does under the hood: it maintains an internal LRU cache so recursive calls return instantly in $O(1)$ time while discarding stale results!

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_database_query(user_id: int):
    # Automatically cached using the LRU eviction policy!
    pass
```

---

## Try it Yourself

**Scenario:** Execute the classic LeetCode 146 validation sequence on our scratch-built `LRUCache` with capacity 2.

*Your Spec:*
1. Initialize `LRUCache(capacity=2)`.
2. `put(1, 1)` and `put(2, 2)`.
3. `get(1)`: Returns `1` and promotes `1` to MRU. (Now `2` is LRU!).
4. `put(3, 3)`: Capacity exceeded! Evicts `2`.
5. `get(2)`: Must return `-1` (successfully evicted).
6. `put(4, 4)`: Capacity exceeded! Evicts `1` (which was older than `3`).
7. `get(1)`: Must return `-1`.
8. `get(3)`: Must return `3`.
9. `get(4)`: Must return `4`.

```python
# 1. Initialize cache with capacity 2
lru = LRUCache(capacity=2)

# 2. Insert key 1 and key 2
lru.put(1, 1)
lru.put(2, 2)

# 3. Access key 1 -> Promotes key 1 to MRU, making key 2 the LRU
assert lru.get(1) == 1

# 4. Insert key 3 -> Capacity exceeded! Evicts key 2 (LRU)
lru.put(3, 3)

# 5. Verify key 2 was evicted
assert lru.get(2) == -1

# 6. Insert key 4 -> Capacity exceeded! Evicts key 1 (LRU)
lru.put(4, 4)

# 7. Verify key 1 was evicted, but keys 3 and 4 remain accessible
assert lru.get(1) == -1
assert lru.get(3) == 3
assert lru.get(4) == 4

print("All LRU Cache operations and eviction sequences verified successfully!")
```
```text
All LRU Cache operations and eviction sequences verified successfully!
```

---

## Common Mistakes

- **Forgetting to update recency on `get()`:** A common error is writing `return self.cache[key].value` without moving the node to the front. Reading an item counts as accessing it! If you don't update recency on reads, frequently accessed data will be erroneously evicted.
- **Forgetting to remove evicted keys from the Hash Map:** When evicting `self.tail.prev`, you must delete the key from the dictionary: `del self.cache[lru_node.key]`. If you only unlink the node from the linked list, the dictionary retains the stale pointer, creating memory leaks and incorrect capacity tracking.
- **Storing only values in the Doubly Linked List Node:** If your `Node` only stores `value` (`self.value`) and not `key` (`self.key`), when you evict `lru_node = self.tail.prev`, you will not know what key to delete from `self.cache`! Nodes **must** store both `key` and `value`.
- **Modifying pointers without sentinel nodes:** Writing doubly linked list pointer mutations without dummy head/tail nodes leads to brittle spaghetti code with null pointer exceptions.

---

## Key Takeaway

The **LRU Cache** solves the bounded memory caching dilemma in strict **$O(1)$ Time Complexity** by marrying a **Hash Map** (for instantaneous $O(1)$ key lookups) with a **Doubly Linked List** (for instantaneous $O(1)$ node removal and head insertion). Sentinel dummy nodes (`head` and `tail`) eliminate boundary edge cases. Python's standard library provides this exact hybrid via `collections.OrderedDict` and the `@functools.lru_cache` decorator.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **LRU Cache** | Least Recently Used Cache; an eviction algorithm that discards the least recently accessed item when capacity is exceeded. |
| **MRU** | Most Recently Used; the item that was accessed or inserted most recently (stored at the head of the list). |
| **Sentinel Node** | A dummy boundary node (like dummy head or tail) that holds no data and eliminates null-pointer edge cases. |
| **Doubly Linked List** | A sequence of nodes where each node contains pointers to both the next and previous nodes. |
| **`OrderedDict`** | A Python dictionary subclass that remembers the order entries were added and supports $O(1)$ reordering. |
| **Eviction Policy** | The algorithmic rule used by a cache to decide which data to discard when memory capacity is full. |

---

## References

- [LeetCode 146: LRU Cache](https://leetcode.com/problems/lru-cache/)
- [Python Documentation: collections.OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict)
- [Python Documentation: functools.lru_cache](https://docs.python.org/3/library/functools.html#functools.lru_cache)
- [GeeksforGeeks: LRU Cache Implementation](https://www.geeksforgeeks.org/lru-cache-implementation/)
