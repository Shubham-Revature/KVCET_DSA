# Sorting in Python

Python's built-in sorting capabilities are widely considered the gold standard of language design. They are blisteringly fast, rock-solid stable, and provide an expressive API that handles complex multi-attribute ordering with minimal code.

Under the hood, Python does not use a textbook Quick Sort or simple Merge Sort. Instead, it relies on **Timsort**—an algorithm specifically engineered to exploit real-world data patterns.

In this lesson, we will uncover how Timsort works, master the distinction between `list.sort()` and `sorted()`, and learn how to wield the powerful `key=` parameter.

---

## 1. Under the Hood: The Genius of Timsort

Invented by Tim Peters in 2002 for CPython, **Timsort** is a hybrid sorting algorithm that combines **Merge Sort** and **Insertion Sort**. It is so effective that it was later adopted as the default sorting engine for Java, Android, Rust, and the V8 JavaScript engine.

### The Real-World Insight
Most textbook sorting algorithms assume input data is uniformly random. But Tim Peters noticed that real-world datasets are almost never random—they naturally contain pre-sorted sequences called **runs**:
- A log file is already mostly ordered by timestamp.
- User lists are often partially alphabetized.
- Concatenating two sorted arrays creates two large sorted runs.

### How Timsort Works:
1. **Find Natural Runs:** Timsort scans the array to identify continuous ascending or strictly descending runs. (Strictly descending runs are reversed in linear time).
2. **Extend Short Runs via Insertion Sort:** If a natural run is shorter than a minimum threshold (`minrun`, typically between 32 and 64 items), Timsort pads and sorts it using an optimized **Binary Insertion Sort**.
3. **Merge Runs:** Timsort merges adjacent runs using Merge Sort, maintaining a balanced stack of runs and employing an intelligent "galloping mode" that skips large blocks of elements when one run dominates another.

### Timsort Complexity:
- **Best-Case Time:** **$\Omega(N)$ Linear Time** (when the array is already sorted or completely reversed!).
- **Worst-Case Time:** **$O(N \log N)$** (Rock-solid mathematical guarantee).
- **Auxiliary Space:** **$O(N)$** (Allocated in high-performance C memory).
- **Stability:** **100% Guaranteed Stable**.

---

## 2. `list.sort()` vs. `sorted()`

Python provides two distinct ways to sort data. Choosing the wrong one can cause subtle bugs or unnecessary memory consumption:

```python
# 1. list.sort() -> In-Place Mutation
numbers = [5, 2, 8, 1]
result = numbers.sort()

print(numbers)  # [1, 2, 5, 8]  (Mutated in-place)
print(result)   # None          (Always returns None!)


# 2. sorted() -> Out-of-Place Copy
words = ("banana", "apple", "cherry")
sorted_words = sorted(words)

print(words)         # ('banana', 'apple', 'cherry') (Untouched!)
print(sorted_words)  # ['apple', 'banana', 'cherry'] (New List!)
```

| Feature | `list.sort()` | `sorted(iterable)` |
|---|---|---|
| **Applicability** | Only available on `list` objects | Works on **any iterable** (tuples, dicts, sets, generators, strings) |
| **Output** | Modifies the list **in-place**; returns `None` | Leaves input untouched; returns a **new `list`** |
| **Memory Cost** | **$O(1)$** extra Python memory | **$O(N)$** auxiliary memory to store new list |
| **Primary Use Case** | When you want to modify existing lists without memory overhead | When input is immutable (e.g., tuple) or original order must be preserved |

---

## 3. The Power of the `key=` Parameter

In Python 2, sorting supported a `cmp` parameter that repeatedly compared pairs of elements ($A$ vs $B$). This was deprecated in Python 3 because calling a Python comparison function $O(N \log N)$ times created massive performance bottlenecks.

Python 3 uses the **Key Extraction Pattern** (the Schwartzian Transform):
- You provide a function: `key=func`.
- Python calls `func(item)` **exactly once per item** ($O(N)$ evaluations) before sorting.
- Python sorts the elements using their extracted keys in high-speed C.

```python
words = ["python", "c", "javascript", "go", "rust"]

# Sort by length of string
words.sort(key=len)
# Result: ['c', 'go', 'rust', 'python', 'javascript']
```

### Multi-Attribute Sorting with Tuples
Python compares tuples element by element from left to right. We can exploit this to perform multi-attribute sorting in a single line:

```python
students = [
    {"name": "Alice", "score": 90, "age": 20},
    {"name": "Bob",   "score": 90, "age": 19},
    {"name": "Charlie", "score": 85, "age": 22},
]

# Sort by Score DESCENDING, then by Age ASCENDING:
# We negate the numerical score to achieve descending order!
students.sort(key=lambda s: (-s["score"], s["age"]))

# Result: Bob (90, age 19) comes before Alice (90, age 20)
```

---

## 4. Custom Comparators: `functools.cmp_to_key`

Occasionally, a sorting rule cannot be reduced to an isolated key value. For example, in the classic **Largest Number** problem, you must decide whether string `a + b > b + a`.

