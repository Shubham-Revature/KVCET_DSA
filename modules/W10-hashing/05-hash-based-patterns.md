# Hash-Based Patterns

In technical interviews and production system design, you will rarely be asked to implement a hash table from scratch. Programming languages already provide battle-tested, hyper-optimized dictionaries and sets.

The real challenge is **algorithmic pattern recognition**: identifying when a problem that appears to require $O(N^2)$ or $O(N^3)$ brute-force comparisons can be transformed into a linear **$O(N)$ solution** through clever hash lookups.

In this lesson, we master the four core **Hash-Based Patterns** that appear repeatedly across high-stakes coding interviews and scalable engineering architectures.

---

## Pattern 1: The Complement Lookup Pattern

The most iconic pattern in computer science transforms a two-element search from quadratic $O(N^2)$ time into linear $O(N)$ time.

> **Problem:** Given an array of numbers and a `target`, find two numbers that sum up to `target`.
> Example: `nums = [2, 7, 11, 15]`, `target = 9` $\implies$ `(2, 7)`.

### The Brute Force Trap
The naive approach uses nested loops to check every possible pair:
```python
# O(N^2) Brute Force
for i in range(len(nums)):
    for j in range(i + 1, len(nums)):
        if nums[i] + nums[j] == target:
            return (i, j)
```

### The Hash-Based Insight: Look Backward, Not Forward
Instead of searching forward through the rest of the array for a matching number, we store numbers we have **already visited** in a Hash Map (`seen`). 

For every number $X$, we ask: **"Did we already see $(target - X)$ in the past?"**

```text
Target: 9
Array:  [ 2,  7,  11,  15 ]

Step 1: Num = 2. Complement = 9 - 2 = 7.
        Is 7 in seen? No.
        Store: seen[2] = 0

Step 2: Num = 7. Complement = 9 - 7 = 2.
        Is 2 in seen? YES! (Found at index 0)
        Return (0, 1) in a single O(N) pass!
```

```python
def two_sum(nums: list[int], target: int) -> tuple[int, int] | None:
    seen = {}  # {value: index}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return (seen[complement], i)
        seen[num] = i
    return None
```

- **Time Complexity:** **$O(N)$** (Single pass; dictionary lookups are $O(1)$).
- **Space Complexity:** **$O(N)$** (Hash map storage).

---

## Pattern 2: Prefix Sum + Hash Map (Subarray Sums)

What if you need to find contiguous sequences inside an array?

> **Problem (LeetCode 560):** Given an array of integers (which may include negative numbers) and an integer $k$, find the **total number of continuous subarrays whose sum equals $k$**.
> Example: `nums = [1, 2, 3, -2, 2]`, `k = 3` $\implies$ `3` subarrays (`[1, 2]`, `[3]`, `[3, -2, 2]`).

### Why Sliding Window Fails
If all numbers were strictly positive, we could use the Sliding Window pattern from Week 6. But because the array contains **negative numbers**, the running sum does not change monotonically—expanding the window might decrease the sum! Sliding window fails completely.

### The Prefix Sum Identity
A **Prefix Sum** $P[j]$ is the sum of all elements from index $0$ to $j$. 

The sum of any contiguous subarray from index $i$ to index $j$ is given by:

$$\text{Subarray Sum}(i, j) = P[j] - P[i - 1]$$

We want this subarray sum to equal $k$:

$$P[j] - P[i - 1] = k \implies P[i - 1] = P[j] - k$$

```text
Current Running Prefix Sum: P[j]
Target Subarray Sum:        k
Needed Past Prefix Sum:     P[j] - k

If a past prefix sum of (P[j] - k) occurred C times, 
then exactly C valid subarrays ending at index j sum to k!
```

```python
def subarray_sum(nums: list[int], k: int) -> int:
    # Base Case: A prefix sum of 0 has occurred once (before any elements)
    prefix_counts = {0: 1}
    current_sum = 0
    total_subarrays = 0

    for num in nums:
        current_sum += num

        # Did a past prefix sum equal (current_sum - k)?
        if (current_sum - k) in prefix_counts:
            total_subarrays += prefix_counts[current_sum - k]

        # Record current prefix sum in hash map
        prefix_counts[current_sum] = prefix_counts.get(current_sum, 0) + 1

    return total_subarrays
```

> [!IMPORTANT]
> **The Secret Base Case `{0: 1}`:**
> Why must `prefix_counts` be initialized with `{0: 1}`?
> Suppose `nums = [3]` and `k = 3`. 
> At index 0, `current_sum = 3`. The complement is `3 - 3 = 0`. 
> If `{0: 1}` is present, the algorithm recognizes that the entire prefix from the start forms a valid subarray! Without `{0: 1}`, subarrays starting at index 0 would be completely missed!

