# Building a Hash Table from Scratch

In the previous two lessons, we explored the mathematical foundations of hashing:
1. **Hash Functions:** Compressing arbitrary data keys into compact integer array bounds.
2. **Collision Resolution:** Handling the inevitable collisions via Separate Chaining or Open Addressing.

Now, we bridge the gap between theory and real-world software engineering. In this lesson, we will build a complete, production-grade **Hash Table from scratch in Python**. 

By the end of this module, you will not only understand how Python's built-in `dict` works under the hood, but you will also implement the engine of scalability that allows hash maps to grow indefinitely: **Dynamic Resizing and Rehashing**.

---

## 1. Anatomy of a Production Hash Table

To build a data structure that behaves just like Python's native dictionary (`ht["apple"] = 5`), our class needs four internal components:

```text
               HashTable (Separate Chaining Model)
┌──────────────────────────────────────────────────────────┐
│ capacity = 8                                            │
│ size = 3                                                │
│ load_factor = 3 / 8 = 0.375                             │
│                                                          │
│ buckets:                                                 │
│ [ 0 ] ──► [ ["key1", val1] ] ──► None                    │
│ [ 1 ] ──► None                                           │
│ [ 2 ] ──► [ ["key2", val2] ] ──► [ ["key3", val3] ]      │
│ [ 3 ] ──► None                                           │
│ ...                                                      │
└──────────────────────────────────────────────────────────┘
```

1. **`self.capacity`:** The total number of available slots (buckets) in the underlying array. We will start with a small capacity (e.g., 4 or 8) to observe resizing in action.
2. **`self.size`:** The count of unique key-value pairs currently stored in the table.
3. **`self.buckets`:** A list of size `capacity`, where each bucket is a chain (list of `[key, value]` pairs) to resolve collisions via Separate Chaining.
4. **`LOAD_FACTOR_THRESHOLD`:** The density trigger (typically `0.75`). When $\alpha = \frac{\text{size}}{\text{capacity}} > 0.75$, the table automatically doubles its capacity to prevent performance degradation.

---

## 2. Python Magic Methods (Dunder Methods)

In Python, you don't want your users to call clumsy methods like `ht.insert("apple", 5)` or `ht.retrieve("apple")`. You want them to use standard dictionary syntax!

To make our custom class feel like a first-class citizen of the Python language, we implement Python's built-in **Data Model Magic Methods**:

| Syntax | Magic Method | Responsibility |
|---|---|---|
| `ht[key] = value` | `__setitem__(self, key, value)` | Inserts a new key-value pair or updates an existing key. |
| `val = ht[key]` | `__getitem__(self, key)` | Retrieves the value for `key`, or raises `KeyError`. |
| `del ht[key]` | `__delitem__(self, key)` | Removes the key-value pair, or raises `KeyError`. |
| `key in ht` | `__contains__(self, key)` | Returns `True` if `key` exists, `False` otherwise. |
| `len(ht)` | `__len__(self)` | Returns the total count of stored items (`self.size`). |

---

## 3. The Core Operations: Step-by-Step

### 1. Hash & Compress (`_hash`)
Every operation begins by hashing the key and compressing it to fit within our bucket array:

$$\text{bucket\_index} = \text{hash}(\text{key}) \bmod \text{capacity}$$

```python
def _hash(self, key) -> int:
    return hash(key) % self.capacity
```

### 2. Inserting or Updating (`__setitem__`)
When adding `[key, value]`:
1. Find `bucket_index = self._hash(key)`.
2. Inspect the bucket chain. If `key` already exists, **update its value in-place** and return. (Do not increment `self.size`!).
3. If `key` is not in the chain, append `[key, value]` to the bucket and increment `self.size += 1`.
4. Check if the load factor exceeds $0.75$. If so, trigger **dynamic resizing**!

### 3. Retrieving (`__getitem__`)
1. Find `bucket_index = self._hash(key)`.
2. Walk the bucket chain comparing each stored key with `==`.
3. If found, return the associated value.
4. If the chain ends without finding the key, raise a standard `KeyError(key)`.

### 4. Deleting (`__delitem__`)
1. Find `bucket_index = self._hash(key)`.
2. Walk the bucket chain. If found, remove the pair from the chain, decrement `self.size -= 1`, and return.
3. If not found, raise `KeyError(key)`.

---

## 4. The Scalability Engine: Dynamic Resizing (Rehashing)

What happens if we insert 10,000 items into a table with only 4 buckets? 

