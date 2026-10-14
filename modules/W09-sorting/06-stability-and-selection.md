# Stability and Selection

When engineers study sorting, they often focus entirely on Big-O time complexity. However, in real-world systems, two other fundamental concepts dictate whether a sorting implementation is correct and optimal:

1. **Stability:** Does the sort preserve the original relative order of equivalent items?
2. **Selection (Order Statistics):** Do we actually need to sort all $N$ items if we only care about finding the single $k$-th smallest item (or the median)?

Understanding these two concepts prevents catastrophic ordering bugs in data pipelines and saves massive computational resources when full sorting is unnecessary.

---

## 1. What is Stability?

A sorting algorithm is defined as **Stable** if elements with identical keys appear in the sorted output in the **exact same relative order** as they appeared in the input.

```text
Input:   [ (Card: 5, Suit: ♠),  (Card: 3, Suit: ♥),  (Card: 5, Suit: ♦) ]
                                   ▲                    ▲
                                5♠ first              5♦ second

Stable Sort (by number):
         [ (3, ♥),  (5, ♠),  (5, ♦) ]   ──► 5♠ still precedes 5♦!

Unstable Sort (by number):
         [ (3, ♥),  (5, ♦),  (5, ♠) ]   ──► 5♦ jumped ahead of 5♠! Order corrupted.
```

### Why Stability Matters: Multi-Column Sorting
Imagine viewing a spreadsheet of employees. 
1. You first sort the spreadsheet alphabetically by **First Name**.
2. You then sort the spreadsheet alphabetically by **Department**.

If your second sort is **Stable**, all employees within the `"Engineering"` department will automatically remain sorted by their First Name! 
If your second sort is **Unstable**, the alphabetical first-name ordering you previously established will be scrambled into random chaos.

