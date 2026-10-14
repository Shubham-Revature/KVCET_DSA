# Heap Sort

In the previous lesson, we witnessed the great sorting dilemma:
- **Merge Sort** guarantees **$O(N \log N)$ worst-case time**, but demands **$O(N)$ extra memory**.
- **Quick Sort** operates **in-place with minimal memory**, but carries a dangerous **$O(N^2)$ worst-case trap**.

This brings us to a historic question: *Can we create an algorithm that has the guaranteed $O(N \log N)$ safety of Merge Sort AND the in-place $O(1)$ memory efficiency of Quick Sort?*

The answer is **Heap Sort**. By viewing a flat array as a complete binary tree, Heap Sort guarantees $O(N \log N)$ performance in all scenarios while using strictly **$O(1)$ auxiliary space**.

---

## 1. The Core Idea: An Array Disguised as a Tree

A **Binary Heap** is a complete binary tree stored inside a standard array without any pointer nodes. 

Because the tree is completely filled at all levels from left to right, we can navigate parent and child relationships using simple index arithmetic:

```text
Tree View:                      Array Representation:
        [0] 90                  Indices: [ 0,  1,  2,  3,  4,  5 ]
       ╱      ╲                 Values:  [90, 75, 80, 25, 40, 60 ]
     [1] 75    [2] 80
     ╱    ╲    ╱
   [3]25 [4]40 [5]60
```

For any element at index `i`:
- **Left Child:** `2 * i + 1`
- **Right Child:** `2 * i + 2`
- **Parent:** `(i - 1) // 2`

### The Max-Heap Property
In a **Max-Heap**, every parent node is greater than or equal to both of its children:
$$\text{arr}[\text{parent}] \ge \text{arr}[\text{child}]$$
The single most important consequence of this rule is: **The absolute largest element in the entire collection is always guaranteed to sit at index `0` (the root).**

---

## 2. The Two Phases of Heap Sort

Heap Sort sorts an array in two distinct phases:

### Phase 1: Build the Max-Heap (`O(N)` Time)
Before we can sort, we must rearrange the unsorted array so that it satisfies the Max-Heap property. 

We do this from the bottom up using a process called **sifting down** (`heapify`). We start at the last non-leaf node (index `n // 2 - 1`) and work backwards to index `0`, sifting each element down to its valid position.

> [!NOTE]
> **A Surprising Mathematical Marvel:**
> Even though individual sift-downs can take $O(\log N)$ time, the vast majority of nodes sit near the bottom of the tree where they only sift down 0 or 1 levels. Mathematically, building the entire heap takes **$O(N)$ Linear Time**, not $O(N \log N)$!

### Phase 2: Extract-and-Sort (`O(N \log N)` Time)
Once the Max-Heap is built, sorting in ascending order is surprisingly straightforward:
1. The largest element sits at `arr[0]`.
2. **Swap** `arr[0]` with the last element of the unsorted heap (`arr[i]`). Now the largest element is locked at its final sorted position at the end of the array!
3. **Shrink** the active heap size by 1.
4. **Sift down** the new root (`arr[0]`) to restore the Max-Heap property. This takes $O(\log N)$ time.
5. Repeat for all elements from right to left.

```text
Step 1: Swap root (90) with last element (60), shrink heap range:
        [ 60,  75,  80,  25,  40 | 90 ]   <-- 90 is locked!
Step 2: Sift down 60 from index 0:
        [ 80,  75,  60,  25,  40 | 90 ]   <-- Max-Heap restored!
Step 3: Swap root (80) with last unsorted element (40):
        [ 40,  75,  60,  25 | 80,  90 ]   <-- 80 is locked!
```

---

## 3. Python Implementation

Here is the complete, self-contained in-place implementation of Heap Sort:

