# Queues

If a Stack is a stack of plates at a buffet, a **Queue** is a line of people waiting for a rollercoaster. 

Like the Stack, a Queue is an Abstract Data Type. It does not dictate how memory is physically stored; it only provides a strict set of rules about how data is allowed to enter and leave the collection. While a Stack enforces recency, a Queue enforces *fairness*.

---

## 1. The FIFO Principle

The defining rule of a Queue is **FIFO: First-In, First-Out**. 

If you are the first person to join the line at a grocery store, you expect to be the first person to check out. The first piece of data inserted into a Queue is guaranteed to be the first piece of data removed.

**Real-World Software Examples:**
- **Printer Queues:** If three people send documents to an office printer, the printer prints them in the exact order they were received.
- **Customer Support Tickets:** When you submit a help ticket, it goes into a Queue. Support agents answer tickets from the front of the Queue (the oldest ones) to ensure nobody waits forever.
- **Breadth-First Search (BFS):** Later in the course, we will use Queues to search through Trees and Graphs layer by layer!

---

## 2. Core Operations

A formal Queue has three allowed operations. Just like a Stack, all of these operations must execute in **$O(1)$ Constant Time**.

1. **Enqueue:** Add a new item to the *back* (end) of the Queue.
2. **Dequeue:** Remove and return the item currently at the *front* of the Queue.
3. **Front / Peek:** Look at the item currently at the front without removing it.

---

## 3. Implementing a Queue in Python

In the first lesson of this week, we learned that we can use a standard Python list to act as a Stack because `.append()` and `.pop()` happen at the end of the array in $O(1)$ time. 

**Can we use a standard Python list for a Queue? NO!**

If we `enqueue` using `list.append()`, it is $O(1)$. But to `dequeue`, we would have to remove the person at the front of the line using `list.pop(0)`. As we learned in Week 3, removing the first item of an array forces Python to shift every single remaining item to the left. This takes **$O(N)$ Linear Time**. 

Using a standard list for a Queue in an interview is an instant red flag.

### The Solution: `collections.deque`

Leveraging the Doubly Linked List concepts we learned in Week 5, Python provides a highly optimized built-in data structure called a **Deque** (Double-Ended Queue). Because it uses pointers instead of contiguous memory, it can remove items from the front instantly without shifting!

```python
from collections import deque

queue = deque()

# Enqueue (Add to back)
queue.append("Alice")
queue.append("Bob")
queue.append("Charlie")
# Queue is now: ["Alice", "Bob", "Charlie"]

# Dequeue (Remove from front)
served = queue.popleft() 
print("Served:", served)  # Prints "Alice"
# Queue is now: ["Bob", "Charlie"]
```

---

## Try it Yourself

**Scenario:** Let's build a simple matchmaking system for a multiplayer game. Players join the queue. When 2 players are in the queue, the system pops them both and starts a match.

*Your Spec:*
1. Import `deque`.
2. Create an empty `matchmaking_queue`.
3. "Player 1" joins.
4. "Player 2" joins.
5. Write an `if` statement checking if the queue length is $\ge$ 2.
6. If so, `dequeue` both players and print a message starting their match.

```python
from collections import deque

matchmaking_queue = deque()

# Players join
matchmaking_queue.append("Player 1")
matchmaking_queue.append("Player 2")

# Matchmaking logic
if len(matchmaking_queue) >= 2:
    p1 = matchmaking_queue.popleft() # O(1) Dequeue
    p2 = matchmaking_queue.popleft() # O(1) Dequeue
    print(f"Starting match: {p1} vs {p2}!")
    
assert len(matchmaking_queue) == 0
```
```text
Starting match: Player 1 vs Player 2!
```

---

## Common Mistakes

- **Using a standard list:** As heavily emphasized above, do not write `my_queue = []` and use `my_queue.pop(0)`. You must memorize the import statement `from collections import deque`. 
- **Confusing `pop()` and `popleft()`:** In a `deque`, `.pop()` removes from the *back* (LIFO behavior). `.popleft()` removes from the *front* (FIFO behavior). If you are building a Queue, you must use `.popleft()`.

---

## Key Takeaway

A **Queue** is an abstract data type governed by the **FIFO (First-In, First-Out)** principle. It is used to enforce fairness, ensuring the oldest data is processed first. Its core operations (**Enqueue** and **Dequeue**) must run in $O(1)$ time. Because standard Python lists take $O(N)$ time to remove from the front, you must always implement a Queue using Python's `collections.deque` and the `.popleft()` method.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Queue** | An abstract data type that serves as a collection of elements, with two principal operations: Enqueue and Dequeue. |
| **FIFO** | First-In, First-Out. The operational rule of a Queue. |
| **Enqueue** | Adding an element to the back of a Queue. |
| **Dequeue** | Removing the front element from a Queue. |
| **`popleft()`** | The specific $O(1)$ method used on a Python `deque` to perform a Dequeue operation. |

---

## References
- [Python Documentation: collections.deque](https://docs.python.org/3/library/collections.html#collections.deque)