For two-argument comparison logic, Python provides `functools.cmp_to_key`:

```python
from functools import cmp_to_key

def custom_compare(a: str, b: str) -> int:
    # Return -1 if a should come before b
    # Return  1 if b should come before a
    # Return  0 if equivalent
    if a + b > b + a:
        return -1
    elif a + b < b + a:
        return 1
    return 0

numbers = ["3", "30", "34", "5", "9"]
numbers.sort(key=cmp_to_key(custom_compare))
# Result: ['9', '5', '34', '3', '30'] -> forms largest number "9534330"
```

---

## Try it Yourself

**Scenario:** You are building an e-commerce catalog sort. Given a list of products, sort them:
1. Primary: **Category** (Alphabetical ascending).
2. Secondary: **Rating** (Descending — highest rated first).
3. Tertiary: **Price** (Ascending — cheapest first).

*Your Spec:*
- Implement the sort using a single `key=lambda ...` tuple transformation.
- Verify sorting accuracy using assertions.

```python
products = [
    {"name": "Laptop A", "category": "Electronics", "rating": 4.5, "price": 1000},
    {"name": "Laptop B", "category": "Electronics", "rating": 4.8, "price": 1200},
    {"name": "Laptop C", "category": "Electronics", "rating": 4.5, "price": 800},
    {"name": "Apple",    "category": "Groceries",   "rating": 4.9, "price": 3},
    {"name": "Banana",   "category": "Groceries",   "rating": 4.9, "price": 2},
]

# Multi-attribute sort:
# 1. category: string ascending
# 2. -rating:  float descending (negated)
# 3. price:    float ascending
products.sort(key=lambda p: (p["category"], -p["rating"], p["price"]))

# Expected Order:
# 1. Laptop B (Electronics, 4.8, 1200)
# 2. Laptop C (Electronics, 4.5, 800)   <- Cheaper than Laptop A
# 3. Laptop A (Electronics, 4.5, 1000)
# 4. Banana   (Groceries, 4.9, 2)       <- Cheaper than Apple
# 5. Apple    (Groceries, 4.9, 3)

assert products[0]["name"] == "Laptop B"
assert products[1]["name"] == "Laptop C"
assert products[2]["name"] == "Laptop A"
assert products[3]["name"] == "Banana"
assert products[4]["name"] == "Apple"

print("E-commerce multi-attribute sort verified successfully!")
for p in products:
    print(f"[{p['category']}] {p['name']}: Rating={p['rating']}, Price=${p['price']}")
```
```text
E-commerce multi-attribute sort verified successfully!
[Electronics] Laptop B: Rating=4.8, Price=$1200
[Electronics] Laptop C: Rating=4.5, Price=$800
[Electronics] Laptop A: Rating=4.5, Price=$1000
[Groceries] Banana: Rating=4.9, Price=$2
[Groceries] Apple: Rating=4.9, Price=$3
```

---

## Common Mistakes

- **Writing `my_list = my_list.sort()`:** `list.sort()` modifies in-place and returns `None`. Reassigning the return value wipes out your data, turning `my_list` into `None`! If you want a return value, use `my_list = sorted(my_list)`.
- **Negating strings for descending order:** Writing `key=lambda x: -x["name"]` will raise a `TypeError: bad operand type for unary -: 'str'`. Negation only works on numbers. To sort strings descending while sorting another field ascending, perform two successive stable sorts in reverse priority or write custom key logic.
- **Calling `sorted()` inside loops:** Repeatedly calling `sorted()` inside a loop re-allocates an entire array on every iteration. Pre-sort your data once before entering the loop.
- **Using legacy `cmp` parameters:** Python 3 removed `cmp`. Always use `key=` with tuples or `functools.cmp_to_key`.

---

## Key Takeaway

Python’s built-in sorting is powered by **Timsort**, a hybrid of Merge Sort and Insertion Sort that achieves **$\Omega(N)$ best-case time** on partially sorted inputs, guaranteed **$O(N \log N)$ worst-case time**, and **100% stability**. Use **`list.sort()`** for zero-allocation in-place list sorting, **`sorted()`** for general iterables and immutability, and tuple keys (`key=lambda x: (a, -b)`) for expressive multi-attribute ordering.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Timsort** | Python's standard sorting algorithm; an adaptive, stable hybrid of Merge Sort and Insertion Sort. |
| **Natural Run** | A contiguous sub-sequence of elements that is already sorted in ascending or strictly descending order. |
| **`list.sort()`** | A list method that rearranges elements in-place and returns `None`. |
| **`sorted()`** | A built-in function that takes any iterable and returns a new sorted list. |
| **Key Function** | A callable passed to `key=` that extracts a comparison key from each element prior to sorting. |
| **Schwartzian Transform** | An optimization pattern where keys are computed once per element rather than repeatedly during comparisons. |

---

## References

- [Python Documentation: Sorting Techniques HOWTO](https://docs.python.org/3/howto/sorting.html)
- [Tim Peters: Original Timsort Description (CPython Source)](https://github.com/python/cpython/blob/main/Objects/listsort.txt)
