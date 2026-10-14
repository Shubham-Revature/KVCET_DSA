# Binary Search on the Answer

So far, we have used Binary Search to find a specific value inside a physical, sorted array. 

However, top-tier technical interviews often present problems where **no sorted array is provided at all**. Instead, you are given an open-ended optimization challenge:
- *"What is the minimum speed needed to finish reading all files before a deadline?"*
- *"What is the minimum cargo ship capacity required to deliver all packages within $D$ days?"*
- *"What is the maximum allocation of resources you can distribute fairly?"*

At first glance, these problems sound like complex Dynamic Programming or Greedy puzzles. But with the right perspective, they can be solved using one of the most powerful paradigms in computer science: **Binary Search on the Answer** (also known as searching over the solution space).

---

## 1. The Core Insight: Monotonicity

The secret to Binary Search has nothing to do with arrays. **Binary Search only requires Monotonicity.**

A problem is monotonic if its truth value transitions predictably in a single direction without flip-flopping:

```text
Capacity (C):     10     11     12     13     14  │  15    16    17    18
Possible in 5d?  False  False  False  False  False │ True  True  True  True
                                                   ▲
                                        First True = Answer (15)
```

Look at the pattern:
- If a ship capacity of `14` fails to deliver all packages on time, **every capacity smaller than 14 is guaranteed to fail**. There is no need to test 13, 12, or 10!
- If a ship capacity of `16` succeeds, **every capacity larger than 16 is guaranteed to succeed**.

Because the feasibility function transitions monotonically from `False` to `True`, we can run Binary Search directly over the **range of possible capacities** to find the exact boundary—the minimum valid answer!

---

## 2. The Two-Part Architecture

Every "Binary Search on the Answer" solution consists of two cleanly separated components:

### Component 1: The Feasibility Checker (`is_possible(candidate)`)
A helper function that takes a trial answer (`mid`) and returns `True` or `False`. It simulates the problem logic (usually in **$O(N)$ Linear Time**) using a simple greedy simulation.

### Component 2: The Binary Search Wrapper
Defines the absolute minimum (`low`) and maximum (`high`) conceivable answers, and uses binary search to pinpoint the optimal threshold:

```text
                     mid = (low + high) // 2
                               │
                    is_possible(mid)?
                     ╱             ╲
               [YES / TRUE]      [NO / FALSE]
                    │                  │
           Answer might work!    Too small / slow!
          Save: result = mid     Must increase candidate:
       Try smaller: high = mid-1        low = mid + 1
```

---

## 3. Classic Walkthrough: Capacity to Ship Packages

**Problem:** You have a conveyor belt with packages of varying weights:
$$\text{weights} = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]$$
You must ship all packages within **$D = 5$ days**. Packages must be loaded in the order given. What is the **minimum ship capacity** required?

### Step 1: Define Search Boundaries (`low` and `high`)
- **Minimum conceivable capacity (`low`):** $\max(\text{weights}) = 10$. The ship must be able to carry the heaviest individual package. If capacity were 9, package `10` could never be shipped!
- **Maximum conceivable capacity (`high`):** $\sum(\text{weights}) = 55$. If the ship has a capacity of 55, it can deliver everything in a single day.
- **Search Space:** Any integer between `10` and `55`.

### Step 2: The $O(N)$ Feasibility Simulation
To test whether capacity `C` can ship all packages within 5 days:
1. Load packages one by one onto the current day's ship.
2. If adding the next package exceeds capacity `C`, send the ship off, increment the day counter, and start a new ship.
3. If total days $\le 5$, return `True`; otherwise, return `False`.

### Step 3: Binary Search Execution
1. Midpoint of $[10, 55]$ is `32`. Can we ship in 5 days with capacity 32? **Yes** (takes only 2 days). Save 32, search left $[10, 31]$.
2. Midpoint of $[10, 31]$ is `20`. Can we ship with capacity 20? **Yes** (takes 4 days). Save 20, search left $[10, 19]$.
3. Midpoint of $[10, 19]$ is `14`. Can we ship with capacity 14? **No** (takes 6 days, too slow!). Search right $[15, 19]$.
4. Midpoint of $[15, 19]$ is `17`. Can we ship with capacity 17? **Yes**. Save 17, search left $[15, 16]$.
5. Midpoint of $[15, 16]$ is `15`. Can we ship with capacity 15? **Yes** (takes exactly 5 days). Save 15, search left $[15, 14]$.
6. Loop terminates. **Minimum capacity = 15!**

---

## 4. Complexity Analysis

