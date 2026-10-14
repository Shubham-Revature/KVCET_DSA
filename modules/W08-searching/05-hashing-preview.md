# Hashing Preview: The Holy Grail of $O(1)$ Search

Throughout this module, we explored the progression of searching algorithms:
- **Linear Search:** Checks items one by one in **$O(N)$** time. It works on unsorted data, but slows down dramatically as datasets grow.
- **Binary Search:** Achieves blazing-fast **$O(\log N)$** time by halving the search space on each step. But it comes with a strict price: the data **must be sorted**.

This raises the ultimate computer science question:
*Can we search for an item in instant **$O(1)$ Constant Time** without having to sort the data first?*

The answer is yes. The solution is **Hashing**—the underlying engine powering Python `dict` and `set`, and the topic of our dedicated deep dive in Week 10.

---

## 1. The Core Intuition: Direct Addressing

Imagine a hotel with 100 rooms numbered `0` through `99`. 
- If a guest is assigned Room `42`, how long does the front desk take to find their room key?
- Do they check Room 0, Room 1, Room 2... (Linear Search)? **No.**
- Do they binary search room numbers? **No.**
- They walk straight to Hook `42` in instant **$O(1)$ time**!

This is called **Direct Addressing**. If every piece of data were simply a small positive integer, we could use a standard array where the data value itself is the index:

```text
Value: 42  ─── Direct Index ───►  array[42]   (O(1) Access!)
```

### The Real-World Problem: Non-Integer & Giant Keys
In the real world, keys are not small clean integers from 0 to 99:
- Usernames: `"alice_wonderland"`, `"bob_developer"`
- Credit Card Numbers: `4532 8912 3456 7890` (16 digits)
- Social Security Numbers / Passports

We cannot allocate an array of $10^{16}$ slots in computer memory just to store a credit card number. How do we map arbitrary strings and huge numbers into small array slots?

---

## 2. The Magic Mechanism: The Hash Function

Enter the **Hash Function**.

A hash function is a deterministic mathematical formula that takes any arbitrary key (a string, float, integer, or tuple) and transforms it into a fixed-size integer:

```text
"alice"   ──► [ Hash Function: hash("alice") % 8 ]  ──► Index 3
"bob"     ──► [ Hash Function: hash("bob")   % 8 ]  ──► Index 7
"charlie" ──► [ Hash Function: hash("charlie") % 8 ]──► Index 1
```

Once the hash function converts the key into an index, the computer jumps straight to that slot in memory. This allows checking membership (`"alice" in my_set`) in **$O(1)$ Constant Time**!

```python
# Python's built-in hash() function
print(hash("alice"))       # e.g., 3871928491823719 (some large integer)
print(hash("alice") % 8)   # Maps directly to an array slot between 0 and 7!
```

---

## 3. The Fundamental Trade-Off: Time vs. Space

If Hashing delivers $O(1)$ lookups without requiring sorted data, why don't we use it for absolutely everything?

Because of the **Time-Space Trade-Off**:

| Factor | Arrays / Lists | Hash Tables (`set` / `dict`) |
|---|---|---|
| **Search Time** | $O(N)$ (or $O(\log N)$ if sorted) | **$O(1)$ Average** |
| **Memory Overhead** | **Low** (compact, contiguous elements) | **High** (requires sparse tables with empty slots) |
| **Ordering** | Preserves order (indices 0, 1, 2...) | No inherent numeric ordering |
| **Range Queries** | Fast on sorted arrays (e.g., values between 10 and 50) | Impossible without scanning every element |
| **Worst-Case Search** | $O(N)$ | $O(N)$ (if hash collisions degrade the table) |

1. **Memory Cost:** To avoid "collisions" (two keys mapping to the exact same slot), a Hash Table must keep 30% to 50% of its memory slots empty at all times.
2. **Loss of Order:** A Hash Table scatters items across memory. You cannot perform range queries like *"find all items between $10$ and $50$"* or find the "floor" and "ceiling" without checking everything.

---

## 4. The Grand Comparison: Three Searching Paradigms

Throughout your software engineering career, you will choose among three core searching strategies:

| Strategy | Time Complexity | Prerequisite | Space Overhead | When to Use |
|---|---|---|---|---|
| **Linear Search** | **$O(N)$** | None (Any iterable) | **$O(1)$** | Small lists ($N < 50$), unsorted data, single one-off queries. |
| **Binary Search** | **$O(\log N)$** | **Sorted Data** | **$O(1)$** | Large static datasets, range queries, finding floor/ceiling, solution-space optimization. |
| **Hash Lookup** | **$O(1)$** | Hashable Keys | **$O(N)$** | Frequent repeated queries, key-value mappings, fast existence checks. |

