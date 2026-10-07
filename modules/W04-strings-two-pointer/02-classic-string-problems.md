# Classic String Problems

String manipulation forms the backbone of countless technical interview questions. Before we dive into advanced algorithmic patterns like the Two-Pointer technique, you should be deeply familiar with the "Classic Three" string problems. 

Almost every junior engineer will be asked to solve one of these three problems during their career: Reversing a String, Checking Palindromes, and Checking Anagrams.

---

## 1. Reversing a String

Because strings are immutable, we cannot reverse a string "in-place" like we did with an array in Week 3. We are forced to create a brand new string.

**The Pythonic Way:**
Python has a built-in slicing trick that reverses an iterable instantly. This is extremely fast and is considered the standard way to reverse a string in Python.
```python
def reverse_string(s):
    return s[::-1]  # The -1 step tells Python to slice backwards
```

**The Manual Way:**
If an interviewer forbids you from using the `[::-1]` trick, you must build the string manually. Remember our rule from the previous lesson: use a list and `.join()`!
```python
def reverse_manual(s):
    chars = []
    # Loop backwards through the string
    for i in range(len(s) - 1, -1, -1):
        chars.append(s[i])
    return "".join(chars)
```

---

## 2. Palindromes

A **Palindrome** is a word, phrase, or sequence that reads the exact same backwards as it does forwards. (Examples: "racecar", "madam", "level").

Checking if a string is a palindrome is incredibly simple if we use the Python slicing trick from above.

```python
def is_palindrome(s):
    # Does the original string equal the reversed string?
    return s == s[::-1]
```
*Note: In the next lesson, we will learn how to check for a palindrome manually using the Two-Pointer technique, which is what interviewers actually want to see!*

---

## 3. Anagrams

An **Anagram** is a word formed by rearranging the exact same letters of a different word. (Example: "listen" and "silent").

To check if two strings are anagrams, we need to verify they have the exact same count of every letter. The easiest way to do this is to sort both strings alphabetically. If they are anagrams, their sorted versions will be identical.

```python
def is_anagram(s1, s2):
    # If lengths are different, they cannot be anagrams
    if len(s1) != len(s2):
        return False
        
    # sorted() converts a string to a sorted list of characters
    return sorted(s1) == sorted(s2)
```
**Time Complexity:** Because Python uses Timsort for `sorted()`, this algorithm takes **$O(N \log N)$** time. *(In Week 5, we will learn how to optimize this to $O(N)$ using Hash Maps!)*

---

## Try it Yourself

**Scenario:** An interviewer asks you to write a palindrome checker, but with a twist: the function must be case-insensitive.

The string `"Racecar"` should return `True`, even though the first letter is capital `"R"` and the last letter is lowercase `"r"`. 

*Your Spec:*
1. Convert the entire string to lowercase first using `.lower()`.
2. Check if the lowercase string equals its reverse.

```python
def is_palindrome_insensitive(s):
    # 1. Standardize the data
    s = s.lower()
    
    # 2. Check the condition
    return s == s[::-1]

assert is_palindrome_insensitive("Racecar") == True
assert is_palindrome_insensitive("Madam") == True
assert is_palindrome_insensitive("Hello") == False
print("Case-insensitive checks passed!")
```
```text
Case-insensitive checks passed!
```

---

## Common Mistakes

- **Forgetting about case sensitivity:** To a computer, `"A"` and `"a"` are completely different characters. (We will see why when we discuss ASCII values). If you are asked to compare strings, always ask the interviewer: *"Is this case-sensitive?"* If it isn't, immediately run `.lower()` or `.upper()` on your inputs.
- **Forgetting about spaces in Anagrams/Palindromes:** Is `"Taco cat"` a palindrome? Yes, but only if you remove the space! Before checking for palindromes or anagrams in a real interview, you often need to remove spaces using `s.replace(" ", "")`.

---

## Key Takeaway

The Classic Three string problems are String Reversal, Palindromes, and Anagrams. Python makes solving these trivial using slicing tricks (`[::-1]`) and built-in sorting (`sorted()`). However, you must always be cautious of hidden constraints like case-sensitivity and spaces, and remember that `sorted()` introduces an $O(N \log N)$ time complexity.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Palindrome** | A word or sequence that reads the same forwards and backwards. |
| **Anagram** | A word formed by rearranging the letters of another word. |
| **String Slicing** | A Python feature that extracts parts of a string. `[::-1]` specifically extracts the whole string in reverse step order. |
| **Case-Insensitive** | Treating uppercase and lowercase letters as equivalent. |

---

## References
- [GeeksforGeeks: Python String Slicing](https://www.geeksforgeeks.org/string-slicing-in-python/)
