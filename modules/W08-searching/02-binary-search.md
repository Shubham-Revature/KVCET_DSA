# Binary Search

In the previous lesson, we saw that Linear Search checks elements one by one in $O(N)$ time. For small collections, this works fine. But imagine searching for a single user among 4 billion accounts—a linear scan could take millions of operations.

If the data is **sorted**, we can do something revolutionary: discard half of the remaining elements with every single comparison. This is **Binary Search**, one of the most elegant, powerful, and fundamental algorithms in computer science.

---

## 1. The Core Intuition: The Number Guessing Game

Imagine someone asks you to guess a secret number between 1 and 100:
- You guess **50**.
- They reply: *"Too low!"*

In a fraction of a second, your brain discards numbers 1 through 50. You don't need to check 1, 2, 3, or 49 individually. Because the numbers are in sorted numerical order, knowing that 50 is too low instantly eliminates 50% of the entire possibilities!

Next, you guess the midpoint of the remaining range (51 to 100), which is **75**:
- They reply: *"Too high!"*

Now you immediately eliminate numbers 75 through 100. In just two questions, you reduced 100 possibilities down to 24 (numbers 51 through 74). 

This is the exact engine behind **Binary Search**.

---

## 2. The Strict Prerequisite: Sorted Order

> [!IMPORTANT]
> **Binary Search ONLY works on sorted collections!**
> If an array is unsorted, there is no relationship between an element's value and its position. Throwing away half the array would mean throwing away data you have never inspected. If data is not sorted and you only need to search once, use Linear Search.

---

## 3. How the Algorithm Works

Binary Search maintains a **search space** defined by two pointers: `left` and `right`. In each iteration, it inspects the element at the midpoint (`mid`).

```text
Target: 70
Sorted Array:  [ 10,  20,  30,  40,  50,  60,  70,  80,  90 ]
                  ▲                       ▲                   ▲
                 left                    mid                right
               (Index 0)               (Index 4)          (Index 8)

Step 1: Compare arr[mid] (50) with target (70).
        Since 50 < 70, target MUST lie to the right of mid!
        Eliminate left half by moving: left = mid + 1 (Index 5).

               [ 10,  20,  30,  40,  50,  60,  70,  80,  90 ]
                                           ▲        ▲        ▲
                                          left     mid     right
                                        (Index 5) (Idx 6) (Idx 8)

Step 2: Calculate new mid = (5 + 8) // 2 = 6.
        Compare arr[mid] (70) with target (70).
        Match found! Return index 6.
```

### The Three Comparison Rules
At each step, comparing `arr[mid]` to `target` yields exactly one of three actions:

1. **`arr[mid] == target`:** Match found! Return `mid` immediately.
2. **`arr[mid] < target`:** Target is larger. It must live in the right half. Discard the left half and midpoint by moving the left boundary:
   $$\text{left} = \text{mid} + 1$$
3. **`arr[mid] > target`:** Target is smaller. It must live in the left half. Discard the right half and midpoint by moving the right boundary:
   $$\text{right} = \text{mid} - 1$$

### Loop Termination (`left <= right`)
The search continues as long as `left <= right`. The moment `left > right`, the search space has shrunk to zero elements without finding the target, meaning the target does not exist in the array.

---

## 4. Calculating the Midpoint & The Overflow Gotcha

In Python, computing the midpoint is typically written as:

```python
mid = (left + right) // 2
```

In lower-level languages like C, C++, and Java, if `left` and `right` are both large 32-bit integers (e.g., $2 \times 10^9$), adding `left + right` causes an **integer overflow**, resulting in a negative number and an instant program crash!

To write defensive, language-agnostic code, engineers use the algebraically equivalent formula:

$$\text{mid} = \text{left} + \frac{\text{right} - \text{left}}{2}$$

While Python automatically handles arbitrarily large integers without overflowing, adopting `mid = left + (right - left) // 2` is a mark of professional algorithmic discipline in technical interviews.

---

## 5. The Staggering Power of $O(\log N)$ Time

Because Binary Search divides the search space in half with every step, the number of steps required to search an array of size $N$ is $\log_2(N)$:

| Array Size ($N$) | Linear Search (Worst Case) | Binary Search (Worst Case) |
|---|---|---|
| **16** | 16 operations | **4** operations |
| **1,024 ($10^3$)** | 1,024 operations | **10** operations |
| **1,048,576 ($10^6$)** | ~1 million operations | **20** operations |
| **1,073,741,824 ($10^9$)** | ~1 billion operations | **30** operations |
| **4,000,000,000** | ~4 billion operations | **32** operations |

