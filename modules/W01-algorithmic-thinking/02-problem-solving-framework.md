# The Problem-Solving Framework

---

## Learning Objectives

By the end of this topic, you will be able to:

- Apply a structured 4-step framework to solve any coding problem.
- Deconstruct a problem description into clear inputs, outputs, and constraints.
- Explain the importance of testing your solution with edge cases before submitting.
- Avoid the most common trap for beginners: coding before planning.

---

## Overview

When solving a new programming problem, don’t start coding right away. First, understand the problem and plan the logic. This helps you avoid getting stuck, writing messy code, and failing tests. Once the logic is clear, coding becomes much easier—you simply turn your solution into Python code.

In this topic, we will look at a reliable, repeatable 4-step framework you can use to solve any algorithmic problem.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Problem-Solving Framework** | A systematic approach to solving problems: Understand, Plan, Execute, and Test. |
| **Happy Path** | The most common, straightforward scenario where everything goes right. |
| **Edge Case** | Extreme or unusual inputs (like an empty list, or a negative number) that often break code. |
| **Desk Checking / Dry Run** | Tracing through your code or spec manually with a pen and paper. |

---

## The 4-Step Framework

This framework is adapted from George Pólya's famous book *How to Solve It* and tailored for software engineering.

### 1. Understand the Problem

You cannot solve a problem you don't understand. Read the problem statement carefully and ask yourself:
*   **What are the inputs?** (Data types, constraints, ranges)
*   **What are the expected outputs?** (Data types, format)
*   **What are the edge cases?** (What if the input is `0`? What if the list is empty? What if all numbers are negative?)

**Daily-life example:** If someone asks you to "buy milk," you need to understand the constraints. What kind of milk? What if the store is closed? How much money do I have?

### 2. Plan (The Spec-First Discipline)

Before writing any Python, write the logic in plain English (or pseudocode). 
*   **Start with Brute-Force:** Don't worry about writing the fastest, most clever algorithm immediately. Find the most obvious way to get the correct answer. 
*   **Break it down:** Write out the step-by-step instructions. 

### 3. Execute (Write Code)

Translate your plain English plan into Python code. Because you have already figured out the logic, this step should just be about syntax: knowing how to write a `for` loop, an `if` statement, or initializing a variable.

### 4. Test (Test-Your-Solution Discipline)

Never assume your code works just because it didn't throw a syntax error. 
*   Test it against the **happy path** (normal inputs).
*   Test it against the **edge cases** you identified in step 1.

---

## Applying the Framework

Let's apply this to a real problem.
**Problem:** *Write a function that takes a list of numbers and returns the sum of all the strictly positive numbers.*

### Step 1: Understand
- **Input:** A list of numbers (can include positive, negative, and zero).
- **Output:** A single number (the sum of the positive ones).
- **Edge cases:** What if the list is empty? (Sum should be 0). What if all numbers are negative? (Sum should be 0).

### Step 2: Plan
- Create a variable to keep a running total, starting at 0.
- Go through every number in the list one by one.
- Check if the current number is greater than 0.
- If it is, add it to the running total.
- After checking all numbers, return the total.

### Step 3: Execute

```python
def sum_of_positives(numbers):
    total = 0               # Running total starting at 0
    for num in numbers:     # Go through every number
        if num > 0:         # Check if strictly positive
            total += num    # Add to total
    return total            # Return the total
```

### Step 4: Test

```python
# Happy Path Test
print(sum_of_positives([1, -4, 7, 12]))  
# Output should be 1 + 7 + 12 = 20

# Edge Case Test 1 (All negatives)
print(sum_of_positives([-1, -2, -3]))    
# Output should be 0

# Edge Case Test 2 (Empty list)
print(sum_of_positives([]))              
# Output should be 0
```
```text
20
0
0
```

---

## Try it Yourself

**Scenario:** You need to write a function that counts how many times the letter "a" (case-insensitive) appears in a given word.

**(a) Apply Step 1 and Step 2:** Write down the Inputs, Outputs, Edge Cases, and your Plain English Spec.

*Your Spec:*
- Input: A single string (the word).
- Output: An integer (the count).
- Edge Cases: Empty string, a string with no "a"s, a string with capital "A"s.
- Plan: Start a counter at 0. Go through every letter in the word. Convert the letter to lowercase. If it is "a", increase the counter by 1. Return the counter at the end.

**(b) Apply Step 3:** Write the Python code.

```python
def count_a(word):
    count = 0
    for letter in word:
        if letter.lower() == 'a':
            count += 1
    return count

print(count_a("Apple"))
```
```text
1
```

---

## Common Mistakes

- **Skipping the Plan:** Jumping straight to Step 3 (Execute) is the #1 reason beginners get stuck. If you don't know *what* you want to code, you won't know *how* to code it.
- **Only testing the "Happy Path":** Your code works for `[1, 2, 3]`, so you submit it. But the automated grader tests `[]` and your code crashes. Always test edge cases manually before submitting.
- **Writing too much code at once:** Don't write 50 lines of code and then hit "Run". Write 5 lines, test them. Write the next 5 lines, test them.

---

## Key Takeaway

Problem-solving is a structured process: Understand, Plan, Execute, and Test. The actual coding (Execute) is only 25% of the work. By thoroughly understanding the problem, identifying edge cases, and planning your algorithm in plain English before writing code, you will write cleaner, more reliable software.

---

## References
- [GeeksforGeeks: How to solve a programming problem](https://www.geeksforgeeks.org/how-to-solve-a-programming-problem/)
