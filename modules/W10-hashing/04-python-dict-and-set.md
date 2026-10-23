# Python Dict and Set: Under the Hood

The dictionary (`dict`) is the beating heart of Python. 

In Python, dictionaries are not just a convenient container for developers; they power the language's internal mechanics:
- Every module's namespace is a dictionary (`globals()`).
- Every class instance stores its attributes in a dictionary (`obj.__dict__`).
- Function local and global variables are resolved via dictionaries.

Because the entire Python language depends on dictionary speed, Python core developers have spent decades optimizing CPython's hash table implementation. 

In this lesson, we unpack the internal architecture of modern Python dictionaries (Raymond Hettinger's **Compact Dict** design), understand how `set` works, and master high-performance container tools like `defaultdict` and `Counter`.

---

## 1. The Revolution: Python's "Compact Dict"

Before Python 3.6, dictionaries consumed enormous amounts of RAM and did **not** guarantee any ordering. In Python 3.6 (and formalized as a language specification in Python 3.7), CPython adopted a revolutionary architecture proposed by PyPy and implemented by Raymond Hettinger: the **Compact Dictionary**.

### The Old Legacy Architecture (Pre-3.6)
Previously, Python dictionaries used a single, sparse table where each slot stored the hash, key pointer, and value pointer:

```text
Legacy Sparse Dict Table (Size 8):
Index    Hash        Key          Value
[ 0 ]    --          None         None
[ 1 ]    19827361    "banana"     20
[ 2 ]    --          None         None
[ 3 ]    --          None         None
[ 4 ]    82736192    "apple"      10
[ 5 ]    --          None         None
[ 6 ]    --          None         None
[ 7 ]    49182736    "cherry"     30
```

Notice the problem: to keep the load factor below $\approx 0.66$, at least **one-third of the slots had to sit completely empty**. But each empty slot still reserved 24 bytes of memory (8 bytes for the hash, 8 bytes for key pointer, 8 bytes for value pointer)! On millions of objects, this was a massive waste of memory.

### The Modern Compact Architecture (Python 3.6+)
Hettinger's innovation split the dictionary into **two separate arrays**:
1. **`indices`:** A sparse array of tiny integers (e.g., 1-byte `int8` numbers).
2. **`entries`:** A dense, packed array containing only the actual key-value records in the exact order they were inserted!

```text
Indices Array (Sparse, 1-byte integers):
[ -1,  1,  -1,  -1,  0,  -1,  -1,  2 ]
       │             │             │
       │             │             └──► Points to entries[2] ("cherry")
       │             └────────────────► Points to entries[0] ("apple")
       └──────────────────────────────► Points to entries[1] ("banana")

Entries Array (Dense, Packed):
Index    Hash        Key          Value
[ 0 ]    82736192    "apple"      10        <-- 1st inserted
[ 1 ]    19827361    "banana"     20        <-- 2nd inserted
[ 2 ]    49182736    "cherry"     30        <-- 3rd inserted
```

### The Two Historic Benefits:
1. **20% to 25% Memory Reduction:** The sparse table only stores 1-byte indices (`-1`, `0`, `1`, `2`) instead of full 24-byte entries.
2. **Guaranteed Insertion Order:** Because new elements are simply appended to the end of the dense `entries` array, **Python dictionaries automatically preserve insertion order!**

---

## 2. CPython's Open Addressing: Perturbation Probing

In Lesson 02, we learned that Linear Probing causes primary clustering, while Double Hashing requires two hash functions.

CPython uses an ingenious hybrid open-addressing formula known as **Perturbation Probing**:

$$j = (5 \cdot j + 1 + \text{perturb}) \bmod \text{capacity}$$

In every probe step:
- The index is multiplied by 5 and incremented by 1 (similar to a Linear Congruential random generator).
- A shifting variable, $\text{perturb}$, starts with the full 64-bit hash code and shifts right by 5 bits on each collision step: $\text{perturb} >>= 5$.

```text
Initial Slot:   j = hash & mask
Probe 1:        j = (5*j + 1 + (hash >> 5)) & mask
Probe 2:        j = (5*j + 1 + (hash >> 10)) & mask
...
```

### Why Perturbation Probing is Brilliant:
1. **No Clustering:** It scrambles probe sequences across the entire table, preventing the long contiguous clumps of linear probing.
2. **Utilizes High-Order Bits:** In standard modulo ($h \bmod 8$), only the last 3 bits of a hash are used. Perturbation systematically brings the top 60 bits down into the calculation, ensuring that two keys with identical low bits diverge immediately after the first collision!

---

## 3. How Python `set` Works

A Python `set` is mathematically defined as an unordered collection of unique, hashable elements.

Under the hood, **a `set` is essentially a dictionary without values!**
- It uses the exact same hash table architecture and open addressing probing.
- Each entry only contains `[hash, key]`.

> [!WARNING]
> **Sets Do NOT Guarantee Insertion Order!**
> While dictionaries preserve insertion order due to the compact `entries` array, Python's `set` implementation does **not** use the compact array layout in the same way. Never rely on the iteration order of a `set`—it is unpredictable and can vary across Python runs!

### Set Operations & Complexities

Python sets provide high-speed mathematical operations executed in optimized C:

| Operation | Python Syntax | Operator | Average Time | Description |
|---|---|---|---|---|
| **Membership** | `x in s` | — | **$O(1)$** | Checks if element exists via hash lookup. |
| **Add / Discard** | `s.add(x)` / `s.discard(x)` | — | **$O(1)$** | Inserts or safely removes element. |
| **Union** | `s1.union(s2)` | `s1 \| s2` | **$O(N_1 + N_2)$** | All elements in either set. |
| **Intersection** | `s1.intersection(s2)` | `s1 & s2` | **$O(\min(N_1, N_2))$** | Elements present in both sets. |
| **Difference** | `s1.difference(s2)` | `s1 - s2` | **$O(N_1)$** | Elements in `s1` that are not in `s2`. |
| **Symmetric Diff** | `s1.symmetric_difference(s2)` | `s1 ^ s2` | **$O(N_1 + N_2)$** | Elements in either `s1` or `s2`, but not both. |

---

## 4. Specialized Dictionary Containers: `defaultdict` and `Counter`

While native dictionaries are powerful, Python's `collections` module provides specialized subclasses engineered for common algorithmic patterns:

### 1. `collections.defaultdict`
When grouping items into lists or tallying frequencies, standard dictionaries require defensive checks to avoid `KeyError`:

```python
# The Clumsy Standard Dict Way
d = {}
if key not in d:
    d[key] = []
d[key].append(value)

# The Elegant defaultdict Way
from collections import defaultdict

d = defaultdict(list)
d[key].append(value)  # Automatically initializes missing key to []!
```

- `defaultdict(list)`: Missing keys initialize to `[]`.
- `defaultdict(int)`: Missing keys initialize to `0`.
- `defaultdict(set)`: Missing keys initialize to `set()`.

### 2. `collections.Counter`
`Counter` is a dictionary subclass specifically designed for counting hashable objects:

```python
from collections import Counter

words = ["apple", "banana", "apple", "cherry", "apple", "banana"]
counts = Counter(words)

print(counts["apple"])           # 3
print(counts["watermelon"])      # 0 (Never raises KeyError!)
print(counts.most_common(1))     # [('apple', 3)]
```

---

## 5. Memory Comparison: List vs. Set vs. Dict

Because hash tables require empty slots to maintain load factors ($\alpha \le 0.66$) and store hash codes alongside pointers, they consume substantially more memory than flat arrays:

```python
import sys

data = list(range(1000))
print("List:", sys.getsizeof(data))          # ~8,056 bytes
print("Set: ", sys.getsizeof(set(data)))     # ~32,984 bytes (4x more!)
print("Dict:", sys.getsizeof(dict.fromkeys(data))) # ~36,960 bytes (4.5x more!)
```

**The Takeaway:** Use dictionaries and sets when you need $O(1)$ lookups, membership tests, or duplicate elimination. Use lists or tuples when sequential storage and memory footprint are the priority.

---

## Try it Yourself

**Scenario:** Demonstrate dictionary insertion order, solve an Anagram Grouping problem using `defaultdict(list)`, count vote frequencies using `Counter`, and perform mathematical set queries.

*Your Spec:*
1. Prove that dictionary keys iterate in the exact order they were inserted.
2. Group anagrams into lists using `defaultdict(list)`.
3. Tally votes and extract the top winner using `Counter.most_common()`.
4. Perform Set Union, Intersection, and Difference on developer skill sets.

```python
from collections import defaultdict, Counter

# 1. Verification of Insertion Order
user_scores = {}
user_scores["Charlie"] = 85
user_scores["Alice"] = 95
user_scores["Bob"] = 90

# Order is guaranteed to be Charlie -> Alice -> Bob!
assert list(user_scores.keys()) == ["Charlie", "Alice", "Bob"]


# 2. Group Anagrams with defaultdict(list)
words = ["eat", "tea", "tan", "ate", "nat", "bat"]
grouped_anagrams = defaultdict(list)

for w in words:
    # Sorted characters form canonical key: "aet", "ant", "abt"
    canonical = "".join(sorted(w))
    grouped_anagrams[canonical].append(w)

assert grouped_anagrams["aet"] == ["eat", "tea", "ate"]
assert grouped_anagrams["ant"] == ["tan", "nat"]
assert grouped_anagrams["abt"] == ["bat"]


# 3. Frequency Analysis with Counter
votes = ["apple", "banana", "apple", "cherry", "apple", "banana"]
vote_tracker = Counter(votes)

assert vote_tracker["apple"] == 3
assert vote_tracker["banana"] == 2
assert vote_tracker["grape"] == 0  # Missing key defaults to 0
assert vote_tracker.most_common(1) == [("apple", 3)]


# 4. Set Mathematical Operations
devs_python = {"Alice", "Bob", "Charlie"}
devs_rust = {"Bob", "David", "Eve"}

both_skills = devs_python & devs_rust          # Intersection
only_python = devs_python - devs_rust          # Difference
all_engineers = devs_python | devs_rust        # Union

assert both_skills == {"Bob"}
assert only_python == {"Alice", "Charlie"}
assert all_engineers == {"Alice", "Bob", "Charlie", "David", "Eve"}

print("All Python dict, set, defaultdict, and Counter operations verified successfully!")
```
```text
All Python dict, set, defaultdict, and Counter operations verified successfully!
```

---

## Common Mistakes

- **Mutating a dictionary while iterating over it:** Writing `for k in my_dict: if condition: del my_dict[k]` will crash immediately with `RuntimeError: dictionary changed size during iteration`. To delete items while looping, iterate over a snapshot copy of the keys: `for k in list(my_dict.keys()):`.
- **Assuming sets preserve insertion order:** Dictionaries preserve insertion order in Python 3.7+; sets do **not**. If you need an ordered set, use `dict.fromkeys(iterable)` or `collections.OrderedDict`.
- **Using mutable objects in sets or as dict keys:** Attempting `s.add([1, 2])` or `d[[1, 2]] = 3` raises `TypeError: unhashable type: 'list'`. Elements must be immutable (e.g., `(1, 2)`).
- **Redundant membership checking before accessing `defaultdict`:** Writing `if key in d: d[key].append(x)` defeats the entire purpose of `defaultdict`. Simply write `d[key].append(x)`.

---

## Key Takeaway

Python's **`dict`** is powered by Raymond Hettinger's **Compact Dict** design: a sparse 1-byte `indices` array combined with a dense, packed `entries` array. This saves 20% to 25% memory and guarantees **insertion order**. CPython resolves collisions using **perturbation open addressing**, mixing high-order hash bits to prevent clustering. **`set`** uses the same table engine without values, offering $O(1)$ membership and set algebra. For common algorithmic patterns, **`defaultdict`** and **`Counter`** eliminate boilerplate and prevent `KeyError` exceptions.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Compact Dict** | Python 3.6+ dictionary architecture separating sparse indices from dense entries to save memory and preserve order. |
| **Perturbation Probing** | CPython's open addressing formula that shifts upper hash bits into the probe sequence to avoid clustering. |
| **`set`** | A built-in container holding unique, immutable elements with $O(1)$ average membership lookups. |
| **`defaultdict`** | A dictionary subclass that calls a factory function to provide default values for missing keys. |
| **`Counter`** | A dictionary subclass optimized for tallying occurrences of hashable items. |
| **`RuntimeError` (Iteration)** | The error raised when adding or deleting keys from a dictionary while actively iterating over it. |

---

## References

- [Python Documentation: Mapping Types — dict](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)
- [Python Documentation: Set Types — set, frozenset](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset)
- [Raymond Hettinger: Modern Dictionaries by Keynote (PyCon 2017)](https://www.youtube.com/watch?v=p33CVV29OG8)
- [CPython Source: Objects/dictobject.c](https://github.com/python/cpython/blob/main/Objects/dictobject.c)