```python
def heap_sort(arr: list[int]) -> list[int]:
    n = len(arr)

    # Phase 1: Build Max-Heap (bottom-up from last non-leaf node)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)

    # Phase 2: Extract elements one by one from the heap
    for i in range(n - 1, 0, -1):
        # Swap current root (maximum) to the end of the array
        arr[0], arr[i] = arr[i], arr[0]

        # Sift down root within the reduced heap of size i
        heapify(arr, i, 0)

    return arr


def heapify(arr: list[int], n: int, i: int) -> None:
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    # Check if left child exists and is greater than current largest
    if left < n and arr[left] > arr[largest]:
        largest = left

    # Check if right child exists and is greater than current largest
    if right < n and arr[right] > arr[largest]:
        largest = right

    # If largest is not root, swap and continue sifting down recursively
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

---

## 4. Complexity & Trade-Offs

| Metric | Heap Sort | Merge Sort | Quick Sort |
|---|---|---|---|
| **Best-Case Time** | **$\Omega(N \log N)$** | **$\Omega(N \log N)$** | **$\Omega(N \log N)$** |
| **Average-Case Time** | **$\Theta(N \log N)$** | **$\Theta(N \log N)$** | **$\Theta(N \log N)$** |
| **Worst-Case Time** | **$O(N \log N)$** (Guaranteed) | **$O(N \log N)$** (Guaranteed) | $O(N^2)$ (Unbalanced) |
| **Auxiliary Space** | **$O(1)$** (Pure In-Place) | $O(N)$ (Allocates buffers) | $O(\log N)$ (Call stack) |
| **Stability** | **Unstable** | **Stable** | **Unstable** |
| **Cache Performance** | **Poor (jumps memory)** | Moderate | **Superior** |

### The Real-World Paradox: Why Isn't Heap Sort the Default?
If Heap Sort guarantees $O(N \log N)$ and uses $O(1)$ space, why doesn't Python or C++ use it as their primary default sorting algorithm?

The answer comes down to **CPU Cache Locality**:
- In Heap Sort, traversing from parent `i` to children `2i + 1` and `2i + 2` causes large, non-sequential jumps across memory addresses. This causes frequent **CPU cache misses**.
- Quick Sort and Timsort read memory sequentially and maintain strong locality, making them 2x to 3x faster in real-world clock time on modern hardware.

### Introsort: The Best of Both Worlds
To protect against Quick Sort's worst case while enjoying its real-world speed, modern standard libraries (like C++ `std::sort`) use **Introsort**:
1. It begins with **Quick Sort**.
2. If Quick Sort's recursion depth exceeds $2 \log_2 N$ (detecting an adversarial input), it automatically switches to **Heap Sort** to guarantee $O(N \log N)$ termination!

---

## Try it Yourself

**Scenario:** Sort an unsorted array of telemetry readings in-place using Heap Sort and verify the outcome against Python's sorted reference.

*Your Spec:*
1. Implement the `heap_sort` and `heapify` functions.
2. Sort an arbitrary list of positive and negative numbers in-place.
3. Validate sorting correctness using assertions.

```python
def heapify(arr: list[int], n: int, i: int) -> None:
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    if left < n and arr[left] > arr[largest]:
        largest = left
    if right < n and arr[right] > arr[largest]:
        largest = right

    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

def heap_sort(arr: list[int]) -> list[int]:
    n = len(arr)
    # 1. Build Max-Heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    # 2. Extract elements from heap
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    return arr

# Test Data
data = [12, 11, 13, -5, 6, 7, 0, 19, -2]
expected = sorted(data)

# Sort in-place
heap_sort(data)

assert data == expected
print("Heap sort executed and verified successfully!")
print(f"Sorted output: {data}")
```
```text
Heap sort executed and verified successfully!
Sorted output: [-5, -2, 0, 6, 7, 11, 12, 13, 19]
```

---

## Common Mistakes

- **Building a Min-Heap for ascending sort:** Beginners often assume an ascending sort requires a Min-Heap. If you place the root of a Min-Heap at the end, you get descending order! To sort in ascending order in-place, you **must use a Max-Heap** so the largest elements are extracted and placed at the end.
- **Thinking building the heap takes $O(N \log N)$:** A common interview question asks for the time complexity of `build_heap`. It is **$O(N)$**, not $O(N \log N)$ (due to converging geometric series of node heights).
- **Using `heapq` and calling it in-place Heap Sort:** Pushing elements into a list with `heapq.heappush` and popping them out into a new list requires $O(N)$ extra memory. True Heap Sort operates directly inside the original array with $O(1)$ extra space.
- **Off-by-one errors in `heapify` boundaries:** Forgetting that child checks must strictly be `< n` (i.e. `left < n` and `right < n`), leading to `IndexError`.

---

## Key Takeaway

**Heap Sort** achieves the elusive combination of **guaranteed $O(N \log N)$ time** in all cases and **$O(1)$ auxiliary memory** by modeling a flat array as an implicit complete binary tree. By building a Max-Heap in $O(N)$ time and systematically swapping root elements to the end in $O(N \log N)$ time, it eliminates Quick Sort's $O(N^2)$ worst-case danger without Merge Sort's memory footprint.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Heap Sort** | A comparison-based sorting algorithm that uses a binary heap data structure to sort in-place in $O(N \log N)$ time. |
| **Binary Heap** | A complete binary tree stored inside a linear array without pointers. |
| **Max-Heap** | A heap where every parent node is greater than or equal to its children, placing the maximum value at the root. |
| **Heapify (Sift Down)** | The process of restoring the heap property by moving an out-of-place root downward through its children. |
| **Introsort** | A hybrid sorting algorithm that combines Quick Sort, Heap Sort, and Insertion Sort to achieve optimal speed and safety. |

---

## References

- [GeeksforGeeks: Heap Sort Algorithm](https://www.geeksforgeeks.org/heap-sort/)
- [Introduction to Algorithms (CLRS): Chapter 6 - Heapsort](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
