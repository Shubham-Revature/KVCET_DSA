# Elementary Sorts

Before modern algorithms like Quick Sort and Merge Sort were discovered, computer scientists developed three intuitive, baseline sorting algorithms: **Bubble Sort**, **Selection Sort**, and **Insertion Sort**.

Collectively known as the **Elementary Sorts**, all three run in worst-case **$O(N^2)$ Quadratic Time** and operate **in-place with $O(1)$ extra space**. While $O(N^2)$ is too slow for massive databases, understanding these three algorithms provides vital intuition about data movement, stability, and adaptive performance. In fact, one of them (Insertion Sort) is so fast on small datasets that it powers Python's modern built-in sort today!

---

## 1. Bubble Sort: The Floating Bubble

**Bubble Sort** repeatedly walks through the array, compares adjacent neighbors, and swaps them if they are in the wrong order. With each complete pass, the largest remaining unsorted element "bubbles up" to its correct position at the far right.

```text
Pass 1: Array = [ 5,  1,  4,  2,  8 ]
Compare 5 & 1:   Swap!  ──► [ 1,  5,  4,  2,  8 ]
Compare 5 & 4:   Swap!  ──► [ 1,  4,  5,  2,  8 ]
Compare 5 & 2:   Swap!  ──► [ 1,  4,  2,  5,  8 ]
Compare 5 & 8:   No swap ─► [ 1,  4,  2,  5,  8 ]  (8 is locked in final position!)
```

### The Early-Exit Optimization
If an entire pass completes without a single swap, the array is already sorted! By adding a boolean `swapped` flag, we can terminate immediately, achieving a Best-Case time of **$\Omega(N)$ Linear Time**.

```python
def bubble_sort(arr: list[int]) -> list[int]:
    n = len(arr)
    for i in range(n):
        swapped = False
        # Last i elements are already in place
        for j in range(0, n - 1 - i):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        # If no swaps occurred, array is already sorted!
        if not swapped:
            break
    return arr
```

- **Best Case:** **$\Omega(N)$** (Already sorted array with early exit).
- **Worst / Average Case:** **$O(N^2)$** (Reverse-sorted or random order).
- **Space Complexity:** **$O(1)$** (In-place).
- **Stability:** **Stable** (Does not swap identical adjacent values).

---

## 2. Selection Sort: The Relentless Scout

**Selection Sort** divides the array into a sorted prefix on the left and an unsorted suffix on the right. In each pass, it scouts through the entire unsorted suffix to find the **absolute minimum element**, and swaps it with the first unsorted slot.

```text
Pass 1: [ 5,  1,  4,  2,  8 ]  ──► Minimum is 1 at index 1.
        Swap with index 0:
        [ 1 | 5,  4,  2,  8 ]   (1 is permanently locked)

Pass 2: [ 1 | 5,  4,  2,  8 ]  ──► Minimum in suffix is 2 at index 3.
        Swap with index 1:
        [ 1,  2 | 4,  5,  8 ]   (1, 2 are permanently locked)
```

```python
def selection_sort(arr: list[int]) -> list[int]:
    n = len(arr)
    for i in range(n):
        min_idx = i
        # Find minimum in unsorted suffix
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        # Swap minimum with first unsorted element
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr
```

### The Special Trait: Minimal Memory Writes
Selection Sort always makes exactly **$\le N - 1$ swaps**. If writing to physical hardware memory is extremely expensive (such as writing to worn-out flash memory or EEPROMs), Selection Sort minimizes write cycles better than any other elementary sort.
- **The Catch:** It **always** performs $\frac{N(N-1)}{2}$ comparisons. Even if the array is already 100% sorted, Selection Sort blindly runs in **$\Theta(N^2)$ time**!
- **Stability:** **Unstable** (Long-distance swaps can reorder identical elements).

---

## 3. Insertion Sort: The Playing Card Trick

Imagine holding a hand of playing cards. To sort them, you pick up one card at a time and slide it into its correct position among the cards already sorted in your left hand.

This is **Insertion Sort**. It builds a sorted subarray from left to right. For each new element, it shifts all larger elements one slot to the right and drops the current element into the resulting gap:

```text
Current Hand:    [ 1,  4,  5 | 2,  8 ]    (Next card is 2)
Key = 2:
Compare with 5:  5 > 2 ──► Shift 5 right: [ 1,  4,  _ , 5,  8 ]
Compare with 4:  4 > 2 ──► Shift 4 right: [ 1,  _ , 4,  5,  8 ]
Compare with 1:  1 < 2 ──► Stop! Drop 2 into gap:
Result:          [ 1,  2,  4,  5 | 8 ]
```

```python
def insertion_sort(arr: list[int]) -> list[int]:
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        # Shift elements greater than key one position ahead
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
    return arr
```

### The Superpower: Adaptive Performance
Insertion Sort is **adaptive**. If an array is already sorted, the `while` loop condition `arr[j] > key` fails on the very first comparison for every element!
- **Best Case:** **$\Omega(N)$ Linear Time** with zero shifting.
- **Nearly-Sorted Data:** If every element is at most $k$ positions away from its target slot, Insertion Sort runs in **$O(N \cdot k)$ time**, which is virtually linear!

---

## 4. Side-by-Side Comparison Matrix

