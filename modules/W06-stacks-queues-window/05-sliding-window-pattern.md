# The Sliding Window Pattern

In Week 4, you learned your first algorithmic pattern: the Two-Pointer technique. Now, it is time for your second. 

The **Sliding Window Pattern** is a crucial strategy designed to turn slow $O(N^2)$ nested loops into blazing fast $O(N)$ solutions. You should immediately think of this pattern anytime an interview question contains the phrases **"contiguous subarray"** or **"substring"**.

---

## 1. What is a Sliding Window?

Imagine you have an array of 10 numbers, and you are asked to find the largest sum of any 3 consecutive numbers.

**The Bad Way ($O(N^2)$):**
You start at index 0, add the next 3 numbers. Then you move to index 1, and add the next 3 numbers from scratch. Then index 2, and add from scratch again. If the required size was 1,000 instead of 3, you would be doing massive amounts of repetitive math!

**The Sliding Window Way ($O(N)$):**
Imagine a physical picture frame (the window) placed over the first 3 items. To move to the next group, you don't build a new frame. You simply slide the frame one space to the right. 
- You look at the single new number that entered the right side of the frame, and **Add** it to your total.
- You look at the single old number that fell out of the left side of the frame, and **Subtract** it from your total.

By only doing two simple math operations (`+` and `-`) per slide, every step takes $O(1)$ time, regardless of how big the window is!

---

## 2. Fixed vs. Dynamic Windows

There are two main variations of this pattern:

### Fixed Window
The problem explicitly gives you the size of the window (e.g., "Find the max sum of *any 3* elements").
- The window size never changes.
- You build the first window manually.
- Then, you slide it across the array exactly one step at a time.

### Dynamic Window
The problem asks you to *find* the size of the window (e.g., "Find the *longest* subarray that adds up to exactly 10").
- The window grows and shrinks like an accordion.
- You expand the right side of the window to let more numbers in until you hit your target.
- If you go over your target, you shrink the left side of the window to kick numbers out until you are back under the target.

---

## Try it Yourself

**Scenario:** Let's write the code for the Fixed Window problem we discussed above. We want to find the maximum sum of any contiguous subarray of size `k=3`.

*Your Spec:*
1. Calculate the sum of the very first window (indices 0, 1, 2) and save it as `current_sum`.
2. Save this as your `max_sum` as well.
3. Start a loop from index `k` to the end of the array.
4. For every step, slide the window: subtract the number that fell out on the left (`arr[i - k]`), and add the new number on the right (`arr[i]`).
5. Update `max_sum` if `current_sum` is higher.

```python
def max_subarray_sum(arr, k):
    if len(arr) < k:
        return 0

    # 1. Build the first window manually
    current_sum = sum(arr[:k])
    max_sum = current_sum
    
    # 3. Slide the window across the rest of the array
    for i in range(k, len(arr)):
        # 4. Add the new item on the right, subtract the old item on the left
        current_sum = current_sum + arr[i] - arr[i - k]
        
        # 5. Update the max
        max_sum = max(max_sum, current_sum)
        
    return max_sum

assert max_subarray_sum([2, 1, 5, 1, 3, 2], 3) == 9  # [5, 1, 3] = 9
print("Fixed Sliding Window successful!")
```
```text
Fixed Sliding Window successful!
```

---

## Common Mistakes

- **Recalculating the window from scratch:** The entire purpose of the Sliding Window is to use the `current_sum + new_right - old_left` math trick to save time. If you write `sum(arr[left:right])` inside your `while` loop, you are forcing Python to do a hidden $O(N)$ loop on every single step. This defeats the whole pattern and results in a failing $O(N^2)$ solution.
- **Forgetting to build the first window:** For a Fixed Window, you must manually calculate the sum of the first `k` elements *before* your main loop starts sliding. If you don't initialize the window, you have nothing to slide!

---

## Key Takeaway

The **Sliding Window Pattern** is the optimal way to solve problems asking for a "contiguous subarray" or "substring". By maintaining a "window" and only calculating the difference when the window moves (adding the new element, dropping the old element), you eliminate repetitive work and reduce $O(N^2)$ nested loops down to highly efficient **$O(N)$ Linear Time** algorithms.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Contiguous Subarray** | A continuous, unbroken slice of an array (e.g., indices 2, 3, and 4). |
| **Fixed Window** | A sliding window pattern where the size ($k$) is given, and the window simply shifts left to right. |
| **Dynamic Window** | A sliding window pattern where the window grows (expands right) and shrinks (contracts left) to find an optimal size based on a condition. |

---

## References
- [GeeksforGeeks: Window Sliding Technique](https://www.geeksforgeeks.org/window-sliding-technique/)