Every bucket chain would grow to an average length of $2,500$ elements. Every search would require walking 2,500 nodes—destroying our $O(1)$ constant time and collapsing our hash table into an $O(N)$ linear search!

To maintain guaranteed $O(1)$ performance, the table must **dynamically resize** whenever the load factor exceeds our threshold ($\alpha > 0.75$).

```text
Before Resize (capacity = 4, size = 4, load_factor = 1.0 > 0.75):
Buckets:  [0] ──► [A] ──► [B]
          [1] ──► [C]
          [2] ──► [D]
          [3] ──► None

After Resize (capacity = 8, size = 4, load_factor = 4 / 8 = 0.50):
All keys are REHASHED across 8 slots!
Buckets:  [0] ──► None
          [1] ──► [C]
          [2] ──► [D]
          [4] ──► [A]  <-- A moved from 0 to 4!
          [6] ──► [B]  <-- B moved from 0 to 6!
```

> [!CRITICAL]
> **Why Can't We Just Copy Over the Buckets?**
> You **cannot** simply copy the old buckets into the new array! 
> Why? Because the bucket index depends directly on `self.capacity`:
> - In a table of size 4: `hash(key) % 4` might equal **0**.
> - In a table of size 8: `hash(key) % 8` might equal **4**!
> 
> When the table doubles, the compression formula changes. Every single existing key must be extracted and **rehashed** to find its correct new slot in the doubled table.

### Amortized Analysis of Rehashing
Rehashing $N$ elements takes $O(N)$ one-time work. 
However, just like the Dynamic Arrays we studied in Week 3, doubling the capacity only occurs after $N$ successful insertions. Spreading this $O(N)$ resizing cost evenly across all $N$ operations yields an **Amortized $O(1)$ Constant Time** per insertion!

---

## 5. Complete Implementation

Here is our complete, fully featured `HashTable` implementation:

```python
class HashTable:
    INITIAL_CAPACITY = 4
    LOAD_FACTOR_THRESHOLD = 0.75

    def __init__(self, capacity: int = INITIAL_CAPACITY):
        self.capacity = capacity
        self.size = 0
        self.buckets = [[] for _ in range(self.capacity)]

    def _hash(self, key) -> int:
        """Compresses key's hash code into [0, capacity - 1]."""
        return hash(key) % self.capacity

    @property
    def load_factor(self) -> float:
        """Current ratio of elements to available slots."""
        return self.size / self.capacity

    def __setitem__(self, key, value) -> None:
        idx = self._hash(key)
        bucket = self.buckets[idx]

        # 1. Update existing key
        for pair in bucket:
            if pair[0] == key:
                pair[1] = value
                return

        # 2. Insert new key-value pair
        bucket.append([key, value])
        self.size += 1

        # 3. Check load factor for dynamic resizing
        if self.load_factor > self.LOAD_FACTOR_THRESHOLD:
            self._resize()

    def __getitem__(self, key):
        idx = self._hash(key)
        for k, v in self.buckets[idx]:
            if k == key:
                return v
        raise KeyError(f"Key '{key}' not found in hash table.")

    def __delitem__(self, key) -> None:
        idx = self._hash(key)
        bucket = self.buckets[idx]

        for i, (k, v) in enumerate(bucket):
            if k == key:
                del bucket[i]
                self.size -= 1
                return

        raise KeyError(f"Key '{key}' not found in hash table.")

    def __contains__(self, key) -> bool:
        idx = self._hash(key)
        for k, v in self.buckets[idx]:
            if k == key:
                return True
        return False

    def __len__(self) -> int:
        return self.size

    def _resize(self) -> None:
        """Doubles capacity and rehashes all existing entries."""
        old_buckets = self.buckets
        self.capacity *= 2
        self.buckets = [[] for _ in range(self.capacity)]
        self.size = 0

        # Rehash all keys into the expanded bucket array
        for bucket in old_buckets:
            for k, v in bucket:
                idx = self._hash(k)
                self.buckets[idx].append([k, v])
                self.size += 1
```

---

## Try it Yourself

**Scenario:** Verify our custom `HashTable` across all dictionary operations, trigger an automatic resize, and verify error handling.

*Your Spec:*
1. Initialize a `HashTable` with initial capacity 4.
2. Insert 3 items: `"apple"`, `"banana"`, `"cherry"`. Verify capacity is still 4.
3. Insert a 4th item (`"date"`). Verify that load factor exceeds 0.75, automatically triggering resize to capacity 8!
4. Verify that all 4 items remain accessible.
5. Update an existing key (`ht["apple"] = 99`) and confirm `len(ht)` does not increase.
6. Delete an item and verify that accessing it raises `KeyError`.

