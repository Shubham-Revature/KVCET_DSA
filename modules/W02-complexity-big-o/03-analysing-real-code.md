# Analyzing Real Code

In the previous topic, we covered the theory of Asymptotic Notation. Now, it is time to apply those golden rules to real Python code. When you write a function, you need to be able to scan it and confidently declare its Big-O Time Complexity. 

In this section, we will look at the three most common time complexities you will encounter as a junior engineer, and how to spot them instantly in your code.

---

## 1. Constant Time: $O(1)$

If a block of code does not depend on the size of the input data ($N$), it runs in **Constant Time** or $O(1)$. It does not matter if the input list has 10 items or 10 billion items; the operation takes the exact same amount of time.

```python
def get_first_item(items):
    # This takes 1 operation, regardless of how long the 'items' list is.
    return items[0] 

def do_math(n):
    # Math operations are constant time.
    x = n * 10
    y = x / 2
    return y
```
Both of the functions above are $O(1)$.

---

## 2. Linear Time: $O(N)$

If your code has to look at every item in the input exactly once (or a constant number of times), it runs in **Linear Time** or $O(N)$. A single `for` loop or `while` loop that iterates over a list is the classic example of $O(N)$.

```python
def print_all_items(items):
    for item in items:     # This loop runs N times
        print(item)        # This is an O(1) operation
```
Because the $O(1)$ print statement happens $N$ times, the total complexity is $O(1) \times N = O(N)$.

### Sequential Loops
What if you have two separate loops, one after the other?

```python
def print_twice(items):
    for item in items:     # Runs N times
        print(item)
        
    for item in items:     # Runs N times
        print(item)
```
The total operations are $N + N = 2N$. Using the First Golden Rule of Big-O (Drop Constants), we drop the 2. The complexity is still just **$O(N)$**.

---

## 3. Quadratic Time: $O(N^2)$

If your code has a loop *inside* of another loop, and both loops depend on the input size $N$, it runs in **Quadratic Time** or $O(N^2)$. 

For every single item in the outer loop, the inner loop must run completely.

```python
def print_pairs(items):
    for i in items:             # Outer loop runs N times
        for j in items:         # Inner loop runs N times
            print(i, j)         # O(1) operation
```
Because the inner loop runs $N$ times, and the outer loop runs $N$ times, the total operations are $N \times N = N^2$. Therefore, the complexity is **$O(N^2)$**.

As we learned earlier, an $O(N^2)$ algorithm will easily trigger a Time Limit Exceeded error if $N$ is large (like 100,000).

---

## Try it Yourself

**Scenario:** Analyze the following Python function. What is its overall Big-O Time Complexity?

```python
def process_data(items):
    # Part 1
    first = items[0]
    
    # Part 2
    for item in items:
        print(item)
        
    # Part 3
    for x in items:
        for y in items:
            print(x, y)
```

*Answer:*
1. Part 1 is $O(1)$.
2. Part 2 is a single loop: $O(N)$.
3. Part 3 is a nested loop: $O(N^2)$.
4. Total mathematical complexity: $f(N) = N^2 + N + 1$.
5. Apply the Second Golden Rule (Drop non-dominant terms). $N^2$ is the dominant term. We drop the $N$ and the $1$.
6. **Final Complexity: $O(N^2)$**.

---

## Common Mistakes

- **Mistaking a fixed loop for $O(N)$:** If you write `for i in range(5): print("Hello")`, that loop always runs exactly 5 times, regardless of the input. Because it does not scale with $N$, that specific loop is $O(1)$ constant time, not $O(N)$.
- **Assuming all nested loops are $O(N^2)$:** If the inner loop is fixed (e.g., `for i in items: for j in range(3):`), the total operations are $N \times 3 = 3N$. Dropping the constant gives us $O(N)$. For it to be $O(N^2)$, *both* loops must scale with $N$.

---

## Key Takeaway

To quickly find the Big-O Time Complexity of real code: look at the loops. No loops means $O(1)$ constant time. A single loop over the data means $O(N)$ linear time. A loop nested inside another loop means $O(N^2)$ quadratic time. If there are multiple parts to the function, find the single slowest part (the dominant term) and drop the rest.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **$O(1)$ (Constant Time)** | The execution time does not change as the input gets larger. |
| **$O(N)$ (Linear Time)** | The execution time grows in direct proportion to the input size. |
| **$O(N^2)$ (Quadratic Time)** | The execution time grows in proportion to the square of the input size (usually a nested loop). |
| **Dominant Term** | The part of the algorithm that takes the most time and dictates the final Big-O complexity. |

---

## References
- [FreeCodeCamp: Big O Notation in Python](https://www.freecodecamp.org/news/big-o-notation-why-it-matters-and-why-it-doesnt-1674cfa8a23c/)
