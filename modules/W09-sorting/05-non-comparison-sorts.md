# Non-Comparison Sorts

In Lesson 01, we established a mathematical law: **No sorting algorithm can beat $\Omega(N \log N)$ in the worst case.** 

So how is it possible that computer science textbooks claim algorithms like **Counting Sort**, **Radix Sort**, and **Bucket Sort** can sort in **$O(N)$ Linear Time**? Did mathematicians make a mistake?

Not at all. The $\Omega(N \log N)$ lower bound strictly applies to **comparison-based** sorting—algorithms that determine order by asking: *"Is $A < B$?"* 

If an algorithm **never compares two elements against each other**, the decision-tree proof no longer applies! By exploiting the mathematical properties of numbers, **Non-Comparison Sorts** achieve linear time by treating values as array indices or sequences of digits.

---

## 1. Counting Sort: Bypassing Comparisons Entirely

Imagine you need to sort the test scores of 100,000 students. The scores are integers ranging strictly from `0` to `100`.

Instead of comparing Student A's score with Student B's score ($O(N \log N)$ comparisons), you can use an array of 101 counters (one for each possible score):
1. **Count:** Walk through the scores and tally how many students got 0, how many got 1, etc.
2. **Reconstruct:** Write out the scores in order based on their counts!

```text
Input:  [ 4,  2,  2,  8,  3,  3,  1 ]      (Range: 1 to 8)
Counts: Value: [ 1,  2,  3,  4,  5,  6,  7,  8 ]
        Count: [ 1,  2,  2,  1,  0,  0,  0,  1 ]

Output: [ 1,  2,  2,  3,  3,  4,  8 ]     (Reconstructed in O(N) time!)
```

### The Stable Counting Sort Algorithm
To make Counting Sort **stable** (preserving original relative order for objects), we compute cumulative prefix sums of the counts and iterate through the input **backwards**:

```python
def counting_sort(arr: list[int]) -> list[int]:
    if not arr:
        return []

    min_val = min(arr)
    max_val = max(arr)
    k = max_val - min_val + 1  # Range of values

    count = [0] * k
    output = [0] * len(arr)

    # 1. Count occurrences
    for num in arr:
        count[num - min_val] += 1

    # 2. Transform into prefix sums (positions)
    for i in range(1, k):
        count[i] += count[i - 1]

    # 3. Build output array backwards to ensure stability
    for num in reversed(arr):
        idx = count[num - min_val] - 1
        output[idx] = num
        count[num - min_val] -= 1

    return output
```

### The Fatal Flaw of Counting Sort: The Range Trap
Counting Sort takes **$O(N + K)$ Time** and **$O(N + K)$ Space**, where $K$ is the range $(\max - \min + 1)$:
- **Ideal Case:** Sorting 10 million customer ages ($N = 10^7, K = 120$). $K$ is negligible $\implies$ **Blazing $O(N)$ speed!**
- **Disaster Case:** Sorting 5 numbers: `[1, 5, 2, 8, 1_000_000_000]`. Here, $N = 5$, but $K = 10^9$. Counting Sort allocates a list of 1 billion integers (4 GB of RAM) just to sort 5 numbers!

---

## 2. Radix Sort: Overcoming the Range Limit

How do we sort large numbers without falling into Counting Sort's range trap? We use **Radix Sort**.

Instead of sorting the entire number at once, Radix Sort sorts the numbers **digit by digit**, from the **Least Significant Digit (LSD)** to the **Most Significant Digit (MSD)**:

```text
Input:       [ 170,  045,  075,  090,  802,  024,  002,  066 ]

Pass 1 (1s): [ 170,  090,  802,  002,  024,  045,  075,  066 ]  (Sorted by 1s)
Pass 2 (10s):[ 802,  002,  024,  045,  066,  170,  075,  090 ]  (Sorted by 10s)
Pass 3 (100s):[002,  024,  045,  066,  075,  090,  170,  802 ]  (Fully sorted!)
```

### Why Stability is Mandatory
Notice why earlier passes do not get ruined by later passes: because the subroutine used on each digit is **stable**, numbers with identical higher digits automatically retain the order established by their lower digits!

### Complexity of Radix Sort
Let $D$ be the maximum number of digits, and $B$ be the base (e.g., $B = 10$ for decimal numbers):
- **Time Complexity:** **$O(D \cdot (N + B))$**.
- For 32-bit integers, $D \le 10$ and $B = 10$, both are fixed constants $\implies$ **$O(N)$ Pure Linear Time!**
- **Space Complexity:** **$O(N + B)$**.

---

## 3. Bucket Sort: The Uniform Distribution Specialist

**Bucket Sort** is used when input elements are uniformly distributed across a continuous range (typically floating-point numbers in $[0.0, 1.0)$):

1. **Scatter:** Divide the range into $N$ equal-sized buckets. Distribute each element into its corresponding bucket: `bucket_idx = int(num * N)`.
2. **Sort:** Sort each individual bucket (typically using Insertion Sort).
3. **Gather:** Concatenate all buckets in sequence.

```text
Input:     [ 0.78,  0.17,  0.39,  0.26,  0.72,  0.94,  0.21,  0.12 ]
Buckets:   [0.1] ──► [0.12, 0.17]
           [0.2] ──► [0.21, 0.26]
           [0.3] ──► [0.39]
           [0.7] ──► [0.72, 0.78]
           [0.9] ──► [0.94]
Gather:    [ 0.12, 0.17, 0.21, 0.26, 0.39, 0.72, 0.78, 0.94 ]
```

