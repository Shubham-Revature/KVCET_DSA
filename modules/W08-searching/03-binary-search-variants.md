# Binary Search Variants

Standard Binary Search is designed to answer a simple question: *"Does this element exist, and if so, at what index?"* It returns the very first match it happens to stumble upon at `mid`.

However, real-world datasets are rarely composed of unique values. They contain repeated scores, duplicate timestamps, and identical prices. When duplicates exist, standard Binary Search behaves unpredictably—it might return index 3, index 7, or index 5 of a cluster of identical elements.

To solve real-world problems, we need **Binary Search Variants**: algorithms that find the **first occurrence**, the **last occurrence**, the **insertion point**, or the **floor and ceiling** of a target.

---

## 1. The Duplicate Dilemma

Consider this sorted array containing duplicate 20s:

```text
Indices:  [  0,   1,   2,   3,   4,   5,   6  ]
Values:   [ 10,  20,  20,  20,  20,  30,  40  ]
                           ▲
                      Initial mid (Index 3)
```

If we search for `20`:
- Midpoint is index `3` (`arr[3] == 20`).
- Standard binary search returns index `3` immediately.

Notice that index `3` is neither the *first* occurrence (which is index `1`) nor the *last* occurrence (which is index `4`). 

> [!WARNING]
> **The Naive Linear Scan Trap:**
> Beginners often find one match using binary search, and then linearly scan left and right with a `while` loop to find the boundaries. If an array of 1,000,000 elements contains 900,000 copies of `20`, that linear scan takes $O(N)$ time, completely destroying the $O(\log N)$ speedup! We must find boundaries using **pure binary search**.

---

## 2. Finding the First Occurrence (Lower Bound)

To find the **first** (leftmost) occurrence, we modify the match condition:
- When `arr[mid] == target`, **do not return immediately!**
- Save `mid` as our current best answer: `result = mid`.
- There might be an *even earlier* match further to the left. Force the search to continue looking left by setting:
  $$\text{right} = \text{mid} - 1$$

```text
Target: 20
Array: [ 10,  20,  20,  20,  20,  30,  40 ]
                ▲        ▲
              First     mid (Match! Save result = 3, move right = mid - 1)
```

```python
def find_first_occurrence(arr: list[int], target: int) -> int:
    left, right = 0, len(arr) - 1
    result = -1

    while left <= right:
        mid = left + (right - left) // 2

        if arr[mid] == target:
            result = mid       # Record candidate index
            right = mid - 1    # Keep searching to the LEFT
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return result
```

---

## 3. Finding the Last Occurrence (Upper Bound)

To find the **last** (rightmost) occurrence, we mirror the logic:
- When `arr[mid] == target`, record `result = mid`.
- Force the search to continue looking right for later occurrences by setting:
  $$\text{left} = \text{mid} + 1$$

```python
def find_last_occurrence(arr: list[int], target: int) -> int:
    left, right = 0, len(arr) - 1
    result = -1

    while left <= right:
        mid = left + (right - left) // 2

        if arr[mid] == target:
            result = mid       # Record candidate index
            left = mid + 1     # Keep searching to the RIGHT
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return result
```

### The Superpower: Counting Duplicates in $O(\log N)$ Time
Once you can find the first and last occurrences in $O(\log N)$ time, you can count how many times an element appears in a massive sorted array without scanning a single duplicate:

$$\text{Total Occurrences} = \text{last\_index} - \text{first\_index} + 1$$

---

## 4. Search Insert Position (Floor & Ceiling)

What if the target is **not** present in the array? Where would it go if we wanted to insert it while preserving sorted order?

```text
Array:   [ 10,  20,  30,  40,  50 ]
Target:  25
Missing! Should be inserted at Index 2 (between 20 and 30).
```

Notice a fascinating mathematical invariant of Binary Search:
When the loop `while left <= right:` finishes without finding the target, the pointers cross (`left > right`). At the moment of termination:
- **`left`** points to the exact **Insertion Index**!
- **`arr[right]`** is the **Floor** (largest element smaller than target).
- **`arr[left]`** is the **Ceiling** (smallest element greater than target).

---

## 5. Python's Standard Library: The `bisect` Module

Python provides a built-in, highly optimized C implementation of binary search variants in the `bisect` module:

