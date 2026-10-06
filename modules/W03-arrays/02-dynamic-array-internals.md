# Dynamic Array Internals

In the last topic, we learned that true arrays have a strict, fixed size. If you ask for 5 slots of memory, you get exactly 5. 

So, how does a Python `list` magically grow whenever you call `.append()`? Is it breaking the rules of computer science? No. Behind the scenes, Python lists are an advanced data structure known as a **Dynamic Array**. In this section, we pull back the curtain to see how they actually manage memory.

---

## 1. How a Dynamic Array Works

A dynamic array starts its life as a normal, fixed-size array. 

When you create an empty Python list `my_list = []`, Python secretly goes to the computer's memory and reserves a small, fixed-size array (let's say, 4 slots).

If you append 4 items, the array is full. When you try to append a 5th item, Python steps in and does the following:
1. **Allocate:** It creates a brand new, empty array that is significantly larger (for example, double the size: 8 slots).
2. **Copy:** It copies your original 4 items from the old array into the new array.
3. **Delete:** It destroys the old, small array to free up memory.
4. **Append:** It finally inserts your 5th item into the newly created array.

---

## 2. Capacity vs. Length (Size)

Because of this resizing behavior, every dynamic array has two different properties:
- **Length (Size):** How many items are actually stored in the list right now. This is what you get when you call `len(my_list)`.
- **Capacity:** How many total slots Python has currently reserved in memory. 

You cannot easily see the capacity in standard Python, but it is always greater than or equal to the length. The extra empty slots are what allow `.append()` to run instantly most of the time.

---

## 3. The Time Complexity of `.append()`

As we briefly discussed in Week 2, what is the time complexity of appending to a dynamic array?

- **99% of the time (Capacity > Length):** The array has empty slots ready to go. The computer just drops the item into the next slot. This is a fast **$O(1)$ Constant Time** operation.
- **1% of the time (Capacity == Length):** The array is full. Python has to trigger the massive Allocate -> Copy -> Delete operation. Copying $N$ items to the new array takes **$O(N)$ Linear Time**.

Because the slow $O(N)$ operation happens so rarely, and it "pays" for a huge number of fast $O(1)$ operations, we average the cost out. We say that the **Amortized Time Complexity of `.append()` is $O(1)$**.

---

## Try it Yourself

**Scenario:** We can actually prove that Python lists reserve extra capacity by checking the physical memory size (in bytes) of a list as it grows.

Run this code snippet in a Python environment.

```python
import sys

my_list = []
print(f"Empty list size: {sys.getsizeof(my_list)} bytes")

for i in range(10):
    my_list.append(i)
    print(f"Length: {len(my_list)} | Memory Size: {sys.getsizeof(my_list)} bytes")
```

**What you will see:** 
You will notice that the `Memory Size` does not increase by a smooth, consistent amount on every single append. Instead, it stays the same for several appends (because there is extra capacity!), and then suddenly jumps by a large amount when Python is forced to resize the array!

---

## Common Mistakes

- **Thinking `.insert(0, item)` is as fast as `.append()`:** When you `.append()`, you just put an item at the very end of the array, which is amortized $O(1)$. If you `.insert(0, item)` at the *beginning* of the array, every single item currently in the array has to be physically shifted one slot to the right to make room. This takes $O(N)$ time and is terribly slow on large lists.
- **Worrying too much about Python's exact growth factor:** Different programming languages grow dynamic arrays at different rates. Java doubles the size ($2\times$), while Python grows by a factor of roughly $1.125\times$. In interviews, you just need to know the *concept* of resizing, not the exact mathematical growth factor.

---

## Key Takeaway

Python lists are not magical; they are **Dynamic Arrays**. Under the hood, they are standard, fixed-size arrays that automatically manage their own memory. When they run out of **capacity**, they create a larger array, copy the old data over, and continue. This resizing process is slow ($O(N)$), but it happens rarely enough that the overall **amortized cost** of appending remains $O(1)$.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Dynamic Array** | An array that automatically resizes itself when it runs out of capacity. |
| **Length (Size)** | The number of actual elements currently stored in the array. |
| **Capacity** | The total amount of memory currently reserved for the array, including empty slots. |
| **Resizing** | The internal process of allocating a larger array and copying elements over when the current array is full. |

---

## References
- [GeeksforGeeks: Dynamic Arrays](https://www.geeksforgeeks.org/how-do-dynamic-arrays-work/)
- [Python Documentation: Time Complexity of List Operations](https://wiki.python.org/moin/TimeComplexity)
