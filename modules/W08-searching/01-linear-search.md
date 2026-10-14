# Linear Search

Before we can organize, filter, or analyze data, we must answer the most fundamental question in computing: **"Is this item here, and if so, where?"** 

The simplest, most intuitive way to find an item is **Linear Search** (also known as Sequential Search). It requires zero prerequisites, works on any collection regardless of order, and serves as the baseline benchmark against which all advanced searching algorithms are measured.

---

## 1. How Linear Search Works

Imagine dropping your car keys in a long hallway lined with closed lockers. You do not know which locker holds your keys, and the lockers are not arranged in any particular order. What is your only option? 

You start at Locker 0, open it, and inspect the contents. If your keys are not there, you move to Locker 1, then Locker 2, and continue checking one locker after another until you either find your keys or reach the very last locker.

In computer science, **Linear Search** inspects every element in a sequence sequentially from left to right:

```text
Target: 42
Array:  [ 15,   8,  42,  90,   3 ]
           │    │    │
Step 1:   15 ≠ 42 (Move right)
Step 2:         8 ≠ 42 (Move right)
Step 3:             42 == 42  ==> FOUND at Index 2! (Stop searching)
```

### The Early-Exit Principle

Notice that as soon as the algorithm finds `42` at index 2, it **stops immediately** and returns the index. Continuing to scan indices 3 and 4 would be a complete waste of CPU cycles. This is known as **early termination** (or early exit).

---

## 2. Complexity Analysis: Best, Average, and Worst Case

Because Linear Search checks elements sequentially, its running time depends heavily on where the target happens to be located:

| Case | Scenario | Time Complexity | Explanation |
|---|---|---|---|
| **Best Case** | Target is at index `0` | **$\Omega(1)$ Constant Time** | The very first item checked is our target. Found in 1 comparison! |
| **Worst Case** | Target is at the final index or not present | **$O(N)$ Linear Time** | We must examine all $N$ elements before concluding the search. |
| **Average Case** | Target is randomly located | **$\Theta(N)$ Linear Time** | On average, we inspect approximately $N / 2$ elements, which simplifies to $\Theta(N)$. |
| **Space Complexity** | Iterative traversal | **$O(1)$ Auxiliary Space** | Only requires a single loop index variable; no extra memory allocated. |

---

## 3. When is Linear Search Actually the Best Choice?

In later lessons this week, we will learn algorithms like Binary Search that find items in lightning-fast $O(\log N)$ time. You might wonder: *why bother with Linear Search at all?*

Linear Search remains indispensable in several critical real-world scenarios:

1. **Unsorted Data:** Binary Search strictly requires data to be sorted. If you have an arbitrary, unsorted list of records, you **cannot** use Binary Search without sorting it first.
2. **One-Time Searches:** Sorting an unsorted array takes $O(N \log N)$ time. If you only plan to search the list once, running a single $O(N)$ Linear Search is dramatically faster than sorting first ($O(N \log N) + O(\log N) > O(N)$)!
3. **Small Datasets:** For tiny lists (e.g., $N \le 50$), Linear Search is so simple and cache-friendly that the overhead of advanced algorithms provides no practical benefit.
4. **Linked Lists:** Linked Lists do not support instant $O(1)$ random indexing (`list[mid]`). Even if a Linked List is sorted, you cannot jump to the middle element in $O(1)$ time, making sequential scanning the natural choice.

---

## 4. Under the Hood: Python's Built-In Searches

Python provides several expressive built-in keywords and methods for searching lists:

```python
fruits = ["apple", "banana", "cherry", "date"]

# 1. Membership test
"cherry" in fruits        # Returns True

# 2. Finding index
fruits.index("cherry")    # Returns 2

# 3. Counting occurrences
fruits.count("cherry")    # Returns 1
```

> [!IMPORTANT]
> **Don't Be Fooled by Elegant Syntax!**
> Writing `"cherry" in fruits` or `fruits.index("cherry")` looks like a single, instantaneous line of code. Under the hood in CPython, both operators execute a classic, iterative **$O(N)$ Linear Search**. Do not assume built-ins are $O(1)$ just because they are written in one line!

