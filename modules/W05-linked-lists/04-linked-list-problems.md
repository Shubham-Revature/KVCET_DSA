# Classic Linked List Problems

Because Linked Lists do not have indices, you cannot use standard array algorithms on them. You cannot use `[::-1]` to reverse them, and you cannot use `len()` to find their size. 

To pass technical interviews, you must master a specific set of Linked List algorithms. The three most famous problems are Reversing a List, Detecting a Cycle, and Merging Two Sorted Lists.

---

## 1. Reversing a Linked List

How do you reverse a Singly Linked List in-place ($O(1)$ memory)? You have to manually flip every single arrow backward. 

To do this without losing the rest of your list, you need three temporary pointers: `prev`, `current`, and `next_node`.

1. **Save:** Save the next node so you don't lose the list.
2. **Flip:** Change `current.next` to point backwards to `prev`.
3. **Step Forward:** Move `prev` to `current`, and `current` to the saved `next_node`.

```python
def reverse_list(head):
    prev = None
    current = head
    
    while current is not None:
        next_node = current.next  # 1. Save the rest of the list
        current.next = prev       # 2. Flip the arrow backwards
        
        prev = current            # 3. Step forward
        current = next_node       # 3. Step forward
        
    return prev  # 'prev' is now the new Head!
```

---

## 2. Cycle Detection (Floyd's Tortoise and Hare)

Imagine a hacker corrupted your Linked List by pointing the Tail node back into the middle of the list, creating an infinite loop (a cycle). If you try to traverse it normally, your `while` loop will run forever and crash the server.

How do you detect a cycle using only $O(1)$ memory? We use the **Two-Pointer (Same Direction)** pattern we learned in Week 4!

We send a `slow` pointer (moving 1 step at a time) and a `fast` pointer (moving 2 steps at a time) down the list. If there is no cycle, `fast` will hit `None`. If there *is* a cycle, `fast` will loop around and eventually lap the `slow` pointer. If they ever collide, you know a cycle exists.

```python
def has_cycle(head):
    slow = head
    fast = head
    
    while fast is not None and fast.next is not None:
        slow = slow.next          # Move 1 step
        fast = fast.next.next     # Move 2 steps
        
        if slow == fast:
            return True  # Collision! There is a cycle.
            
    return False
```

---

## 3. Merging Two Sorted Lists

If you are given two Linked Lists that are already sorted, how do you merge them into a single sorted list?

Think of it like zipping up a jacket. You look at the head of List A and the head of List B. Whichever is smaller gets attached to your new list. You then move that pointer forward, and compare again.

*(We will cover this specific merging logic in-depth during the coding lab!)*

---

## Try it Yourself

**Scenario:** Let's practice the logic behind Floyd's Tortoise and Hare algorithm.

Imagine a circular track with 10 spaces (indices 0 to 9). 
- `slow` starts at 0 and moves +1.
- `fast` starts at 0 and moves +2.

*(a) Where are `slow` and `fast` after 1 step?*
- `slow` is at 1.
- `fast` is at 2.

*(b) Walk through the next few steps mentally. On what step do they collide (meaning their values are exactly the same)?*
- Step 2: `slow`=2, `fast`=4
- Step 3: `slow`=3, `fast`=6
- Step 4: `slow`=4, `fast`=8
- Step 5: `slow`=5, `fast`=10 (which wraps around to 0)
- Step 6: `slow`=6, `fast`=2
- Step 7: `slow`=7, `fast`=4
- Step 8: `slow`=8, `fast`=6
- Step 9: `slow`=9, `fast`=8
- Step 10: `slow`=10 (wraps to 0), `fast`=10 (wraps to 0). **Collision at 0!**

The math guarantees they will always collide if a loop exists!

---

## Common Mistakes

- **Forgetting to save `next_node` during reversal:** If you write `current.next = prev` before saving `current.next` in a temporary variable, the pointer to the rest of the list is destroyed. You have orphaned the rest of the list in memory.
- **Trying to use array logic:** Beginners often try to convert the Linked List into a Python Array first, solve the problem using array tricks, and then convert it back. While this works, it takes $O(N)$ Auxiliary Space. Interviewers will instantly reject this approach; they want to see you manipulate the pointers directly in $O(1)$ space.

---

## Key Takeaway

Because Linked Lists lack indices, you must rely heavily on **Pointers** to solve problems. Reversing a list requires carefully juggling three pointers (`prev`, `current`, `next_node`) to flip arrows without losing data. Detecting cycles relies on the brilliant **Tortoise and Hare** algorithm, which uses a slow and fast pointer to find infinite loops in $O(1)$ memory. 

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **In-Place Reversal** | Reversing the arrows of a Linked List without creating any new nodes ($O(1)$ space). |
| **Cycle** | An infinite loop in a Linked List caused by a node pointing back to a previous node. |
| **Floyd's Tortoise and Hare** | A classic algorithm that uses a slow and fast pointer to detect cycles. |

---

## References
- [GeeksforGeeks: Reverse a Linked List](https://www.geeksforgeeks.org/reverse-a-linked-list/)
- [GeeksforGeeks: Detect Loop in a Linked List](https://www.geeksforgeeks.org/detect-loop-in-a-linked-list/)