---

## 5. The Evolutionary Classic: Two-Sum Across Three Paradigms

The famous **Two-Sum** problem perfectly illustrates how our three searching paradigms transform algorithmic performance:

> **Problem:** Given an array of numbers and a `target`, find two numbers that sum up to `target`.
> Example: `nums = [2, 7, 11, 15]`, `target = 9` $\implies$ `(2, 7)`.

### Evolution 1: Brute Force (Linear Search)
For every number, linearly search through the rest of the array for its complement (`target - num`).
- **Time Complexity:** **$O(N^2)$** Quadratic Time.

### Evolution 2: Sort + Binary Search
Sort the array ($O(N \log N)$), then for each element, binary search for its complement ($O(\log N)$).
- **Time Complexity:** **$O(N \log N)$** Linearithmic Time.

### Evolution 3: Hash Map Lookup
As you iterate through the array once, check if the complement already exists in a Hash Table in **$O(1)$** time. If not, insert the current number and continue.
- **Time Complexity:** **$O(N)$** Linear Time!

---

## Try it Yourself

**Scenario:** Implement the optimal $O(N)$ Two-Sum solver using a Python dictionary (Hash Map) to find indices of the two elements that sum to `target`.

*Your Spec:*
1. Initialize an empty dictionary `seen = {}` (mapping `value -> index`).
2. Enumerate through `nums`.
3. Compute `complement = target - num`.
4. If `complement` is already in `seen`, return `(seen[complement], index)`.
5. Otherwise, store `seen[num] = index`.
6. Return `None` if no pair exists.

```python
def two_sum_hash(nums: list[int], target: int) -> tuple[int, int] | None:
    seen = {}  # Hash Map storing: {number_value: original_index}

    for current_index, num in enumerate(nums):
        complement = target - num

        # O(1) Instant Hash Lookup!
        if complement in seen:
            return (seen[complement], current_index)

        seen[num] = current_index

    return None

# Test 1: Standard match
nums1 = [2, 7, 11, 15]
assert two_sum_hash(nums1, 9) == (0, 1)

# Test 2: Unsorted array
nums2 = [3, 2, 4]
assert two_sum_hash(nums2, 6) == (1, 2)

# Test 3: No valid pair
nums3 = [1, 2, 3]
assert two_sum_hash(nums3, 10) is None

print("All Two-Sum hash lookup test cases passed successfully!")
```
```text
All Two-Sum hash lookup test cases passed successfully!
```

---

## Common Mistakes

- **Assuming all Python objects are hashable:** In Python, only **immutable** objects can be hashed. Numbers, strings, and tuples are hashable. Mutable structures like `list`, `set`, and `dict` are **unhashable**; trying to use a `list` as a dictionary key or set element will trigger an immediate `TypeError: unhashable type: 'list'`.
- **Assuming Hashing is strictly $O(1)$ in the worst case:** Under malicious inputs or poor hash functions, multiple keys can collide in the same bucket (a **hash collision**), degrading lookup time from $O(1)$ to $O(N)$.
- **Using Hash Tables when ordered operations are needed:** If your application needs to find the "next smallest element" or iterate in sorted order, Hash Tables offer no advantage over sorted arrays or Binary Search Trees.
- **Ignoring memory overhead:** Hash tables allocate extra bucket space to prevent collisions. For memory-constrained embedded systems, compact sorted arrays with Binary Search are often preferred.

---

## Key Takeaway

**Hashing** achieves the holy grail of search algorithms: **$O(1)$ Constant Time lookups** without requiring sorted data. By passing keys through a mathematical **Hash Function**, items are directly mapped to specific memory buckets. This speed comes at the cost of **$O(N)$ extra memory overhead** and the loss of sorted order. Hashing completes our searching toolkit and sets the stage for our deep dive into Hash Tables in Week 10!

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Hashing** | The process of mapping arbitrary data of any size to fixed-size values using a mathematical function. |
| **Hash Function** | A deterministic function that converts a key into a numeric index. |
| **Direct Addressing** | A technique where each element's key directly corresponds to an array index for $O(1)$ access. |
| **Hash Collision** | A scenario where two different keys produce the exact same hash index. |
| **Hash Table** | A data structure that stores key-value pairs using a hash function for $O(1)$ average-time operations. |
| **Hashable** | An object with a fixed hash value that never changes during its lifetime (in Python, immutable types like `int`, `str`, `tuple`). |

---

## References

- [Python Documentation: Mapping Types — dict](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)
- [Python Documentation: Set Types — set, frozenset](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset)
- [GeeksforGeeks: Introduction to Hashing](https://www.geeksforgeeks.org/introduction-to-hashing-data-structure-and-algorithm-tutorials/)
