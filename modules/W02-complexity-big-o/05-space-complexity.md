# Space Complexity

So far, every time we have discussed Big-O, we have been talking about **Time Complexity**: how many operations an algorithm takes as the input gets larger. But time is only half the battle. 

Algorithms also consume memory (RAM) to store variables, lists, and objects. **Space Complexity** uses the exact same Big-O notation to describe how the amount of memory required by an algorithm scales as the input size ($N$) grows.

---

## 1. Space Complexity vs. Auxiliary Space

There is a subtle but critical difference you must understand for interviews:
- **Space Complexity:** The total amount of memory used by the algorithm, *including the input data itself*.
- **Auxiliary Space:** The *extra, temporary* memory used by the algorithm to do its job, *excluding* the input data.

If an interviewer asks you for the "Space Complexity", 99% of the time they actually mean the **Auxiliary Space**. If you receive a list of 1 million items and you sort it without creating any new lists, the Auxiliary Space is $O(1)$. You shouldn't say $O(N)$ just because the input list is large. You didn't create the input list; it was given to you.

---

## 2. In-Place Algorithms: $O(1)$ Space

An algorithm is considered **In-Place** if it requires a constant amount of extra memory, regardless of how large $N$ is. This usually means you are only creating a few integer or boolean variables to help you process the data.

```python
def sum_list(items):
    total = 0               # We created 1 integer variable
    for num in items:
        total += num
    return total
```
Even if `items` contains 1 billion numbers, we only ever created one extra variable (`total`). The auxiliary space complexity is **$O(1)$**.

---

## 3. Out-of-Place Algorithms: $O(N)$ Space

An algorithm uses $O(N)$ auxiliary space if it creates new data structures (like a new list, a dictionary, or a set) that grow proportionately to the input size $N$. 

```python
def get_even_numbers(items):
    evens = []              # We created a new list!
    for num in items:
        if num % 2 == 0:
            evens.append(num)
    return evens
```
In the worst case (where every number is even), the new `evens` list will be the exact same size as the input list. Because the memory we requested grows alongside $N$, the auxiliary space complexity is **$O(N)$**.

---

## 4. The Time-Space Tradeoff

In software engineering, you rarely get something for nothing. There is a universal concept known as the **Time-Space Tradeoff**. 

Often, if you want an algorithm to run blazingly fast (optimize Time Complexity), you have to use a lot of extra memory (sacrifice Space Complexity) to store helpful data. Conversely, if you want an algorithm to use almost no memory (optimize Space Complexity), it might take much longer to calculate the answer (sacrifice Time Complexity).

When designing systems, engineers must decide which resource is more scarce for their specific problem: CPU time or RAM.

---

## Try it Yourself

**Scenario:** You need to write a function that takes a list of numbers and doubles every number in it. Look at the two approaches below.

**Approach A:**
```python
def double_a(items):
    for i in range(len(items)):
        items[i] = items[i] * 2
    return items
```

**Approach B:**
```python
def double_b(items):
    new_list = []
    for i in range(len(items)):
        new_list.append(items[i] * 2)
    return new_list
```

**(a) What is the Auxiliary Space Complexity of Approach A?**
*Answer:* $O(1)$. It modifies the original list directly (in-place). It does not create any new lists that scale with $N$. 

**(b) What is the Auxiliary Space Complexity of Approach B?**
*Answer:* $O(N)$. It creates an entirely new list that is exactly the same size as the input list. 

---

## Common Mistakes

- **Including the input in your answer:** As mentioned above, if an interviewer asks "What is the space complexity?", always assume they mean Auxiliary Space. State clearly, "The auxiliary space is $O(1)$ because we only use a few pointers and modify the input array in-place."
- **Forgetting that strings are immutable:** In Python, you cannot change a string in-place. If you modify a string, Python secretly creates a brand new string behind the scenes. Therefore, string manipulation algorithms almost always use at least $O(N)$ space in Python.

---

## Key Takeaway

Algorithms consume both time (CPU) and space (RAM). Space complexity measures how the *extra* memory required by your code scales with the input size. In-place algorithms ($O(1)$ space) are highly memory-efficient, while algorithms that duplicate data into new lists ($O(N)$ space) consume more RAM. Good engineers constantly balance the Time-Space Tradeoff to fit the needs of their application.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Space Complexity** | How the total memory used by an algorithm scales with input size (Technically includes input data). |
| **Auxiliary Space** | The *extra, temporary* memory requested by the algorithm, excluding the input data. |
| **In-Place Algorithm** | An algorithm that modifies data directly without creating copies, usually taking $O(1)$ auxiliary space. |
| **Time-Space Tradeoff** | The concept that improving speed often requires consuming more memory, and saving memory often requires sacrificing speed. |

---

## References
- [GeeksforGeeks: Time and Space Complexity](https://www.geeksforgeeks.org/time-and-space-complexity-in-data-structure/)
