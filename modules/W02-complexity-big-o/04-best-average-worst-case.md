# Best, Average, and Worst Case

So far, we have looked at how an algorithm's execution time grows as the input size ($N$) gets larger. However, for many algorithms, the execution time doesn't just depend on the *size* of the input, but also on the *actual data* inside it. 

To give a complete picture of an algorithm's efficiency, computer scientists analyze it under three different scenarios: the best case, the average case, and the worst case.

---

## 1. The Three Scenarios (Linear Search)

Imagine you have a list of $N$ unread emails, and you are looking for an email from your boss. You start at the top and check them one by one. This algorithm is called **Linear Search**.

```python
def find_email(inbox, target_sender):
    for i in range(len(inbox)):
        if inbox[i] == target_sender:
            return f"Found at index {i}!"
    return "Not found."
```

Depending on where the email actually is, the number of operations changes drastically.

### The Best Case
What if the email from your boss is the **very first email** in your inbox?
- The loop runs exactly 1 time.
- The algorithm finishes instantly.
- The best-case time complexity is **$O(1)$**.

### The Worst Case
What if the email from your boss is the **very last email**, or they never sent one at all?
- You have to check every single email.
- The loop runs $N$ times.
- The worst-case time complexity is **$O(N)$**.

### The Average Case
What if you run this algorithm every day for a year? Statistically, the email will be somewhere in the middle.
- On average, you will check $N / 2$ emails.
- Using our Big-O Golden Rule (Drop Constants), we drop the division by 2 (which is mathematically $1/2 \times N$). 
- The average-case time complexity is **$O(N)$**.

---

## 2. Why We Focus on the Worst Case

If an interviewer asks you, "What is the time complexity of Linear Search?", they do not want to hear $O(1)$. 

In software engineering, we must prepare for the worst. If you build a web server and assume the best case, your server will crash the moment user data gets messy. We engineer for the worst case to guarantee stability. Unless an interviewer explicitly asks for the best or average case, **always answer with the worst-case Big-O complexity**.

*(Note: There are a few rare exceptions, like the Quicksort algorithm, where the average case is heavily used in industry because the worst case is incredibly rare. We will cover this in Week 9).*

---

## Try it Yourself

**Scenario:** Analyze the following function. It takes a list of numbers and checks if they are perfectly sorted from smallest to largest.

```python
def is_sorted(numbers):
    for i in range(len(numbers) - 1):
        if numbers[i] > numbers[i + 1]:
            return False  # We found an out-of-order pair!
    return True
```

**(a) What is the Best-Case scenario, and what is its Big-O complexity?**
*Answer:* The list is completely unsorted right at the beginning (e.g., `[9, 1, 2, 3]`). The `if` statement triggers on the very first check, and the function immediately returns `False`. The loop runs 1 time. The Best-Case is **$O(1)$**.

**(b) What is the Worst-Case scenario, and what is its Big-O complexity?**
*Answer:* The list is perfectly sorted (e.g., `[1, 2, 3, 9]`). The function has to check every single pair to confirm no numbers are out of order. The loop runs $N-1$ times. Dropping the non-dominant $-1$, the Worst-Case is **$O(N)$**.

---

## Common Mistakes

- **Assuming "Average Case" is always easy to calculate:** For simple loops, the average case is just $N/2$. For complex algorithms (like sorting algorithms or binary search trees), calculating the true mathematical average case requires advanced statistics and probability. 
- **Selling the Best Case in Interviews:** Telling an interviewer your algorithm runs in $O(1)$ time because "we might get lucky and find the answer on the first try" shows a misunderstanding of engineering priorities. Always sell the worst case.

---

## Key Takeaway

The efficiency of an algorithm often depends on the arrangement of the input data, not just its size. We evaluate algorithms by looking at the Best Case (we get extremely lucky), the Average Case (the statistical norm), and the Worst Case (we get extremely unlucky). Because software must be reliable, the software industry and technical interviews almost exclusively focus on the **Worst-Case Time Complexity**.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Best Case** | The input data is arranged in the most optimal way possible, resulting in the minimum number of operations. |
| **Average Case** | The expected number of operations averaged over all possible inputs. |
| **Worst Case** | The input data is arranged in the most unhelpful way possible, resulting in the maximum number of operations. |
| **Linear Search** | An algorithm that searches for an item by checking each element in a list one by one from start to finish. |

---

## References
- [GeeksforGeeks: Worst, Average, and Best Case Analysis of Algorithms](https://www.geeksforgeeks.org/worst-average-and-best-case-analysis-of-algorithms/)