Let $N$ be the number of packages and $R = (\text{high} - \text{low})$ be the range of possible answers:

- **Brute Force:** Testing every candidate from 10 to 55 takes $O(N \times R)$ time. If packages sum to $10^9$, this takes billions of operations and times out.
- **Binary Search on the Answer:** 
  - Number of search steps: $\log_2(R) \approx \log_2(55 - 10) \approx 6$ steps.
  - Feasibility check per step: $O(N)$.
  - **Total Time Complexity:** **$O(N \log R)$**.
  - **Space Complexity:** **$O(1)$** Auxiliary Space.

Even if the answer range spans from $1$ to $10^9$, $\log_2(10^9)$ is only **30 iterations**!

---

## Try it Yourself

**Scenario:** Implement the package shipping capacity solver to find the minimal ship capacity required to deliver all cargo within a deadline.

*Your Spec:*
1. Implement `can_ship(weights, capacity, max_days)` which returns `True` if all weights can be shipped within `max_days`.
2. Implement `ship_within_days(weights, days)` using binary search over the range `[max(weights), sum(weights)]`.

```python
def can_ship(weights: list[int], capacity: int, max_days: int) -> bool:
    days_used = 1
    current_load = 0
    
    for w in weights:
        if current_load + w > capacity:
            days_used += 1
            current_load = w
        else:
            current_load += w
            
    return days_used <= max_days


def ship_within_days(weights: list[int], days: int) -> int:
    # 1. Establish search boundaries
    low = max(weights)      # Must at least hold the single heaviest package
    high = sum(weights)     # Can carry everything in 1 day
    result = high

    # 2. Binary search over the answer space
    while low <= high:
        mid = low + (high - low) // 2

        if can_ship(weights, mid, days):
            result = mid       # Feasible! Record candidate and try smaller
            high = mid - 1
        else:
            low = mid + 1      # Impossible! Capacity is too small

    return result

# Test Cargo
cargo = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Should require minimum capacity of 15 for 5 days
assert ship_within_days(cargo, 5) == 15

# If given only 1 day, capacity must equal sum of all weights (55)
assert ship_within_days(cargo, 1) == 55

# If given 10 days, capacity only needs to hold the maximum single item (10)
assert ship_within_days(cargo, 10) == 10

print("All shipping capacity test cases passed successfully!")
```
```text
All shipping capacity test cases passed successfully!
```

---

## Common Mistakes

- **Incorrect `low` bound:** Setting `low = 0` or `low = min(weights)`. If `capacity < max(weights)`, an individual package can never fit on the ship, causing an infinite loop or incorrect day counts in the feasibility check. Always set `low = max(weights)` for capacity problems.
- **Applying to non-monotonic problems:** If a problem's feasibility fluctuates (e.g., True for 5, False for 6, True for 7), Binary Search **cannot** be applied. Ensure the problem exhibits clean monotonic progression before using this pattern.
- **Forgetting to record the candidate result:** When `can_ship(mid)` returns `True`, immediately save `result = mid`. If you simply adjust `high = mid - 1` without recording `mid`, you might lose your best valid answer if subsequent iterations fail.
- **Overcomplicating the search space:** Don't spend excessive time calculating exact bounds. As long as `low` is guaranteed to be $\le$ the answer and `high` is guaranteed to be $\ge$ the answer, $\log_2(\text{range})$ shrinks huge intervals in milliseconds.

---

## Key Takeaway

**Binary Search on the Answer** is a transformative algorithmic paradigm used to solve optimization problems without an explicit sorted array. As long as the problem's validity function is **monotonic** (transitions cleanly from False to True or vice versa), we can binary search across the **range of possible numerical solutions**. Combined with an $O(N)$ greedy feasibility check, it turns brute-force $O(N \times R)$ searches into lightning-fast **$O(N \log R)$** solutions.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Binary Search on Answer** | Applying binary search over a continuous or discrete numerical solution space rather than an array. |
| **Monotonicity** | A property where a function's output changes in a single direction (e.g., always non-decreasing or transitioning from False to True once). |
| **Feasibility Function** | A helper predicate (e.g., `can_ship(mid)`) that verifies whether a candidate solution satisfies problem constraints. |
| **Solution Space ($R$)** | The interval between the minimum conceivable answer (`low`) and the maximum conceivable answer (`high`). |

---

## References

- [LeetCode 1011: Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)
- [GeeksforGeeks: Binary Search on Answer Space](https://www.geeksforgeeks.org/binary-search-on-answer-with-examples/)