Searching through **4 billion items** takes at most **32 comparisons**! 

### Complexity Summary
- **Best Case:** **$\Omega(1)$** (The target happens to be sitting right at the initial midpoint).
- **Worst Case:** **$O(\log N)$** (Target is at the very end of the division chain or absent).
- **Average Case:** **$\Theta(\log N)$**.
- **Space Complexity:** **$O(1)$** Auxiliary Space for the iterative implementation.

---

## 6. Iterative vs. Recursive Implementation

Binary Search can be written either iteratively (with a `while` loop) or recursively:

- **Iterative (Standard):** Uses a simple `while` loop and consumes **$O(1)$ Auxiliary Space**. This is universally preferred in production code.
- **Recursive:** Makes recursive function calls passing updated `left` and `right` indices. It takes **$O(\log N)$ Stack Space** due to recursive call frames.

```python
# Iterative Binary Search (Production Preferred)
def binary_search(arr: list[int], target: int) -> int:
    left = 0
    right = len(arr) - 1

    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

---

## Try it Yourself

**Scenario:** You are building an employee lookup tool for an enterprise security badge database. The badge numbers are guaranteed to be stored in ascending sorted order. Given a sorted list of badge numbers and an employee's badge ID, return the database index, or `-1` if the badge is unassigned.

*Your Spec:*
1. Initialize `left = 0` and `right = len(badges) - 1`.
2. While `left <= right`:
   - Compute `mid = left + (right - left) // 2`.
   - If `badges[mid] == target`, return `mid`.
   - If `badges[mid] < target`, update `left = mid + 1`.
   - Else, update `right = mid - 1`.
3. Return `-1` if loop terminates without finding the badge.

```python
def find_badge_index(badges: list[int], target_badge: int) -> int:
    left = 0
    right = len(badges) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if badges[mid] == target_badge:
            return mid
        elif badges[mid] < target_badge:
            left = mid + 1
        else:
            right = mid - 1

    return -1

# Sorted database records
badge_database = [102, 205, 310, 415, 520, 625, 730, 835, 940]

# 1. Target located in the right half
assert find_badge_index(badge_database, 730) == 6

# 2. Target located at the very start
assert find_badge_index(badge_database, 102) == 0

# 3. Target located at the initial midpoint (Best Case: 1 step)
assert find_badge_index(badge_database, 520) == 4

# 4. Target not present in the database
assert find_badge_index(badge_database, 999) == -1

print("All binary search test cases passed successfully!")
```
```text
All binary search test cases passed successfully!
```

---

## Common Mistakes

- **Running on unsorted data:** Binary Search depends entirely on the monotonicity of sorted data. If the array is unsorted, the algorithm will make incorrect discard decisions and return false negatives.
- **Using `left < right` instead of `left <= right`:** If you write `while left < right:`, the loop will terminate prematurely when `left == right`, completely skipping the final remaining candidate element!
- **Forgetting `+ 1` or `- 1` when updating boundaries:** If you write `left = mid` or `right = mid`, you will trap your algorithm in an **infinite loop** when only two elements remain, because `mid` will repeatedly evaluate to the same index.
- **Using list slicing in recursive implementations:** Writing `binary_search(arr[:mid], target)` in Python creates a **brand-new copy** of the sub-array in $O(N)$ time and space! This completely destroys the $O(\log N)$ efficiency, turning the algorithm into an accidental $O(N)$ disaster. Always pass boundary indices `left` and `right` instead of slicing.

---

## Key Takeaway

**Binary Search** is a divide-and-conquer searching algorithm that operates on **sorted** collections. By comparing the target with the middle element, it eliminates half of the remaining search space in each iteration, achieving an astonishing **$O(\log N)$ Time Complexity** and **$O(1)$ Auxiliary Space**. It reduces searching billions of elements to just a handful of comparisons.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Binary Search** | An efficient search algorithm that repeatedly divides a sorted search interval in half. |
| **Search Space** | The range of candidate indices (from `left` to `right`) where the target could possibly exist. |
| **Midpoint (`mid`)** | The index at the center of the current search interval, calculated as `left + (right - left) // 2`. |
| **Logarithmic Time ($O(\log N)$)** | Running time that grows proportionally to the logarithm of the input size, doubling the input only adds 1 step. |
| **Integer Overflow** | An error in fixed-width numeric systems where a computed value exceeds the maximum representable integer. |

---

## References

- [GeeksforGeeks: Binary Search Algorithm](https://www.geeksforgeeks.org/binary-search/)
- [Python Documentation: bisect module](https://docs.python.org/3/library/bisect.html)
