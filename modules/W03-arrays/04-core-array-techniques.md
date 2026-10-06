# Core Array Techniques

Now that we understand how arrays are stored in memory and the exact cost of Python's built-in operations, we can start writing our own optimized algorithms. 

To pass technical interviews, you cannot rely entirely on Python's built-in functions. Interviewers want to see if you can manipulate data manually. In this section, we will look at two foundational array techniques: **In-Place Swapping** and **Prefix Sums**.

---

## 1. The Classic Swap

How do you swap two elements in an array? If you simply say `array[0] = array[1]`, you have destroyed the original value at `array[0]`.

In most programming languages, you have to use a temporary variable to hold one of the values while you overwrite it.

```python
# The standard programming swap
def swap_standard(arr, i, j):
    temp = arr[i]       # Hold the value safely
    arr[i] = arr[j]     # Overwrite the first slot
    arr[j] = temp       # Put the held value in the second slot
```

### Python's Swap Shortcut
Python provides a very clean, "Pythonic" way to swap two variables on a single line without explicitly creating a `temp` variable. Under the hood, Python is essentially creating a temporary tuple and unpacking it.

```python
def swap_pythonic(arr, i, j):
    arr[i], arr[j] = arr[j], arr[i]
```

---

## 2. In-Place Reversal (Preview to Two-Pointers)

A common interview question is to reverse an array *in-place*. 
- **Out-of-place (Bad):** You create an empty list, loop backwards through the original, and `.append()` to the new list. This takes $O(N)$ auxiliary space.
- **In-place (Good):** You modify the original array without creating a new one. This takes $O(1)$ auxiliary space.

We can achieve this by combining our Swap technique with a simple Two-Pointer logic (which we will cover fully next week). We place a pointer at the start and a pointer at the end, swap their values, and move them towards the middle.

```python
def reverse_in_place(arr):
    left = 0
    right = len(arr) - 1
    
    while left < right:
        # Swap the outer edges
        arr[left], arr[right] = arr[right], arr[left]
        # Move pointers inward
        left += 1
        right -= 1
        
    return arr
```
This algorithm reverses the array in **$O(N)$ Time** and **$O(1)$ Space**.

---

## 3. Prefix Sums

Imagine you have an array of daily sales data for a year (365 integers). Your boss asks you for the total sales between Day 10 and Day 20. Then they ask for the total between Day 50 and Day 100. 

If you write a standard loop for every query, the time complexity is $O(N)$ for every single question. If they ask 10,000 questions, the total time is $O(Q \times N)$ where Q is the number of queries.

We can optimize this heavily using a **Prefix Sum Array**. 
A prefix sum array stores the *running total* of all elements up to that index.

**Original Array:** `[2, 4, 1, 5, 3]`
**Prefix Sum Array:** `[2, 6, 7, 12, 15]`

Why is this useful? If we want the sum of indices 1 through 3 (`4 + 1 + 5 = 10`), we don't have to loop. We just take the prefix sum at index 3 (`12`) and subtract the prefix sum just before our range at index 0 (`2`). 
$12 - 2 = 10$.

By building the Prefix Sum array once (which takes $O(N)$ time), we can now answer an infinite number of sum queries in **$O(1)$ Constant Time**!

---

## Try it Yourself

**Scenario:** You need to build a Prefix Sum array in Python.

Write a function `build_prefix(arr)` that takes an array and returns a new array where every element is the running total. 

*Your Spec:*
1. Create a new array called `prefix`.
2. Keep a `running_total` variable starting at 0.
3. Loop through the original array.
4. For each number, add it to the `running_total`.
5. Append the `running_total` to the `prefix` array.
6. Return `prefix`.

```python
def build_prefix(arr):
    prefix = []
    running_total = 0
    for num in arr:
        running_total += num
        prefix.append(running_total)
    return prefix

assert build_prefix([2, 4, 1, 5, 3]) == [2, 6, 7, 12, 15]
print("Prefix sum successful!")
```
```text
Prefix sum successful!
```

---

## Common Mistakes

- **Using `.reverse()` in an interview:** If an interviewer asks you to reverse an array, they usually want to see you write the Two-Pointer logic shown above. If you just write `arr.reverse()`, they will ask you to do it again manually.
- **Recomputing sums in a loop:** Anytime you see a problem asking you to repeatedly sum a subarray or calculate a running balance, your brain should immediately think "Prefix Sums". Writing a loop inside another loop for this will result in a $O(N^2)$ Time Limit Exceeded error.

---

## Key Takeaway

Arrays are powerful when manipulated correctly. **In-place operations** allow you to modify arrays (like reversing them) using only $O(1)$ auxiliary space by swapping values. **Prefix sums** are a classic technique where you spend $O(N)$ time upfront to pre-calculate running totals, which allows you to answer complex subarray sum queries instantly in $O(1)$ time. 

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **In-Place Modification** | Altering the contents of a data structure directly without creating a new copy ($O(1)$ space). |
| **Two-Pointer Technique** | Using two variables to keep track of two different indices in an array simultaneously. |
| **Prefix Sum** | An array that stores the cumulative sum of the elements of a given array up to each index. |

---

## References
- [GeeksforGeeks: Prefix Sum Array](https://www.geeksforgeeks.org/prefix-sum-array-implementation-applications-competitive-programming/)
