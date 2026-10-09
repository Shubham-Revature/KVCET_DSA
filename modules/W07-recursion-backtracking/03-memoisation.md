# Memoisation (Caching)

In the last lesson, we saw that naïve Tree Recursion (like our Fibonacci function) results in an incredibly slow **$O(2^N)$ Exponential Time** complexity. 

The problem is that the function has amnesia. It calculates `fib(5)`, and then completely forgets the answer. When the right side of the tree asks for `fib(5)` again, it has to redo all the exact same math from scratch. 

To fix this, we need to give our function a memory. This technique is called **Memoisation**.

---

## 1. What is Memoisation?

Memoisation (from the word "memorandum", meaning "to be remembered") is an optimization technique used to speed up computer programs. 

The logic is simple: **Never calculate the exact same input twice.**
If a function requires intense math to solve a problem, we should save the answer to a dictionary. If the function is ever called again with that exact same input, we just pull the answer straight from the dictionary in $O(1)$ time!

---

## 2. Implementing Memoisation

To memoise our Fibonacci function, we just need to pass a dictionary (which we'll call `memo`) into our function. 

**The 3-Step Memoisation Process:**
1. **Check:** Before doing any recursive math, check if the input `n` already exists as a key in the `memo` dictionary. If it does, return the saved answer instantly!
2. **Calculate:** If it isn't in the dictionary, do the standard recursive math.
3. **Save:** Before returning the final calculated answer, save it to the dictionary so you never have to calculate it again.

```python
def fib_memo(n, memo=None):
    # Initialize the dictionary on the first run
    if memo is None:
        memo = {}
        
    # 1. CHECK: Have we seen this before?
    if n in memo:
        return memo[n]
        
    # Standard Base Cases
    if n == 0: return 0
    if n == 1: return 1
        
    # 2. CALCULATE: Do the standard tree recursion
    answer = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    
    # 3. SAVE: Store the answer for future branches to use
    memo[n] = answer
    
    return answer
```

---

## 3. The Time Complexity Magic

By adding those three simple steps, the right side of the recursion tree gets completely pruned (cut off). The computer zips down the left side of the tree, calculates the answers once, saves them to the dictionary, and when the right side of the tree asks for them, they are instantly returned in $O(1)$ time.

This drops our Time Complexity from an unplayable **$O(2^N)$ Exponential Time** down to a lightning-fast **$O(N)$ Linear Time**.

---

## Try it Yourself

**Scenario:** In the last lesson, we saw that the naïve `fib(10)` took exactly **177 function calls** to complete. Let's run our memoised version with a call counter to see the difference.

*Your Spec:*
1. Set a global `calls = 0`.
2. Write the `fib_memo` function from above, incrementing `calls` at the very top.
3. Call `fib_memo(10)`.

```python
calls = 0

def fib_memo(n, memo=None):
    global calls
    calls += 1
    
    if memo is None: memo = {}
    if n in memo: return memo[n]
    if n == 0: return 0
    if n == 1: return 1
        
    answer = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    memo[n] = answer
    return answer

result = fib_memo(10)
print(f"The 10th Fibonacci number is {result}.")
print(f"It took {calls} function calls to find it!")

assert result == 55
```
```text
The 10th Fibonacci number is 55.
It took 19 function calls to find it!
```
*We went from 177 calls down to 19 calls! If we ran `fib(50)`, we would go from 20 billion calls down to just 99 calls. That is the power of Memoisation.*

---

## Common Mistakes

- **The Mutable Default Argument Trap:** A very common mistake in Python is writing `def fib(n, memo={}):`. Because dictionaries are mutable, Python will use the *exact same dictionary* across entirely different function calls in your program. This can cause weird bugs if you try to calculate different sequences. Always use `memo=None` and create the dictionary inside the function!
- **Python's Built-in Cheat Code:** In a real-world Python job, you rarely write manual `memo` dictionaries. You can just import Python's built-in decorator! If you put `@lru_cache(maxsize=None)` directly above your function, Python will automatically memoise the function for you behind the scenes! *(However, you must know how to do it manually for technical interviews).*

---

## Key Takeaway

Tree Recursion often suffers from $O(2^N)$ Exponential Time because it recalculates overlapping sub-problems. **Memoisation** is an optimization technique that gives the function a "memory" using a dictionary. By checking the dictionary before doing any math, and saving answers to the dictionary after doing math, we can prune the recursion tree and reduce the time complexity down to $O(N)$.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Memoisation** | An optimization technique where the results of expensive function calls are stored and reused. |
| **Overlapping Sub-problems** | When a recursive algorithm repeatedly calculates the exact same input branches. |
| **`@lru_cache`** | A built-in Python decorator from the `functools` module that automatically memoises a function. |

---

## References
- [GeeksforGeeks: Memoization (1D, 2D and 3D)](https://www.geeksforgeeks.org/memoization-1d-2d-and-3d/)
