# Deques and Priority Queues

We know that a standard Queue enforces strict FIFO (First-In, First-Out) fairness. However, the real world is rarely that simple. Sometimes people are allowed to cut in line, or VIPs are allowed to skip to the very front. 

To handle these complex scenarios, we rely on two advanced queue structures: the Double-Ended Queue (Deque) and the Priority Queue.

---

## 1. The Deque (Double-Ended Queue)

We have already used Python's `collections.deque` module to build a standard Queue. But as the name suggests, it is actually a **Double-Ended Queue**. 

A true Deque does not force you to only enter at the back and leave at the front. It allows you to **Enqueue and Dequeue from BOTH ends in instant $O(1)$ time**.

**Use Cases for a Deque:**
- **Stealing Jobs:** In advanced operating systems, processors maintain a queue of tasks. If Processor A finishes all its tasks, it can go to Processor B's queue and steal a task from the *back* of the line (while Processor B continues taking from the front).
- **Palindrome Checking:** You can check if a word is a palindrome by putting it in a Deque, and simultaneously popping a letter from the left and a letter from the right. If they ever mismatch, it isn't a palindrome!

---

## 2. The Priority Queue

Imagine a hospital emergency room. Patients sit in a queue waiting to see a doctor. 
- Patient A arrives at 1:00 PM with a paper cut.
- Patient B arrives at 1:05 PM with a heart attack.

If the hospital used a strict FIFO Queue, Patient A would be seen first, which would be disastrous. Instead, the hospital uses a **Priority Queue**. 

In a Priority Queue, FIFO is completely ignored. Instead, every item is given a "priority score". **The item with the highest priority is always dequeued first, regardless of when it arrived in the line.**

### Python's `heapq`
Under the hood, Priority Queues are built using a complex Tree data structure called a **Heap** (which we will cover in depth during Week 8). 

In Python, we do not use lists or `collections.deque` for a Priority Queue. We use the built-in `heapq` module.
Because it uses a Tree structure, pushing and popping from a Priority Queue takes **$O(\log N)$ Logarithmic Time** (much faster than $O(N)$, but slightly slower than the $O(1)$ of a standard Queue).

---

## Try it Yourself

**Scenario:** Let's write the code for our Hospital Emergency Room using Python's `heapq`.

*Note: Python's `heapq` is a "Min-Heap", meaning the LOWEST number is considered the HIGHEST priority.*
- Priority 1: Heart Attack
- Priority 2: Broken Leg
- Priority 3: Paper Cut

```python
import heapq

# Create an empty priority queue (which is just a standard list that heapq manages)
er_queue = []

# Patients arrive in random order. 
# We push tuples into the queue: (Priority_Score, Patient_Name)
heapq.heappush(er_queue, (3, "Alice (Paper Cut)"))
heapq.heappush(er_queue, (1, "Bob (Heart Attack)"))
heapq.heappush(er_queue, (2, "Charlie (Broken Leg)"))

# The doctor is ready for the first patient. Who gets popped?
priority, patient = heapq.heappop(er_queue)

assert patient == "Bob (Heart Attack)"
print(f"Doctor is seeing: {patient}")
```
```text
Doctor is seeing: Bob (Heart Attack)
```
Even though Bob arrived second, he was dequeued first because his priority score (`1`) was the lowest number!

---

## Common Mistakes

- **Sorting a list to simulate a Priority Queue:** Beginners often try to create a priority queue by appending items to a standard list and then running `.sort()`. Because sorting takes $O(N \log N)$ time, doing this for every new item will cause your program to freeze on large datasets. You must use `heapq`, which inserts items efficiently in $O(\log N)$ time.
- **Forgetting that `heapq` is a Min-Heap:** By default, Python pops the *smallest* number first. If you want the *largest* number to pop first (a Max-Heap), you have to multiply all your priority scores by `-1` before pushing them into the queue.

---

## Key Takeaway

When strict FIFO rules are too limiting, we use advanced queues. A **Deque** (`collections.deque`) allows $O(1)$ insertions and deletions from both ends of the line. A **Priority Queue** (`heapq`) completely abandons FIFO, ensuring that the item with the highest priority is always served first. `heapq` operations run in $O(\log N)$ time.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Deque** | A Double-Ended Queue allowing $O(1)$ pushes and pops from both the front and the back. |
| **Priority Queue** | A queue where elements are served based on their assigned priority score rather than their arrival time. |
| **Heap** | The underlying tree data structure used to build an efficient Priority Queue. |
| **`heapq`** | Python's built-in module for implementing Priority Queues (specifically Min-Heaps). |

---

## References
- [Python Documentation: heapq module](https://docs.python.org/3/library/heapq.html)
