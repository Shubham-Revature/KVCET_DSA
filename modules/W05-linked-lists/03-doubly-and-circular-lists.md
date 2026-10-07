# Doubly and Circular Linked Lists

The Singly Linked List we built in the last section is simple and memory-efficient. However, it has one massive flaw: it is a one-way street. If you are traversing the list and you accidentally go one node too far, you cannot go backwards. You have to start all the over from the Head!

To solve this problem, and to handle other specific engineering scenarios, computer scientists created two major variations of the Linked List.

---

## 1. The Doubly Linked List

To allow two-way travel, we must upgrade our basic Node class. In a **Doubly Linked List**, every single Node holds *two* pointers: one pointing forward to the next node, and one pointing backward to the previous node.

### The Upgraded Node Class
```python
class DoublyNode:
    def __init__(self, value):
        self.val = value
        self.next = None    # Points forward
        self.prev = None    # Points backward
```

### The Trade-Off
Why don't we just use Doubly Linked Lists for everything? Because of the **Time-Space Tradeoff**. 
- **The Pro:** You can traverse the list in both directions. Deleting a node is also much easier if you only have a pointer to the node you want to delete (because you can look backward to find the previous node).
- **The Con:** Every single node now requires extra memory to store that `prev` pointer. Furthermore, writing the code to insert or delete a node becomes twice as complicated because you have to carefully update arrows going in both directions without losing data!

*Real-World Example:* A web browser's history feature. You can click "Forward" (`next`) and "Back" (`prev`) endlessly.

---

## 2. The Circular Linked List

In a standard Linked List, you know you have reached the end when the Tail node points to `None`. 

In a **Circular Linked List**, the Tail does not point to `None`. Instead, the Tail's `next` pointer loops all the way back around and points to the Head! This creates an infinite, continuous loop. 

Circular lists can be Singly Linked (forward only) or Doubly Linked (forwards and backwards).

### How to Traverse a Circular List
If the loop is infinite, how does your `while` loop know when to stop? Instead of looking for `None`, you must record the `head` node, and stop traversing when your `current` pointer equals the `head` again.

*Real-World Example:* Multiplayer turn-based games (like Monopoly). When Player 4 finishes their turn, the `next` pointer loops back around to Player 1.

---

## Try it Yourself

**Scenario:** You want to manually build a small Doubly Linked List with two nodes: `node_a` ("Alice") and `node_b` ("Bob"). 

*Your Spec:*
1. Create both nodes using the `DoublyNode` class.
2. Link Alice's `next` to point to Bob.
3. Link Bob's `prev` to point to Alice.
4. Verify that you can travel from Bob back to Alice.

```python
# 1. Create the nodes
node_a = DoublyNode("Alice")
node_b = DoublyNode("Bob")

# 2 & 3. Link them in both directions
node_a.next = node_b
node_b.prev = node_a

# 4. Travel backwards!
assert node_b.prev.val == "Alice"
print("Reverse travel successful!")
```
```text
Reverse travel successful!
```

---

## Common Mistakes

- **Breaking the backward chain:** When inserting a new node into a Doubly Linked List, beginners often remember to update the `next` pointers but forget to update the `prev` pointers. This results in a broken list: you can travel forward perfectly, but if you try to travel backward, you skip nodes or crash! Always draw the arrows on a whiteboard to ensure you updated both directions.
- **Infinite loops in Circular Lists:** If you forget to write the logic that stops the traversal when `current == head`, your program will get stuck in an infinite loop and crash your computer.

---

## Key Takeaway

While Singly Linked Lists are a one-way street, **Doubly Linked Lists** add a `prev` pointer to allow two-way traversal, at the cost of extra memory and more complex insert/delete logic. **Circular Linked Lists** connect the Tail back to the Head, creating an infinite loop perfect for round-robin or turn-based systems.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Doubly Linked List** | A Linked List where every node has both a `next` and `prev` pointer, allowing traversal in both directions. |
| **Circular Linked List** | A Linked List where the last node points back to the first node, creating a loop. |
| **`prev` Pointer** | The variable in a DoublyNode that stores the memory address of the previous node. |

---

## References
- [GeeksforGeeks: Doubly Linked List](https://www.geeksforgeeks.org/doubly-linked-list/)
- [GeeksforGeeks: Circular Linked List](https://www.geeksforgeeks.org/circular-linked-list/)