| Property | Bubble Sort | Selection Sort | Insertion Sort |
|---|---|---|---|
| **Best-Case Time** | **$\Omega(N)$** (with flag) | $\Omega(N^2)$ | **$\Omega(N)$** |
| **Average Time** | $\Theta(N^2)$ | $\Theta(N^2)$ | $\Theta(N^2)$ |
| **Worst-Case Time** | $O(N^2)$ | $O(N^2)$ | $O(N^2)$ |
| **Auxiliary Space** | **$O(1)$** | **$O(1)$** | **$O(1)$** |
| **Total Swaps/Writes** | $O(N^2)$ swaps | **$O(N)$ swaps** | $O(N^2)$ shifts |
| **Stability** | **Stable** | **Unstable** | **Stable** |
| **Adaptive?** | Yes (early exit) | No | **Yes (extremely fast on nearly sorted)** |

---

## 5. Why Insertion Sort Lives On in Modern Engines

You might assume that $O(N^2)$ algorithms are obsolete in the age of Quick Sort and Merge Sort. 

In reality, **Insertion Sort is used in production systems every single day!**
- Modern hybrid sorting engines like **Timsort** (Python, Java, Android) and **Introsort** (C++ `std::sort`) recursively divide arrays down into small chunks.
- When sub-arrays shrink to size $N \le 32$ or $N \le 64$, the overhead of recursive calls in Merge Sort and Quick Sort outweighs their benefits.
- At tiny sizes, Insertion Sort's cache locality and lack of function-call overhead make it the fastest sorting algorithm on Earth!

---

## Try it Yourself

**Scenario:** Demonstrate the adaptive superpower of Insertion Sort by comparing the number of inner comparisons on a nearly-sorted list versus a reverse-sorted list.

*Your Spec:*
1. Implement `instrumented_insertion_sort(arr)` which counts how many inner while-loop comparisons are executed.
2. Run it on `nearly_sorted = [1, 2, 3, 5, 4]` (only one pair out of place).
3. Run it on `reversed_arr = [5, 4, 3, 2, 1]` (worst-case inverse order).
4. Assert that the sorted results match and observe the comparison difference.

```python
def instrumented_insertion_sort(arr: list[int]) -> tuple[list[int], int]:
    comparisons = 0
    a = list(arr)
    
    for i in range(1, len(a)):
        key = a[i]
        j = i - 1
        while j >= 0:
            comparisons += 1
            if a[j] > key:
                a[j + 1] = a[j]
                j -= 1
            else:
                break
        a[j + 1] = key
        
    return a, comparisons

# 1. Nearly Sorted: [1, 2, 3, 5, 4]
res1, comps1 = instrumented_insertion_sort([1, 2, 3, 5, 4])
assert res1 == [1, 2, 3, 4, 5]

# 2. Reverse Sorted: [5, 4, 3, 2, 1]
res2, comps2 = instrumented_insertion_sort([5, 4, 3, 2, 1])
assert res2 == [1, 2, 3, 4, 5]

print(f"Nearly sorted comparisons: {comps1}")
print(f"Reverse sorted comparisons: {comps2}")
```
```text
Nearly sorted comparisons: 5
Reverse sorted comparisons: 10
```

---

## Common Mistakes

- **Swapping instead of shifting in Insertion Sort:** Beginners often write `arr[j], arr[j+1] = arr[j+1], arr[j]` inside the insertion loop. Repeated tuple swaps require 3 writes per shift. Shifting `arr[j+1] = arr[j]` and dropping `key` at the end requires only 1 write per shift!
- **Forgetting the early-exit check in Bubble Sort:** Without the `swapped` boolean check, Bubble Sort runs in $O(N^2)$ even on an already-sorted array, forfeiting its best-case $\Omega(N)$ benefit.
- **Assuming Selection Sort is adaptive:** Many assume Selection Sort speeds up if the list is already sorted. It does not! Selection Sort always scans the entire unsorted subarray to find the minimum, guaranteeing $\Theta(N^2)$ comparisons every time.
- **Using elementary sorts on large datasets:** Never use Bubble, Selection, or Insertion sort on arrays with thousands of elements in production. They scale quadratically and will freeze your application.

---

## Key Takeaway

The three **Elementary Sorts** (**Bubble**, **Selection**, and **Insertion**) operate in **$O(N^2)$ worst-case time** with **$O(1)$ auxiliary space**. Bubble Sort pushes large items to the right via adjacent swaps; Selection Sort minimizes total swaps ($O(N)$) by scouting for the absolute minimum; and Insertion Sort dynamically slides items into a growing sorted hand. Because of its exceptional cache locality and adaptive $\Omega(N)$ speed on nearly-sorted data, **Insertion Sort** remains an essential component of modern production hybrid algorithms like Timsort.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Elementary Sorts** | The fundamental $O(N^2)$ comparison sorting algorithms: Bubble, Selection, and Insertion Sort. |
| **Bubble Sort** | An algorithm that repeatedly swaps adjacent elements that are out of order until the list is sorted. |
| **Selection Sort** | An algorithm that repeatedly finds the minimum element from the unsorted portion and places it at the beginning. |
| **Insertion Sort** | An algorithm that builds the final sorted array one item at a time by inserting each element into its proper position. |
| **Adaptive Algorithm** | An algorithm whose running time improves when the input data is already partially or completely sorted. |
| **Stability** | A property of a sorting algorithm where elements with equal values retain their relative original order. |

---

## References

- [GeeksforGeeks: Bubble Sort Algorithm](https://www.geeksforgeeks.org/bubble-sort-algorithm/)
- [GeeksforGeeks: Selection Sort Algorithm](https://www.geeksforgeeks.org/selection-sort-algorithm-2/)
- [GeeksforGeeks: Insertion Sort Algorithm](https://www.geeksforgeeks.org/insertion-sort-algorithm/)