---

## Pattern 3: Hash Set Boundary Expansion

Sometimes, an array contains an unordered scatter of numbers, and you need to find contiguous chains.

> **Problem (LeetCode 128):** Given an unsorted array of integers, find the length of the **longest consecutive elements sequence** in **$O(N)$ time**.
> Example: `nums = [100, 4, 200, 1, 3, 2]` $\implies$ Longest sequence is `[1, 2, 3, 4]`, length = **4**.

### Why Sorting is Forbidden
Sorting the array takes $O(N \log N)$ time. The problem explicitly demands an **$O(N)$ Linear Time** solution.

### The Hash Set Strategy: The Intelligent Start-Check
1. Insert all numbers into a `set` for instant $O(1)$ lookups.
2. Iterate through each number $X$ in the set.
3. **The Key Insight:** Only start counting a sequence if $X$ is the **absolute beginning** of that sequence! 
   How do you know if $X$ is the beginning? 
   **Check if $(X - 1)$ is in the set!**
   - If $(X - 1) \in \text{set}$: $X$ is part of an ongoing chain. **Skip it!**
   - If $(X - 1) \notin \text{set}$: $X$ is a sequence leader! Count forward: $X + 1, X + 2, X + 3 \dots$ until the chain breaks.

```text
Set: { 100, 4, 200, 1, 3, 2 }

- Check 100: Is 99 in set? No -> Leader! Chain: [100] (Length 1)
- Check 4:   Is 3 in set?  YES! -> Skip! (Not a leader)
- Check 200: Is 199 in set? No -> Leader! Chain: [200] (Length 1)
- Check 1:   Is 0 in set?  No -> Leader! Chain: [1, 2, 3, 4] (Length 4!)
- Check 3:   Is 2 in set?  YES! -> Skip!
- Check 2:   Is 1 in set?  YES! -> Skip!
```

### Why This is Strictly $O(N)$ Time:
Even though there is a `while` loop inside the `for` loop, each number is visited **at most twice**:
1. Once during the outer loop inspection.
2. Once during a forward chain traversal.
Numbers that are not leaders are skipped in $O(1)$ time!

```python
def longest_consecutive(nums: list[int]) -> int:
    num_set = set(nums)
    longest_streak = 0

    for num in num_set:
        # Only check if num is the start of a sequence
        if (num - 1) not in num_set:
            current_num = num
            current_streak = 1

            while (current_num + 1) in num_set:
                current_num += 1
                current_streak += 1

            longest_streak = max(longest_streak, current_streak)

    return longest_streak
```

---

## Pattern 4: Canonical Key Grouping & Isomorphism

When items are equivalent under a specific transformation (e.g., anagrams, structural isomorphisms), we map them to a **canonical form** that acts as a standardized dictionary key.

### Example: Isomorphic Strings (LeetCode 205)
Two strings `s` and `t` are **isomorphic** if the characters in `s` can be replaced to get `t`, while preserving character order and maintaining a strict one-to-one (bijective) mapping:
- `"egg"` and `"add"` $\implies$ True (`e` $\to$ `a`, `g` $\to$ `d`).
- `"foo"` and `"bar"` $\implies$ False (`o` cannot map to both `a` and `r`).
- `"badc"` and `"baba"` $\implies$ False (`d` and `b` cannot both map to `a`).

### The Two-Way Hash Map Strategy:
To guarantee a true bijection, we maintain two dictionaries (or map characters to their first-seen index):

```python
def is_isomorphic(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False

    map_s_to_t = {}
    map_t_to_s = {}

    for c1, c2 in zip(s, t):
        # Verify s -> t mapping
        if c1 in map_s_to_t and map_s_to_t[c1] != c2:
            return False
        # Verify t -> s mapping
        if c2 in map_t_to_s and map_t_to_s[c2] != c1:
            return False

        map_s_to_t[c1] = c2
        map_t_to_s[c2] = c1

    return True
```

---

## Try it Yourself

**Scenario:** Implement both the **Subarray Sum Equals $K$** solver and the **Longest Consecutive Sequence** solver to verify their $O(N)$ efficiency across test cases.

*Your Spec:*
1. Implement `subarray_sum(nums, k)` using the Prefix Sum + Hash Map pattern.
2. Implement `longest_consecutive(nums)` using the Hash Set boundary expansion pattern.
3. Validate both implementations against edge cases (negative numbers, empty arrays, single elements).

