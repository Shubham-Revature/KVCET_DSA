# Sort-Based Patterns

In technical interviews and real-world system architecture, you will rarely be asked to write Quick Sort or Merge Sort from scratch. Interviewers know that production languages already provide optimized $O(N \log N)$ sorting engines.

Instead, the true test of algorithmic skill is **pattern recognition**: knowing when an intractable, complex problem can be made instantly solvable by sorting the input first.

In this concluding lesson of Week 9, we examine the four premier **Sort-Based Patterns** that recur across FAANG interviews and high-performance engineering.

---

## Pattern 1: Interval Merging & Scheduling

Consider the challenge of overlapping time intervals (such as calendar meetings or CPU reservation blocks):
$$\text{Intervals} = [[8, 10], [2, 6], [1, 3], [15, 18]]$$

If the intervals are unsorted, checking whether interval $A$ overlaps with interval $B$ requires comparing every interval against every other interval—a quadratic **$O(N^2)$ nightmare**.

### The Breakthrough: Sort by Start Time
The moment you sort intervals by their start time (`intervals.sort(key=lambda x: x[0])`), a magical invariant emerges:
**Any intervals that overlap are guaranteed to sit directly adjacent to one another!**

```text
Unsorted:  [ [8, 10],  [2, 6],  [1, 3],  [15, 18] ]
Sorted:    [ [1, 3],   [2, 6],  [8, 10], [15, 18] ]
              └───┬───┘
               Overlaps! Merge into [1, 6]
Merged:    [ [1, 6],            [8, 10], [15, 18] ]
```

### The Merge Logic:
1. Initialize `merged = [intervals[0]]`.
2. For each subsequent `current` interval:
   - If `current.start <= merged[-1].end`: They overlap! Extend the merged interval:
     $$\text{merged}[-1].\text{end} = \max(\text{merged}[-1].\text{end}, \text{current}.\text{end})$$
   - Otherwise, they do not overlap. Append `current` to `merged`.
3. Total Time: **$O(N \log N)$** for the sort + **$O(N)$** for the single scan = **$O(N \log N)$** overall!

---

## Pattern 2: Two-Pointer Squeezing (The 3-Sum Pattern)

The famous **3-Sum Problem** asks: *Given an integer array, find all unique triplets $[a, b, c]$ such that $a + b + c = 0$.*

- **Brute Force:** Testing all combinations using three nested loops takes **$O(N^3)$ Cubic Time**.
- **The Sort-Based Fix:**
  1. Sort the array first in **$O(N \log N)$**.
  2. Iterate through the array with pointer `i` from $0$ to $N - 1$.
  3. For each fixed `nums[i]`, find two numbers in the remaining subarray that sum to `-nums[i]` using **Two Pointers (`left` and `right`)** squeezing inward in **$O(N)$ time**!

```text
Fixed nums[i] = -4
Remaining:  [ -1,  -1,   0,   1,   2 ]
               ▲                    ▲
             left                 right
             sum = -1 + 2 = 1 (Too small, move left rightward!)
```

### The Added Benefit: Trivial Duplicate Elimination
Sorting clusters duplicate values together. When avoiding duplicate triplets in 3-Sum, you simply skip repeated elements with `if i > 0 and nums[i] == nums[i-1]: continue`. Doing this on unsorted data requires expensive hash set allocations.
- **Total Time:** **$O(N^2)$ Quadratic Time** (vastly superior to $O(N^3)$).

---

## Pattern 3: Minimizing Differences & Greedy Pairing

**Problem:** Given an unsorted array of numbers, find the pair of elements with the **minimum absolute difference** $|a - b|$.

- **Without Sorting:** Compare all pairs: $\frac{N(N-1)}{2}$ comparisons $\implies O(N^2)$.
- **With Sorting:** In any sorted array, the closest neighbor to any number $X$ is guaranteed to be either its immediate left neighbor or its immediate right neighbor!
  1. Sort the array in **$O(N \log N)$**.
  2. Perform a single linear pass comparing adjacent neighbors `arr[i+1] - arr[i]` in **$O(N)$**.
  3. Total Time: **$O(N \log N)$**.

---

## Pattern 4: Anagram Grouping via Canonical Keys

An **anagram** is a word formed by rearranging the letters of another word (e.g., `"listen"` and `"silent"`).

How do you group 100,000 words into lists of anagrams?
- If you sort the characters of any anagram alphabetically, they produce the exact same **canonical string**:
  - `sorted("listen")` $\implies$ `"eilnst"`
  - `sorted("silent")` $\implies$ `"eilnst"`
