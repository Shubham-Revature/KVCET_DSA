# Strings as a Data Structure

You use text all the time in everyday programming (e.g., `print("Hello World")`). But in algorithmic interviews and data structure courses, Strings are treated as formal data structures. 

In many ways, strings are just arrays of characters, meaning they share the same superpowers as arrays. However, in Python, they have one massive limitation that you must be aware of to avoid catastrophic performance issues.

---

## 1. Strings are Just Arrays

Under the hood, a string is a contiguous block of memory holding characters. Because of this, everything you learned about arrays in Week 3 applies to strings!

- **$O(1)$ Instant Indexing:** You can access the 100th character of a string instantly: `letter = my_string[99]`.
- **Iteration:** You can loop through a string exactly like a list: `for char in my_string:`.
- **Slicing:** You can easily grab substrings: `my_string[0:5]`.

---

## 2. The Big Catch: Immutability

The defining difference between a Python List and a Python String is **Immutability**. 

A Python list is *mutable* (changeable). You can do `my_list[0] = "X"`, and Python changes that slot in memory.
A Python string is *immutable* (unchangeable). Once a string is created in memory, **it can never be altered**. 

If you try to do this:
```python
name = "Hello"
name[0] = "J"
```
Python will immediately crash with a `TypeError: 'str' object does not support item assignment`.

---

## 3. The Hidden Cost of String Concatenation

If strings are unchangeable, how do we add text to them? 

When you write `name += " World"`, Python doesn't actually alter the original "Hello" string. Instead, it does the following:
1. It requests a brand new, larger block of memory.
2. It copies "H", "e", "l", "l", "o" into the new block.
3. It copies " ", "W", "o", "r", "l", "d" into the new block.
4. It destroys the original "Hello" string.

Every time you add a single character to a string, Python has to copy the entire string over again. This copying takes $O(N)$ time.

### The $O(N^2)$ String Builder Disaster
If you try to build a long string inside a loop using `+=`, it triggers that $O(N)$ copy process *every single time the loop runs*. 

```python
# VERY BAD: O(N^2) Time Complexity
def build_string_bad(n):
    result = ""
    for i in range(n):
        result += "a"  # Copies the entire string every loop!
    return result
```
If $N$ is 100,000, this simple loop will freeze your program and trigger a Time Limit Exceeded error.

---

## 4. The Solution: Lists and `.join()`

To build or modify strings efficiently, we borrow the power of Dynamic Arrays (Lists). 

Because lists are mutable, we can `.append()` to them in $O(1)$ amortized time. When we are completely finished building our text, we merge the list together into a final string using Python's highly optimized `"".join()` method.

```python
# EXCELLENT: O(N) Time Complexity
def build_string_good(n):
    result_list = []
    for i in range(n):
        result_list.append("a")  # Fast O(1) amortized append
        
    # Merge the list into a string exactly once
    return "".join(result_list)
```

---

## Try it Yourself

**Scenario:** You are writing a function to remove all vowels from a string. 

Because strings are immutable, you cannot delete characters from the original string. Use the `.join()` technique to build the new string efficiently.

*Your Spec:*
1. Create an empty list called `chars`.
2. Loop through the input string.
3. If the character is not in `"aeiou"`, append it to `chars`.
4. Return the joined list.

```python
def remove_vowels(text):
    chars = []
    for char in text:
        if char.lower() not in "aeiou":
            chars.append(char)
    
    return "".join(chars)

assert remove_vowels("algorithmic") == "lgrthmc"
print("Vowels successfully removed!")
```
```text
Vowels successfully removed!
```

---

## Common Mistakes

- **Trying to change a string in place:** As mentioned, `text[0] = "A"` will crash your program. If you need to swap characters (like reversing a string), you must convert the string to a list first: `char_list = list(text)`, do your swaps, and then `"".join(char_list)`.
- **Using `+=` in a loop:** Never use string concatenation (`+` or `+=`) inside a `for` or `while` loop if the loop runs many times. It is a guaranteed way to fail performance tests in interviews.

---

## Key Takeaway

Strings are essentially arrays of characters, meaning you can index and loop through them in $O(1)$ time. However, unlike arrays, strings are **immutable** (cannot be changed). Modifying a string forces Python to create a brand new copy of it, which takes $O(N)$ time. To build or modify strings efficiently, always accumulate the characters in a Python List first, and use `"".join()` at the very end.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Immutable** | An object whose state cannot be modified after it is created. |
| **Mutable** | An object whose state can be modified after it is created (like a list). |
| **String Concatenation** | The operation of joining two strings end-to-end (e.g., `"a" + "b"`). |
| **`"".join()`** | A highly optimized string method in Python that merges an iterable (like a list) into a single string. |

---

## References
- [GeeksforGeeks: Mutable vs Immutable Objects in Python](https://www.geeksforgeeks.org/mutable-vs-immutable-objects-in-python/)