When data is uniformly distributed, each bucket contains an average of $O(1)$ elements. Sorting each bucket takes $O(1)$ time, yielding **$O(N)$ Average Time**.

---

## 4. Comparison of Non-Comparison Sorts

| Algorithm | Average Time | Worst-Case Time | Auxiliary Space | Data Requirements |
|---|---|---|---|---|
| **Counting Sort** | **$O(N + K)$** | $O(N + K)$ | $O(N + K)$ | Discrete integers within a small, bounded range $K$. |
| **Radix Sort** | **$O(D \cdot (N + B))$** | $O(D \cdot (N + B))$ | $O(N + B)$ | Integers or fixed-length strings with a fixed number of digits $D$. |
| **Bucket Sort** | **$O(N + K)$** | $O(N^2)$ (Clustering) | $O(N + K)$ | Continuous values uniformly distributed over a known interval. |

---

## Try it Yourself

**Scenario:** Implement Radix Sort to sort a list of transaction IDs in $O(N)$ time using digit-by-digit counting sort.

*Your Spec:*
1. Implement a helper `counting_sort_by_digit(arr, exp)` that sorts based on the digit at place value `exp` ($1, 10, 100, \dots$).
2. Implement `radix_sort(arr)` that finds the maximum value and iteratively calls the helper for each place value.
3. Validate sorting correctness using assertions.

```python
def counting_sort_by_digit(arr: list[int], exp: int) -> list[int]:
    n = len(arr)
    output = [0] * n
    count = [0] * 10  # Base 10 digits (0 to 9)

    # 1. Count occurrences of current digit: (num // exp) % 10
    for num in arr:
        digit = (num // exp) % 10
        count[digit] += 1

    # 2. Cumulative count for stable positioning
    for i in range(1, 10):
        count[i] += count[i - 1]

    # 3. Build output array in reverse to maintain stability
    for num in reversed(arr):
        digit = (num // exp) % 10
        idx = count[digit] - 1
        output[idx] = num
        count[digit] -= 1

    return output


def radix_sort(arr: list[int]) -> list[int]:
    if not arr:
        return []

    max_val = max(arr)
    exp = 1  # 1s, 10s, 100s, 1000s...

    # Run stable counting sort for each digit position
    while max_val // exp > 0:
        arr = counting_sort_by_digit(arr, exp)
        exp *= 10

    return arr

# Test Transactions
transactions = [170, 45, 75, 90, 802, 24, 2, 66]
expected = sorted(transactions)

sorted_tx = radix_sort(transactions)

assert sorted_tx == expected
print("Radix sort executed and verified successfully!")
print(f"Sorted output: {sorted_tx}")
```
```text
Radix sort executed and verified successfully!
Sorted output: [2, 24, 45, 66, 75, 90, 170, 802]
```

---

## Common Mistakes

- **Using Counting Sort on unbounded ranges:** If values range up to $10^9$, Counting Sort will crash your program with an `OutOfMemoryError`. Only use Counting Sort when the value range $K$ is on the same order of magnitude as $N$ ($K \le N$).
- **Using an unstable subroutine in Radix Sort:** Radix Sort **strictly requires** its inner sorting subroutine to be stable. If you replace stable Counting Sort with an unstable sort (like Quick Sort), earlier sorted digits will be jumbled, completely destroying the final result.
- **Neglecting negative numbers:** Basic counting and radix sort assume non-negative integers. To handle negatives, you must offset values by adding $|\min|$ before sorting, and subtract it back afterward.
- **Assuming $O(N)$ is always faster than $O(N \log N)$:** For moderate $N$, the constant factors and auxiliary memory overhead of Radix Sort can make it slower than highly optimized C implementations like Timsort.

---

## Key Takeaway

The $\Omega(N \log N)$ theoretical lower bound only applies to comparison-based sorting algorithms. By abandoning comparisons and using values as direct array indices (**Counting Sort**), processing numbers digit by digit (**Radix Sort**), or distributing items into intervals (**Bucket Sort**), non-comparison sorts achieve genuine **$O(N)$ Linear Time**. However, they trade generality for speed: they strictly require discrete, bounded, or uniformly distributed data and consume auxiliary memory.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Non-Comparison Sort** | A sorting algorithm that determines order without pairwise element comparisons (`<` or `>`). |
| **Counting Sort** | An algorithm that sorts integers by tallying the frequency of each distinct value. |
| **Radix Sort** | An algorithm that sorts numbers or fixed-length strings digit-by-digit using a stable subroutine. |
| **Bucket Sort** | A distribution-based algorithm that divides a range into buckets, sorts each bucket, and concatenates them. |
| **Least Significant Digit (LSD)** | The rightmost, lowest-value digit of a number (e.g., the 1s place). |
| **Most Significant Digit (MSD)** | The leftmost, highest-value digit of a number. |

---

## References

- [GeeksforGeeks: Counting Sort Algorithm](https://www.geeksforgeeks.org/counting-sort/)
- [GeeksforGeeks: Radix Sort Algorithm](https://www.geeksforgeeks.org/radix-sort/)
- [Introduction to Algorithms (CLRS): Chapter 8 - Sorting in Linear Time](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
