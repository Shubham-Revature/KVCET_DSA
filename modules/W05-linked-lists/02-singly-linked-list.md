# The Singly Linked List

In the previous topic, we established the theory: a Linked List is a series of scattered **Nodes** connected by **Pointers**. 

Now, it is time to translate that theory into Python. Unlike arrays, which are built into Python as `list` objects, Linked Lists are not built-in. If you want a Linked List, you have to build the architecture yourself using Object-Oriented Programming (Classes). 

---

## 1. Building the Node Class

The foundation of any Linked List is the Node. A Node only has two jobs: it holds a piece of data, and it holds a pointer to the next Node.

```python
class Node:
    def __init__(self, value):
        self.val = value    # The actual data
        self.next = None    # The pointer to the next Node
```
When we create a new Node (`Node(5)`), the pointer `self.next` defaults to `None`. This simply means "I am currently the end of the line; I don't point to anything yet."

---

## 2. Linking Nodes Manually

Let's create three separate Nodes and scatter them in memory.

```python
# 1. Create three scattered Nodes
node_a = Node("Alice")
node_b = Node("Bob")
node_c = Node("Charlie")

# 2. Link them together like a chain
node_a.next = node_b
node_b.next = node_c
```

Congratulations! You just built a **Singly Linked List**. It is called "Singly" because the arrows only point in one direction (forwards). You can go from Alice to Bob, but you cannot go backwards from Bob to Alice. 

In this list, `node_a` is the **Head** (the start), and `node_c` is the **Tail** (the end, because `node_c.next` is still `None`).

---

## 3. Traversing the List

Because Linked Lists do not have indices, you cannot write `print(my_list[1])` to find Bob. 

If you want to read the data in a Linked List, you must perform a **Traversal**. You start at the Head, and you follow the pointers one by one until you reach `None`.

```python
def print_linked_list(head):
    # Create a temporary pointer to keep track of where we are
    current = head
    
    # Keep going until we hit the end (None)
    while current is not None:
        print(current.val)          # Print the data
        current = current.next      # Move the pointer to the next Node!
```
**Time Complexity:** Because we have to visit every single Node exactly once, traversal takes **$O(N)$ Linear Time**.

---

## Try it Yourself

**Scenario:** You need to write a function that finds the *length* of a Linked List. Because we built this list manually, Python's `len()` function will not work on it! 

*Your Spec:*
1. Create a `count` variable starting at 0.
2. Create a `current` pointer starting at the `head`.
3. Loop through the list (while `current` is not `None`).
4. Increment the `count` by 1.
5. Move `current` to the next Node.
6. Return the total count.

```python
def get_length(head):
    count = 0
    current = head
    
    while current is not None:
        count += 1
        current = current.next
        
    return count

# Reusing our Alice -> Bob -> Charlie list from above
assert get_length(node_a) == 3
print("Length calculation successful!")
```
```text
Length calculation successful!
```

---

## Common Mistakes

- **Losing the Head:** The absolute most common mistake beginners make is writing `head = head.next` instead of creating a `current` variable. If you move the actual `head` variable, you have permanently lost the beginning of your list. Python's Garbage Collector will see that no variables are pointing to Alice anymore, and it will permanently delete her from memory! *Always create a temporary pointer (like `current`) when traversing.*
- **Checking `current.next is not None`:** Sometimes beginners write `while current.next is not None:`. If you do this, the loop will stop *before* it processes the final Node (the Tail), causing you to miss the last piece of data. Always use `while current is not None:`.

---

## Key Takeaway

A **Singly Linked List** is built using a custom `Node` class that holds a value and a `next` pointer. To read data, you cannot use an index; you must **traverse** the list in $O(N)$ time by starting at the Head and repeatedly updating a temporary `current` pointer to `current.next` until you hit `None`.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Singly Linked List** | A Linked List where every node only has a pointer to the *next* node (one-way traffic). |
| **Traversal** | The process of visiting every single node in a data structure, usually done with a `while` loop in a Linked List. |
| **Tail** | The final Node in a Linked List. You know it is the Tail because its `next` pointer is `None`. |
| **Garbage Collection** | Python's automatic system for deleting data from memory once no variables are pointing to it anymore. |

---

## References
- [GeeksforGeeks: Linked List Data Structure](https://www.geeksforgeeks.org/data-structures/linked-list/)