```python
# --- 1. Subarray Sum Equals K ---
def subarray_sum(nums: list[int], k: int) -> int:
    prefix_counts = {0: 1}
    current_sum = 0
    count = 0

    for num in nums:
        current_sum += num
        complement = current_sum - k

        if complement in prefix_counts:
            count += prefix_counts[complement]

        prefix_counts[current_sum] = prefix_counts.get(current_sum, 0) + 1

    return count

# Test Subarray Sum
assert subarray_sum([1, 1, 1], 2) == 2            # [1, 1] at (0,1) and (1,2)
assert subarray_sum([1, 2, 3], 3) == 2            # [1, 2] and [3]
assert subarray_sum([1, -1, 0], 0) == 3           # [1, -1], [0], and [1, -1, 0]
assert subarray_sum([3], 3) == 1                  # Single element equal to k
print("Subarray Sum Equals K tests passed successfully!")


# --- 2. Longest Consecutive Sequence ---
def longest_consecutive(nums: list[int]) -> int:
    num_set = set(nums)
    longest_streak = 0

    for num in num_set:
        # Only start sequence if num is the sequence leader
        if (num - 1) not in num_set:
            current_num = num
            current_streak = 1

            while (current_num + 1) in num_set:
                current_num += 1
                current_streak += 1

            longest_streak = max(longest_streak, current_streak)

    return longest_streak

# Test Longest Consecutive Sequence
assert longest_consecutive([100, 4, 200, 1, 3, 2]) == 4  # Sequence: [1, 2, 3, 4]
assert longest_consecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]) == 9  # [0, 1, 2, 3, 4, 5, 6, 7, 8]
assert longest_consecutive([]) == 0                       # Empty list
assert longest_consecutive([10]) == 1                     # Single element
print("Longest Consecutive Sequence tests passed successfully!")
```
```text
Subarray Sum Equals K tests passed successfully!
Longest Consecutive Sequence tests passed successfully!
```

---

## Common Mistakes

- **Forgetting `{0: 1}` in Prefix Sum Hash Maps:** Failing to initialize `prefix_counts = {0: 1}` means that any valid subarray starting at index 0 will be completely ignored, causing subtle test failures.
- **Omitting the sequence-start check in Longest Consecutive:** If you remove `if (num - 1) not in num_set:` and blindly run the `while` loop for every number, an array like `[5, 4, 3, 2, 1]` will perform $5 + 4 + 3 + 2 + 1 = 15$ checks, degrading the algorithm back to **$O(N^2)$ quadratic time**! The start-check is mandatory to guarantee $O(N)$.
- **One-way mapping in Isomorphic Strings:** Using only a single dictionary `map_s[c1] = c2` allows two different characters in `s` to map to the same character in `t` (e.g., `"badc"` $\to$ `"baba"`). You must maintain bidirectional 1-to-1 mappings.
- **Using list lookups instead of set lookups:** Writing `if (num - 1) in nums:` where `nums` is a Python `list` executes an $O(N)$ linear scan on every iteration, accidentally turning your code into an $O(N^2)$ bottleneck! Always convert to a `set` first.

---

## Key Takeaway

Hash tables turn search bottlenecks into instant constant-time operations. The four master patterns:
1. **Complement Lookup:** Converts nested pair searches from $O(N^2)$ to $O(N)$ by caching past elements.
2. **Prefix Sum + Hash Map:** Solves contiguous subarray problems on arrays with negative numbers in $O(N)$ time using the identity $P[j] - P[i-1] = k$.
3. **Set Boundary Expansion:** Finds consecutive chains in unsorted arrays in $O(N)$ time by only counting from verified sequence leaders (`num - 1 not in set`).
4. **Canonical Mapping:** Standardizes structural equivalence (anagrams, isomorphic strings) into deterministic dictionary keys.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Complement** | The required value that, when paired with the current element, satisfies a target sum ($target - X$). |
| **Prefix Sum** | The cumulative sum of elements from the start of an array up to the current index. |
| **Subarray** | A contiguous, unbroken sequence of elements within an array. |
| **Sequence Leader** | An element $X$ in a set for which $X - 1$ does not exist, marking the beginning of a consecutive chain. |
| **Bijection** | A one-to-one and onto mapping between two sets where every element in set A maps to a unique element in set B. |

---

## References

- [LeetCode 1: Two Sum](https://leetcode.com/problems/two-sum/)
- [LeetCode 560: Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
- [LeetCode 128: Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)
- [LeetCode 205: Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)