### Which Algorithms are Stable?
- **Stable Sorts:** Merge Sort, Insertion Sort, Bubble Sort, Counting Sort, Timsort (Python's built-in).
- **Unstable Sorts:** Quick Sort, Heap Sort, Selection Sort.

> [!TIP]
> **How to Make Any Unstable Sort Stable:**
> If you are forced to use an unstable sort (like Quick Sort), you can guarantee stability by appending each item's original 0-based index to its key: `(original_key, original_index)`. When primary keys tie, the algorithm falls back to comparing indices, preserving initial order!

---

## 2. The Selection Problem: Finding the $k$-th Element

Suppose you run a gaming platform with 10 million daily active users, and you need to find the **median score** or the **100th highest score**:
- **The Naive Approach:** Sort all 10 million scores ($O(N \log N)$), then read `scores[k]`.
- **The Engineer's Question:** Why waste CPU cycles organizing all 10 million elements if we only care about a single rank?

This is known as the **Selection Problem**. We can solve it in **$O(N)$ Average Linear Time** using **Quickselect**.

---

## 3. Quickselect: Hoare's Selection Algorithm

Discovered by Tony Hoare (the creator of Quick Sort), **Quickselect** uses the exact same `partition()` function as Quick Sort, but with a brilliant shortcut:

- In **Quick Sort**, after the pivot lands in its final sorted index `p`, you recursively sort **BOTH** the left half and the right half ($O(N \log N)$).
- In **Quickselect**, you compare the pivot index `p` to your target index `k`:
  1. **`p == k`:** The pivot landed directly on index `k`! We found our answer! **Stop immediately.**
  2. **`p > k`:** The target index lives in the left partition. **Discard the right half entirely!** Only search the left partition.
  3. **`p < k`:** The target index lives in the right partition. **Discard the left half entirely!** Only search the right partition.

```text
Find k=3 in array of size 8:
Pass 1: Partition array of size 8 (Pivot lands at p=4).
        Target k=3 < 4 ──► Discard right half! (Only search left 4 items)
Pass 2: Partition sub-array of size 4 (Pivot lands at p=2).
        Target k=3 > 2 ──► Discard left half! (Only search right 1 item)
Pass 3: Partition sub-array of size 1 (Pivot lands at p=3).
        p == k ──► Target found in O(N) total time!
```

### The $O(N)$ Mathematical Proof
Because Quickselect only explores **one** branch at each recursive level:
$$\text{Total Work} = N + \frac{N}{2} + \frac{N}{4} + \frac{N}{8} + \dots = N \left(1 + \frac{1}{2} + \frac{1}{4} + \dots\right) \le 2N = \mathbf{O(N)}$$

Instead of doing $O(N \log N)$ work, Quickselect runs in **$O(N)$ Average Linear Time**!

---

## Try it Yourself

**Scenario:** Implement Quickselect to find the $k$-th smallest element (0-indexed) in an unsorted array in $O(N)$ time, and demonstrate multi-key stable sorting in Python.

*Your Spec:*
1. Implement `quickselect(arr, k)` using in-place Lomuto partitioning.
2. Find the median (index 3) of an unsorted list of 7 integers.
3. Demonstrate Python's stable `sort(key=...)` on structured records.

```python
# --- 1. Quickselect (k-th Smallest in O(N) Average Time) ---
def partition(arr: list[int], low: int, high: int) -> int:
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1


def quickselect(arr: list[int], k: int) -> int:
    low, high = 0, len(arr) - 1

    while low <= high:
        p_idx = partition(arr, low, high)

        if p_idx == k:
            return arr[p_idx]
        elif p_idx > k:
            high = p_idx - 1  # Discard right half
        else:
            low = p_idx + 1   # Discard left half

    return -1


# Test Quickselect
nums = [38, 27, 43, 3, 9, 82, 10]
# Sorted: [3, 9, 10, 27, 38, 43, 82]
# Index 0 (Smallest): 3
# Index 3 (Median):   27
# Index 6 (Largest):  82

assert quickselect(list(nums), 0) == 3
assert quickselect(list(nums), 3) == 27
assert quickselect(list(nums), 6) == 82
print("Quickselect successfully identified k-th elements in O(N) time!")


# --- 2. Demonstrating Stability in Multi-Level Sorting ---
students = [
    ("Alice", "B"),
    ("Bob", "A"),
    ("Charlie", "B"),
    ("David", "A"),
]

# Sort by Grade (A before B). Because Python sort is STABLE,
# Bob will remain before David, and Alice before Charlie!
students.sort(key=lambda s: s[1])

assert students == [
    ("Bob", "A"),
    ("David", "A"),
    ("Alice", "B"),
    ("Charlie", "B"),
]
print("Stable multi-key sorting verified successfully!")
```
```text
Quickselect successfully identified k-th elements in O(N) time!
Stable multi-key sorting verified successfully!
```

---

## Common Mistakes

- **Sorting an entire array to find top-$k$ or median:** Calling `arr.sort()` to find the 5th smallest element of 1,000,000 items is an unnecessary $O(N \log N)$ operation. Use Quickselect (or a Heap of size $k$) for $O(N)$ execution.
- **Assuming Quickselect sorts the array:** Quickselect only guarantees that the element at index `k` is in its correct sorted position (and elements before it are $\le$ and elements after it are $\ge$). The surrounding partitions remain unsorted!
- **Off-by-one errors with $k$:** Remember whether the problem specifies 0-indexed or 1-indexed rank. The "1st smallest" item is at index `k = 0`. The "3rd smallest" item is at index `k = 2`.
- **Breaking stability with arbitrary ties:** If an algorithm uses `<` instead of `<=` when moving elements during merge or insertion, it will inadvertently swap equivalent elements and destroy stability.

---

## Key Takeaway

- **Stability** guarantees that equal elements maintain their original relative input order. It is crucial for multi-column sorting pipelines, data preservation, and is a hallmark of Merge Sort, Insertion Sort, and Python's Timsort.
- **Quickselect** solves the Selection Problem by applying Quick Sort's partitioning to discard half the array on every step. By focusing exclusively on the branch containing index $k$, it finds medians and order statistics in **$O(N)$ Average Linear Time** without paying the full $O(N \log N)$ cost of sorting.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Stability** | A property where elements with equal sorting keys retain their original relative order after sorting. |
| **Order Statistic** | The $k$-th smallest element in a statistical dataset or array. |
| **Median** | The middle value in an ordered sequence of numbers (the $\lfloor N/2 \rfloor$-th order statistic). |
| **Quickselect** | A selection algorithm based on Quick Sort's partition that finds the $k$-th smallest element in $O(N)$ average time. |
| **Multi-Level Sort** | Sorting records sequentially across multiple attributes (e.g., Department, then Salary, then Name). |

---

## References

- [GeeksforGeeks: Stability in Sorting Algorithms](https://www.geeksforgeeks.org/stability-in-sorting-algorithms/)
- [GeeksforGeeks: Quickselect Algorithm](https://www.geeksforgeeks.org/quickselect-algorithm/)
- [Introduction to Algorithms (CLRS): Chapter 9 - Medians and Order Statistics](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