---

## 5. Algorithmic Optimization: The Sentinel Search

In standard Linear Search, every iteration performs **two comparisons**:
1. Has the index exceeded the bounds of the array? (`i < len(arr)`)
2. Does the current element match the target? (`arr[i] == target`)

In performance-critical low-level code, engineers historically used a technique called **Sentinel Linear Search**. By temporarily placing the target value at the very end of the array (as a "sentinel" guard), you guarantee the search will always succeed, eliminating the need to check bounds on every single iteration! Once found, you simply verify whether the match occurred before the sentinel.

While modern compilers and Python handle bounds checks efficiently, the Sentinel technique is a classic example of algorithmic optimization that demonstrates how every CPU comparison matters.

---

## Try it Yourself

**Scenario:** You are building an audit log monitor. Given an unsorted stream of transaction IDs, implement a linear search function that returns the 0-based index of a flagged transaction, or `-1` if it is not found.

*Your Spec:*
1. Iterate through the array using `enumerate` to track both index and value.
2. If the current value matches `target`, return the index immediately (early exit).
3. If the loop completes without finding the target, return `-1`.

```python
def linear_search(transactions: list[int], target: int) -> int:
    for index, transaction_id in enumerate(transactions):
        if transaction_id == target:
            return index  # Early exit on match
    return -1  # Target not present in log

# Test Log
audit_log = [1042, 9081, 4421, 3105, 8812, 5500]

# 1. Target present in the middle
assert linear_search(audit_log, 3105) == 3

# 2. Target present at the very beginning (Best case: 1 comparison)
assert linear_search(audit_log, 1042) == 0

# 3. Target absent from the log (Worst case: scans all 6 items)
assert linear_search(audit_log, 9999) == -1

print("All linear search test cases passed successfully!")
```
```text
All linear search test cases passed successfully!
```

---

## Common Mistakes

- **Assuming `item in list` is $O(1)$:** A frequent blunder in technical interviews is using `in` inside a loop over a list (e.g., `for x in A: if x in B:`), accidentally creating an unintentional $O(N^2)$ quadratic bottleneck because `x in B` executes a linear search every time. If you need $O(1)$ membership testing, use a `set` or `dict`.
- **Sorting before a single search:** Never sort an array just to run Binary Search if you only need to perform a single query. Sorting takes $O(N \log N)$, whereas a straightforward Linear Search finishes in $O(N)$.
- **Missing early termination:** Never continue looping once the target has been identified (unless specifically counting occurrences or collecting all duplicate indices). Returning or breaking the instant a match is found preserves average-case efficiency.
- **Handling missing items with exceptions:** In Python, calling `list.index(target)` raises a `ValueError` if the item is not present. Always wrap it in a `try/except ValueError` block or verify with `if target in list:` if you rely on the built-in.

---

## Key Takeaway

**Linear Search** is the simplest searching algorithm: it sequentially checks every element in a collection from beginning to end until the target is found or the collection is exhausted. It runs in **$O(1)$ Best Case** and **$O(N)$ Worst / Average Case** time while using **$O(1)$ Space**. Because it requires no sorted order and has minimal overhead, it is the ideal tool for unsorted data, small collections, and one-off queries.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Linear Search** | An algorithm that searches for a target value within a list by examining each element in sequence until a match is found. |
| **Sequential Search** | Another widely used term for Linear Search. |
| **Early Exit** | Terminating a loop immediately upon finding the target to avoid redundant computations. |
| **Sentinel Search** | A variation of linear search that places the target at the boundary to eliminate per-iteration bounds checking. |
| **Membership Testing** | Checking whether an element exists within a collection (e.g., using Python's `in` operator). |

---

## References

- [GeeksforGeeks: Linear Search Algorithm](https://www.geeksforgeeks.org/linear-search/)
- [Python Documentation: Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations)
