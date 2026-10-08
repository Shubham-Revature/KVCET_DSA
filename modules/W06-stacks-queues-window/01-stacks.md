# Stacks

In previous weeks, we learned how data is physically stored in memory (Arrays vs. Linked Lists). A **Stack** is slightly different; it is an *Abstract Data Type*. This means it isn't a new way of organizing physical memory, but rather a strict set of rules about how you are allowed to add and remove data. 

---

## 1. The LIFO Principle

The defining rule of a Stack is **LIFO: Last-In, First-Out**. 

Imagine a physical stack of plates at a buffet. When a waiter brings a clean plate, they put it on the very top of the stack. When you go to grab a plate, you take one off the very top. The *last* plate the waiter put down is the *first* plate you pick up. 

In a Stack, you are completely forbidden from pulling data out of the middle or the bottom. You can only interact with the "Top".

**Real-World Software Examples:**
- **The "Undo" Button (Ctrl+Z):** Every time you type a word, the computer puts that action on top of a Stack. When you press Undo, it pulls the most recent action off the top and reverses it.
- **Browser History:** When you click a link, your current page goes onto a Stack. When you click the "Back" arrow, the browser pulls the last page off the top of the Stack.
- **The Call Stack:** As we learned in Week 2, Python uses a Stack to keep track of recursive function calls!

---

## 2. Core Operations

A formal Stack only has three allowed operations. Because we only interact with the top, all three operations run in blazing fast **$O(1)$ Constant Time**.

1. **Push:** Add a new item to the top of the Stack.
2. **Pop:** Remove and return the item currently at the top of the Stack.
3. **Peek (or Top):** Look at the item currently at the top without removing it.

---

## 3. Implementing a Stack in Python

In some languages (like Java), there is a dedicated `Stack` object you must import. In Python, we do not need to import anything. We use a standard Python `list` to act as our Stack.

Why? Because the right-side of a Python list acts exactly like the top of a Stack!

- `list.append(x)` adds an item to the end of the array in $O(1)$ time. This is our **Push**.
- `list.pop()` removes an item from the end of the array in $O(1)$ time. This is our **Pop**.
- `list[-1]` looks at the last item without removing it. This is our **Peek**.

```python
stack = []

# Push items onto the stack
stack.append("A")
stack.append("B")
stack.append("C")
# Stack is now: ["A", "B", "C"] (Top is "C")

# Peek at the top item
print("Top item is:", stack[-1])  # Prints "C"

# Pop the top item
removed_item = stack.pop()
print("Removed:", removed_item)   # Prints "C"
# Stack is now: ["A", "B"]
```

---

## Try it Yourself

**Scenario:** Let's build a mini-simulator for a Web Browser's "Back" button using a Stack.

*Your Spec:*
1. Create an empty list called `history`.
2. A user visits "google.com", then "youtube.com", then "github.com". Push these strings onto the stack.
3. The user clicks the "Back" button once. Pop the current page off the stack and store it in a variable.
4. Peek at the stack to see what page the user is currently looking at now.

```python
history = []

# 2. User navigates
history.append("google.com")
history.append("youtube.com")
history.append("github.com")

# 3. User clicks Back
closed_page = history.pop()

# 4. Where are we now?
current_page = history[-1]

assert closed_page == "github.com"
assert current_page == "youtube.com"
print("Browser history successfully navigated!")
```
```text
Browser history successfully navigated!
```

---

## Common Mistakes

- **Popping from an empty Stack:** If your Stack has 0 items in it, and you run `stack.pop()` or `stack[-1]`, Python will crash with an `IndexError`. In interviews, you must always verify the stack isn't empty first by writing `if len(stack) > 0:` or simply `if stack:`.
- **Using `.pop(0)`:** Beginners sometimes get confused and think index 0 is the "top" of the Stack. If you use `.pop(0)`, it takes $O(N)$ time because the entire array has to shift left. Always use `.append()` and `.pop()` without arguments to ensure $O(1)$ performance.

---

## Key Takeaway

A **Stack** is an abstract data type governed by the **LIFO (Last-In, First-Out)** principle. You can only interact with the "Top" of the stack. Because all actions happen at one end, the core operations (**Push**, **Pop**, and **Peek**) all execute in $O(1)$ time. In Python, we implement Stacks simply by using a standard list and limiting ourselves to the `.append()` and `.pop()` methods.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Stack** | An abstract data type that serves as a collection of elements, with two principal operations: Push and Pop. |
| **LIFO** | Last-In, First-Out. The operational rule of a Stack. |
| **Push** | Adding an element to the top of a Stack. |
| **Pop** | Removing the top element from a Stack. |
| **Peek / Top** | Reading the value of the top element without removing it. |

---

## References
- [GeeksforGeeks: Stack Data Structure](https://www.geeksforgeeks.org/stack-data-structure/)

