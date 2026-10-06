# Recursion and Amortized Cost

We have analyzed simple loops to find our Big-O time and space complexity. But what happens when an algorithm doesn't use a loop, and instead a function calls itself? What happens when a piece of code is incredibly fast 99% of the time, but incredibly slow 1% of the time? 

To complete your foundational understanding of complexity analysis, we need to introduce two new concepts: analyzing Recursion, and understanding Amortized Cost.

---

## 1. The Complexity of Recursion

**Recursion** is when a function calls itself to solve a smaller piece of the same problem. We will dedicate an entire week to this later, but for now, you must understand how it affects complexity.

### Time Complexity
To find the time complexity of a recursive algorithm, you generally ask: **How many times will this function call itself?**

```python
def count_down(n):
    if n == 0:
        return
    print(n)
    count_down(n - 1)
```
If we call `count_down(5)`, the function will call itself for 4, 3, 2, 1, and 0. It makes $N$ function calls. Therefore, the time complexity is **$O(N)$**.

### Space Complexity (The Call Stack)
This is a classic interview trap. Look at the `count_down` code above. Notice that we didn't create any new lists or dictionaries. So the Auxiliary Space must be $O(1)$, right? **Wrong.**

When a function calls itself, Python has to pause the current function and remember exactly where it was before jumping into the new function call. It stores this "memory" in a hidden data structure called the **Call Stack**.

If a function recurses $N$ times, there will be $N$ paused functions stacked on top of each other in memory. Therefore, the auxiliary space complexity of our simple recursion is **$O(N)$**.

---

## 2. Amortized Cost

Sometimes, a specific operation isn't perfectly $O(1)$ or perfectly $O(N)$. What if an operation is instantaneous 99% of the time, but massive and slow 1% of the time?

We use the concept of **Amortized Cost** to average out that rare penalty over time. 

### The Dynamic Array Example
The perfect example of amortized cost is Python's built-in list (`.append()`).

Under the hood, when you create a list, Python reserves a fixed block of memory (e.g., 4 slots).
1. `append(A)` -> Fast! $O(1)$.
2. `append(B)` -> Fast! $O(1)$.
3. `append(C)` -> Fast! $O(1)$.
4. `append(D)` -> Fast! $O(1)$. The list is now full.
5. `append(E)` -> Uh oh. 

To add 'E', Python has to pause, go find a brand new, larger block of memory (e.g., 8 slots), copy A, B, C, and D over to the new block, and *then* append E. That copying process took $O(N)$ time.

However, because Python doubled the size of the array, you now have 3 free slots. Your next three appends will be guaranteed $O(1)$. Because the $O(N)$ penalty happens so rarely, and it "pays" for many future $O(1)$ operations, we average the cost out. 

We confidently say that the **Amortized Time Complexity of `.append()` is $O(1)$**.

---

## Try it Yourself

**Scenario:** Look at the two functions below. They both calculate the sum of numbers from 1 to $N$.

**Algorithm A (Iterative):**
```python
def sum_iterative(n):
    total = 0
    for i in range(1, n + 1):
        total += i
    return total
```

**Algorithm B (Recursive):**
```python
def sum_recursive(n):
    if n == 1:
        return 1
    return n + sum_recursive(n - 1)
```

**(a) What is the Time Complexity for both?**
*Answer:* Both algorithms require $N$ steps to reach the answer. Both have a Time Complexity of **$O(N)$**.

**(b) What is the Auxiliary Space Complexity for both?**
*Answer:* 
- Algorithm A only creates a single `total` variable. Its space complexity is **$O(1)$**. 
- Algorithm B calls itself $N$ times, forcing Python to remember $N$ suspended functions on the Call Stack. Its space complexity is **$O(N)$**. 
- This proves why iterative loops are generally more memory-efficient than recursion!

---

## Common Mistakes

- **Forgetting the Call Stack:** The most common mistake juniors make is claiming a recursive algorithm has $O(1)$ space complexity because they didn't manually create a list. Always account for the memory the computer uses to manage the recursion.
- **Assuming `.append()` is purely $O(1)$:** While we treat it as $O(1)$ for general analysis, you should know for advanced system design that it occasionally spikes to $O(N)$. This is why high-performance, real-time systems (like self-driving cars or trading algorithms) sometimes avoid dynamic arrays to prevent unpredictable delays.

---

## Key Takeaway

Time and space complexity analysis extends beyond simple loops. Recursive algorithms hide memory usage in the computer's **Call Stack**, often resulting in $O(N)$ auxiliary space. Furthermore, some operations like `.append()` have a heavy $O(N)$ penalty that happens so rarely we can average it out, resulting in an **Amortized** $O(1)$ complexity. 

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Recursion** | A technique where a function calls itself to solve a smaller instance of the same problem. |
| **Call Stack** | A hidden memory structure the computer uses to keep track of active, paused functions (crucial for recursion). |
| **Amortized Cost** | The average time taken per operation, used when an operation is mostly fast but occasionally triggers a slow resize/rebuild process. |

---

## References
- [FreeCodeCamp: Recursion in Python](https://www.freecodecamp.org/news/how-recursion-works-explained-with-flowcharts-and-a-video-de61f40cb7f9/)
- [StackOverflow: What is Amortized Analysis?](https://stackoverflow.com/questions/200384/constant-amortized-time)
