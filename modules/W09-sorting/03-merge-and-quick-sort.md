# Merge Sort and Quick Sort

While the Elementary Sorts struggle at $O(N^2)$, modern computing relies on two algorithmic titans: **Merge Sort** and **Quick Sort**.

Both algorithms conquer large datasets using the **Divide and Conquer** paradigm we studied in Week 7. By breaking an array of $N$ elements into smaller subproblems, both algorithms break through the quadratic barrier to achieve the optimal **$O(N \log N)$ Linearithmic Time**.

However, their internal philosophies are diametrically opposed:
- **Merge Sort** divides easily and does the heavy lifting on the way **up** (during the Merge step).
- **Quick Sort** does the heavy lifting on the way **down** (during the Partition step) and combines effortlessly.

---

## 1. Merge Sort: The Reliable Workhorse

Invented by John von Neumann in 1945, **Merge Sort** follows three strict steps:
1. **Divide:** Find the midpoint and split the array blindly in half.
2. **Conquer:** Recursively sort the left half and right half until reaching single-element base cases ($N=1$).
3. **Combine (Merge):** Zipper-merge the two sorted halves back together into a single sorted array in $O(N)$ time.

```text
Split Phase (Divide):
                   [ 38, 27, 43, 3, 9, 82, 10 ]
                    ╱                        ╲
            [ 38, 27, 43, 3 ]            [ 9, 82, 10 ]
             ╱            ╲               ╱         ╲
         [ 38, 27 ]     [ 43, 3 ]      [ 9, 82 ]   [ 10 ]
          ╱      ╲       ╱     ╲        ╱     ╲       │
        [38]    [27]   [43]    [3]    [9]    [82]   [10]
──────────────────────────────────────────────────────────
Merge Phase (Combine):
        [ 27, 38 ]     [ 3, 43 ]       [ 9, 82 ]    [ 10 ]
            ╲              ╱               ╲          ╱
          [ 3, 27, 38, 43 ]              [ 9, 10, 82 ]
                    ╲                         ╱
                 [ 3, 9, 10, 27, 38, 43, 82 ]
```

### The Two-Pointer Merge Engine
The heart of Merge Sort is the merge function. Because both sublists are already sorted, we maintain two pointers at the start of each sublist, compare their values, and append the smaller value:

```python
def merge_sort(arr: list[int]) -> list[int]:
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left_half = merge_sort(arr[:mid])
    right_half = merge_sort(arr[mid:])

    return merge(left_half, right_half)


def merge(left: list[int], right: list[int]) -> list[int]:
    merged = []
    i = j = 0

    # Zipper-merge the two sorted sublists
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:    # <= preserves stability!
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1

    # Append any remaining elements
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged
```

### Merge Sort Strengths & Trade-offs
- **Guaranteed Consistency:** Best, Average, and Worst cases are all **$\Theta(N \log N)$**. It never degrades on adversarial inputs!
- **Stable:** Equal elements preserve their original relative order.
- **The Catch ($O(N)$ Space):** Merging standard arrays requires allocating auxiliary memory buffers. It is **not in-place**.
- **Ideal For:** **Linked Lists** (where merging takes $O(1)$ extra space via pointer re-linking) and **External Sorting** (sorting terabytes of data too large to fit in RAM).

---

## 2. Quick Sort: The Speed Demon

Discovered by Tony Hoare in 1959, **Quick Sort** reverses Merge Sort's philosophy. It does all the hard work during the division phase:

1. **Pick a Pivot:** Choose an element from the array to act as a benchmark.
2. **Partition:** Rearrange the array in-place so that:
   - All elements $\le$ the pivot move to the left.
   - All elements $>$ the pivot move to the right.
   - The pivot is now locked in its **final, exact sorted index**.
3. **Recurse:** Recursively quick-sort the left partition and right partition.

```text
Array:   [ 10,  80,  30,  90,  40,  50,  70 ]     (Pivot = 70)
                                         ▲
Partition Step:
Move elements <= 70 to left, > 70 to right:
         [ 10,  30,  40,  50 | 70 | 90,  80 ]
                                ▲
                 Pivot 70 is permanently locked!
Recurse:
QuickSort left:  [ 10, 30, 40, 50 ]
QuickSort right: [ 90, 80 ]
```

### In-Place Partitioning (Lomuto Partition)