- By using the sorted string as a dictionary key, you can categorize words into buckets in $O(N \cdot K \log K)$ time (where $K$ is the length of each word):

```python
from collections import defaultdict

words = ["eat", "tea", "tan", "ate", "nat", "bat"]
anagram_map = defaultdict(list)

for w in words:
    canonical = "".join(sorted(w))
    anagram_map[canonical].append(w)

# Groups: [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

---

## Try it Yourself

**Scenario:** Implement the classic **Merge Intervals** algorithm (LeetCode 56) to consolidate overlapping calendar booking slots.

*Your Spec:*
1. Sort intervals by start time.
2. Initialize `merged` with the first interval.
3. Iterate through intervals, checking if the current interval starts before or at the end of the last merged interval.
4. Merge or append accordingly.
5. Validate using assertions.

```python
def merge_intervals(intervals: list[list[int]]) -> list[list[int]]:
    if not intervals:
        return []

    # 1. Sort by start time (O(N log N))
    intervals.sort(key=lambda x: x[0])

    merged = [intervals[0]]

    # 2. Linear sweep (O(N))
    for current in intervals[1:]:
        prev = merged[-1]

        # Overlapping condition: current start <= prev end
        if current[0] <= prev[1]:
            # Extend end boundary to the maximum of both
            prev[1] = max(prev[1], current[1])
        else:
            # Disjoint interval; start a new slot
            merged.append(current)

    return merged

# Test Cases
test_1 = [[1, 3], [2, 6], [8, 10], [15, 18]]
assert merge_intervals(test_1) == [[1, 6], [8, 10], [15, 18]]

test_2 = [[1, 4], [4, 5]]  # Touching boundaries
assert merge_intervals(test_2) == [[1, 5]]

test_3 = [[1, 10], [2, 3], [4, 8]]  # Completely engulfed intervals
assert merge_intervals(test_3) == [[1, 10]]

print("All interval merging test cases passed successfully!")
```
```text
All interval merging test cases passed successfully!
```

---

## Common Mistakes

- **Forgetting `max()` when updating interval ends:** Writing `prev[1] = current[1]` instead of `prev[1] = max(prev[1], current[1])`. If the previous interval completely engulfs the current interval (`[1, 10]` and `[2, 5]`), blindly setting the end to `5` will shrink your interval!
- **Sorting by end time instead of start time:** In interval merging, sorting by end time does not guarantee consecutive overlap. You must sort by **start time**. (Note: In contrast, Greedy Activity Selection sorts by *end time*).
- **Failing to skip duplicates in 3-Sum:** When implementing 3-Sum, forgetting to advance pointers past duplicate values produces redundant triplets, failing edge test cases.
- **Sorting when hashing is faster:** If you only need to find whether two numbers sum to target (Two-Sum), a Hash Set is **$O(N)$**, whereas sorting is **$O(N \log N)$**. Only sort if you must return unique sorted triplets or maintain $O(1)$ space.

---

## Key Takeaway

Sorting is rarely the final goal of an algorithm; it is the **enabling foundation** that unlocks downstream simplicity. By paying an upfront cost of **$O(N \log N)$**, sorting transforms chaotic data into an ordered structure where:
1. Overlapping intervals become consecutive.
2. Two-pointer squeezing eliminates brute-force loops (e.g., $O(N^3) \to O(N^2)$).
3. Closest numerical pairs become immediate neighbors.
4. Anagrams resolve to identical canonical keys.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Sort-Based Pattern** | An algorithmic paradigm where sorting the input enables linear or two-pointer downstream solutions. |
| **Interval Merging** | Combining overlapping or contiguous ranges into a minimal set of non-overlapping intervals. |
| **Two-Pointer Squeezing** | Moving pointers inward from opposite ends of a sorted array to evaluate target conditions in $O(N)$ time. |
| **Canonical Form** | A standardized, unique representation of an object (e.g., `"eilnst"` for `"listen"`). |
| **Disjoint Intervals** | Intervals that do not overlap and share no common points. |

---

## References

- [LeetCode 56: Merge Intervals](https://leetcode.com/problems/merge-intervals/)
- [LeetCode 15: 3Sum](https://leetcode.com/problems/3sum/)
- [GeeksforGeeks: Interval Scheduling and Merging](https://www.geeksforgeeks.org/merging-intervals/)