```python
# Instantiate table with initial capacity 4
ht = HashTable(capacity=4)

# 1. Insert 3 items (Load factor = 3/4 = 0.75 -> No resize yet)
ht["apple"] = 10
ht["banana"] = 20
ht["cherry"] = 30

assert len(ht) == 3
assert ht.capacity == 4
assert ht["apple"] == 10
assert ht["banana"] == 20
assert ht["cherry"] == 30

# 2. Insert 4th item -> size=4, load_factor = 4/4 = 1.0 > 0.75 -> TRIGGERS RESIZE!
ht["date"] = 40

assert len(ht) == 4
assert ht.capacity == 8  # Capacity doubled to 8!
assert ht.load_factor == 4 / 8  # 0.50

# 3. Verify all original and new keys remain accessible after rehashing
assert ht["apple"] == 10
assert ht["banana"] == 20
assert ht["cherry"] == 30
assert ht["date"] == 40

# 4. Update existing key (mutation, not insertion)
ht["apple"] = 99
assert ht["apple"] == 99
assert len(ht) == 4  # Length remains 4!

# 5. Test membership testing (__contains__)
assert "cherry" in ht
assert "watermelon" not in ht

# 6. Test deletion (__delitem__)
del ht["banana"]
assert len(ht) == 3
assert "banana" not in ht

# 7. Verify KeyError on missing key
try:
    _ = ht["banana"]
    assert False, "Should have raised KeyError!"
except KeyError:
    pass  # Expected behavior

print("All HashTable from scratch tests passed successfully!")
print(f"Final state: size={len(ht)}, capacity={ht.capacity}, load_factor={ht.load_factor}")
```
```text
All HashTable from scratch tests passed successfully!
Final state: size=3, capacity=8, load_factor=0.375
```

---

## Common Mistakes

- **Incrementing `self.size` on an update:** If someone does `ht["key"] = 1` followed by `ht["key"] = 2`, the key was updated, not added! If you increment `self.size` without checking if the key already exists, your table count and load factor will be completely corrupted.
- **Forgetting to rehash during resize:** Merely copying the sub-lists from `old_buckets` to the first half of `new_buckets` means half of your new table sits completely unused, and keys that hash to slots $\ge \text{old\_capacity}$ can never be found! Every key must be recalculated with `hash(k) % new_capacity`.
- **Hardcoding fixed array sizes:** Building a hash table without dynamic resizing creates a fragile data structure that inevitably slows to $O(N)$ when data volume grows.
- **Catching `Exception` instead of raising `KeyError`:** Standard Python dictionaries raise `KeyError` when a key is absent. Returning `None` instead of raising `KeyError` makes it impossible to differentiate between a key that doesn't exist and a key whose stored value happens to be `None`!

---

## Key Takeaway

Building a production-grade Hash Table requires combining **hash compression**, **collision handling (Separate Chaining)**, and **dynamic resizing (Rehashing)**. By implementing Python's magic dunder methods (`__setitem__`, `__getitem__`, `__delitem__`, `__contains__`), our custom class behaves identically to native dictionaries. By automatically doubling the bucket capacity and rehashing elements whenever the load factor exceeds $0.75$, we guarantee that chain lengths remain small and operations execute in **$O(1)$ Amortized Constant Time**.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Dunder Methods** | Python special methods prefixed and suffixed by double underscores (e.g., `__getitem__`) that hook into language syntax. |
| **Bucket Chain** | The collision list stored at an individual array index in a Separate Chaining hash table. |
| **Dynamic Resizing** | Automatically expanding the underlying storage array when the load factor exceeds a safe threshold. |
| **Rehashing** | The process of recomputing bucket indices for all existing keys using the new expanded capacity. |
| **Amortized $O(1)$** | An operational time complexity where an occasional expensive $O(N)$ resize is distributed across $N$ fast $O(1)$ operations. |
| **`KeyError`** | The standard Python exception raised when looking up or deleting a key that is not present in a mapping. |

---

## References

- [Python Documentation: Emulating container types (Data Model)](https://docs.python.org/3/reference/datamodel.html#emulating-container-types)
- [Introduction to Algorithms (CLRS): Chapter 11.2 - Hash Tables](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
- [Fluent Python (Luciano Ramalho): Chapter 3 - Dictionaries and Sets](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/)