```python
def quick_sort(arr: list[int], low: int, high: int) -> None:
    if low < high:
        # Partition index: pivot is now in its permanent position
        p_idx = partition(arr, low, high)

        # Recursively sort elements before and after partition
        quick_sort(arr, low, p_idx - 1)
        quick_sort(arr, p_idx + 1, high)


def partition(arr: list[int], low: int, high: int) -> int:
    pivot = arr[high]  # Choosing last element as pivot
    i = low - 1        # Boundary of elements <= pivot

    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    # Place pivot in its correct position
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

### The $O(N^2)$ Pivot Trap & How to Avoid It
- **Average Time:** **$O(N \log N)$** with exceptional cache locality and tiny constant factors.
- **Worst-Case Time ($O(N^2)$):** If the pivot is consistently chosen as the smallest or largest element (e.g., picking the last element of an already-sorted array), the partitions split into $0$ and $N-1$ elements. The recursion tree degenerates into a linear chain of depth $N$!
- **The Fix:** **Randomized Pivot** or **Median-of-Three** (inspecting first, middle, and last elements and choosing the median as the pivot). This reduces the probability of hitting worst-case $O(N^2)$ to virtually zero in practice.

---

## 3. The Ultimate Showdown: Merge Sort vs. Quick Sort

| Property | Merge Sort | Quick Sort |
|---|---|---|
| **Strategy** | Hard work during **Combine** | Hard work during **Divide** |
| **Best-Case Time** | **$\Omega(N \log N)$** | **$\Omega(N \log N)$** |
| **Average-Case Time** | **$\Theta(N \log N)$** | **$\Theta(N \log N)$** |
| **Worst-Case Time** | **$O(N \log N)$** (Rock solid) | **$O(N^2)$** (Unbalanced pivot) |
| **Auxiliary Space** | $O(N)$ (Extra memory buffer) | **$O(\log N)$** (Recursive call stack only) |
| **In-Place?** | No (for arrays) | **Yes** |
| **Stability** | **Stable** | **Unstable** |
| **Hardware Cache** | Moderate (merges across buffers) | **Superior (local memory access)** |
| **Best Used For** | Linked lists, guaranteed latency, external files | General-purpose in-memory array sorting |

---

## Try it Yourself

**Scenario:** Implement and verify both Merge Sort and in-place Quick Sort on an unsorted dataset.

*Your Spec:*
1. Run `merge_sort` returning a new sorted list.
2. Run `quick_sort` sorting an identical array in-place.
3. Assert that both yield identical, properly sorted results.

```python
# --- Merge Sort ---
def merge_sort(arr: list[int]) -> list[int]:
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    merged = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged

# --- Quick Sort ---
def quick_sort(arr: list[int], low: int, high: int) -> None:
    if low < high:
        pivot = arr[high]
        i = low - 1
        for j in range(low, high):
            if arr[j] <= pivot:
                i += 1
                arr[i], arr[j] = arr[j], arr[i]
        arr[i + 1], arr[high] = arr[high], arr[i + 1]
        p_idx = i + 1
        
        quick_sort(arr, low, p_idx - 1)
        quick_sort(arr, p_idx + 1, high)

# Test Data
original = [38, 27, 43, 3, 9, 82, 10]
expected = sorted(original)

# 1. Merge Sort test
merged_result = merge_sort(original)
assert merged_result == expected

# 2. Quick Sort test (in-place)
quick_arr = list(original)
quick_sort(quick_arr, 0, len(quick_arr) - 1)
assert quick_arr == expected

print("Both Merge Sort and Quick Sort executed successfully!")
```
```text
Both Merge Sort and Quick Sort executed successfully!
```

---

## Common Mistakes

- **Slicing inside recursive calls:** Writing `quick_sort(arr[:mid])` creates a copy and breaks in-place mutation. Always pass boundary indices (`low`, `high`) for in-place algorithms.
- **Picking a fixed pivot on sorted data:** Choosing `arr[0]` or `arr[-1]` as the pivot in Quick Sort without randomization makes already-sorted data run in the slowest possible $O(N^2)$ time!
- **Breaking stability in Merge Sort:** Using `<` instead of `<=` when comparing `left[i]` and `right[j]` in the merge step swaps identical elements from the left and right halves, stripping Merge Sort of its stability guarantee.
- **Ignoring $O(N)$ space overhead in Merge Sort:** In memory-constrained embedded environments, Merge Sort's $O(N)$ auxiliary allocation can cause Out-Of-Memory crashes where in-place Quick Sort succeeds.

---

## Key Takeaway

**Merge Sort** and **Quick Sort** are the twin pillars of $O(N \log N)$ divide-and-conquer sorting:
- **Merge Sort** delivers guaranteed **$O(N \log N)$ worst-case time** and **stability** at the expense of **$O(N)$ auxiliary memory**.
- **Quick Sort** sorts **in-place** with **minimal memory overhead ($O(\log N)$ stack)** and blazing real-world cache performance, but requires thoughtful pivot selection to avoid its worst-case **$O(N^2)$** pitfall.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Divide and Conquer** | An algorithmic paradigm that breaks a problem into smaller subproblems, solves them recursively, and combines their solutions. |
| **Merge Sort** | A stable, divide-and-conquer algorithm that splits arrays into halves and merges sorted sub-arrays in $O(N \log N)$ time. |
| **Quick Sort** | An in-place, divide-and-conquer algorithm that partitions an array around a pivot element. |
| **Pivot** | A reference element selected in Quick Sort around which the remaining elements are partitioned. |
| **Partitioning** | Reordering elements such that all values smaller than the pivot precede it and larger values follow it. |
| **External Sorting** | Sorting algorithms designed to handle massive datasets that do not fit entirely in main memory (RAM). |

---

## References

- [GeeksforGeeks: Merge Sort Algorithm](https://www.geeksforgeeks.org/merge-sort/)
- [GeeksforGeeks: Quick Sort Algorithm](https://www.geeksforgeeks.org/quick-sort-algorithm/)
- [MIT OpenCourseWare: Introduction to Algorithms - Lecture 3: Divide & Conquer](https://ocw.mit.edu/)