```python
import bisect

scores = [10, 20, 20, 20, 30, 40]

# bisect_left: First index where target could be inserted (before duplicates)
idx_left = bisect.bisect_left(scores, 20)    # Returns 1 (First occurrence!)

# bisect_right: Last index where target could be inserted (after duplicates)
idx_right = bisect.bisect_right(scores, 20)  # Returns 4 (Index AFTER last 20)

# Number of occurrences of 20
count = idx_right - idx_left                 # 4 - 1 = 3
```

- **`bisect_left(arr, x)`:** Returns the leftmost insertion index. If `x` exists, it points directly to the **first occurrence**.
- **`bisect_right(arr, x)`:** Returns the rightmost insertion index. If `x` exists, it points to the index **immediately after the last occurrence**.

---

## Try it Yourself

**Scenario:** You are analyzing a server outage. You have a sorted list of integer timestamps representing error spikes. Given a target timestamp `404`, determine:
1. The first index where error `404` was logged.
2. The last index where error `404` was logged.
3. The total number of error `404` events recorded.

*Your Spec:*
- Implement both `find_first` and `find_last` functions.
- If the target is not present, both return `-1` and the count is `0`.
- Verify using assertions.

```python
def find_first(arr: list[int], target: int) -> int:
    left, right = 0, len(arr) - 1
    result = -1
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            result = mid
            right = mid - 1  # Squeeze search range left
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return result

def find_last(arr: list[int], target: int) -> int:
    left, right = 0, len(arr) - 1
    result = -1
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            result = mid
            left = mid + 1   # Squeeze search range right
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return result

# Sorted error log timestamps
error_log = [100, 200, 301, 404, 404, 404, 404, 404, 500, 503]

first_idx = find_first(error_log, 404)
last_idx = find_last(error_log, 404)
total_count = (last_idx - first_idx + 1) if first_idx != -1 else 0

assert first_idx == 3
assert last_idx == 7
assert total_count == 5

print(f"Error 404 first seen at index {first_idx}, last seen at index {last_idx}.")
print(f"Total error events: {total_count}")
```
```text
Error 404 first seen at index 3, last seen at index 7.
Total error events: 5
```

---

## Common Mistakes

- **Linear scanning after a match:** Calling standard binary search and using a `while` loop to scan left and right for duplicates causes worst-case $O(N)$ runtime. Always run two independent $O(\log N)$ binary searches to find both boundaries.
- **Off-by-one in frequency count:** Forgetting the `+ 1` in `(last - first + 1)`. If an element appears from index 3 to 3, `3 - 3 = 0`, but the element appears once! The correct formula is `3 - 3 + 1 = 1`.
- **Confusing `bisect_left` and `bisect_right`:** Remember that `bisect_right` returns the index *after* the existing elements. The last occurrence index is `bisect_right(arr, x) - 1`, not `bisect_right(arr, x)`.
- **Returning prematurely:** When looking for boundary positions, never write `return mid` inside the `arr[mid] == target` block. You must store `mid` in a tracker variable and continue moving your boundary pointers until the loop finishes.

---

## Key Takeaway

When sorted arrays contain duplicate values, standard Binary Search only finds an arbitrary match. By caching candidate indices and continuing to squeeze the search boundary (`right = mid - 1` for first occurrence, `left = mid + 1` for last occurrence), we can find boundary positions and frequency counts in pure **$O(\log N)$ time**. In Python, the built-in `bisect` module implements these lower- and upper-bound routines out of the box.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **First Occurrence (Lower Bound)** | The smallest index at which a target value appears in a sorted collection. |
| **Last Occurrence (Upper Bound)** | The largest index at which a target value appears in a sorted collection. |
| **Search Insert Position** | The index where a non-existent element should be placed to preserve sorted order. |
| **Floor** | The largest element in the array that is less than or equal to the target. |
| **Ceiling** | The smallest element in the array that is greater than or equal to the target. |
| **`bisect` Module** | Python standard library module implementing bisection algorithms. |

---

## References

- [GeeksforGeeks: Find First and Last Position of an Element in Sorted Array](https://www.geeksforgeeks.org/find-first-and-last-positions-of-an-element-in-a-sorted-array/)
- [Python Documentation: bisect — Array bisection algorithm](https://docs.python.org/3/library/bisect.html)
