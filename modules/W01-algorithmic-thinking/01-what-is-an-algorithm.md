# What is an Algorithm?

---

## Learning Objectives

By the end of this topic, you will be able to:

- Define what an **algorithm** is in the context of computer science.
- Understand the difference between an algorithm and a computer program.
- Identify the key characteristics of a good algorithm (correctness and efficiency).
- Explain why **algorithmic thinking** is an essential skill for software engineers.
- Differentiate between a brute-force approach and an optimized approach.

---

## Overview

In the first semester, you learned how to write Python code to make the computer perform tasks. Now, we are shifting our focus from *writing code* to *solving problems*. An algorithm is the step-by-step logic used to solve a problem. Before writing a single line of Python, you must first design the algorithm. Understanding algorithms is the foundation of becoming an industry-ready engineer who can write efficient, scalable, and correct software.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Algorithm** | A finite, unambiguous sequence of steps to solve a problem. |
| **Spec-First Discipline** | Describing the solution in plain English before writing code. |
| **Brute-Force** | The most straightforward, obvious approach to solve a problem, often slow but correct. |
| **Optimization** | Improving an algorithm to run faster or use less memory. |
| **Algorithmic Thinking** | Breaking down a problem and designing a logical sequence of steps to solve it. |

---

## Defining an Algorithm

An **algorithm** is a finite sequence of well-defined, unambiguous instructions to solve a specific problem or perform a computation. 

**Daily-life example:** Think of a recipe for baking a cake. The recipe provides a list of ingredients (inputs) and a sequence of steps (instructions) to produce the cake (output). If you follow the steps exactly, you will get the cake. 

**Beginner-friendly way to remember:** A program is the *code* you write in Python. An algorithm is the *idea* or *logic* behind the code. The algorithm exists before the code is even written.

### Key Characteristics of a Good Algorithm

Not all step-by-step instructions are good algorithms. A proper computational algorithm must be:
1. **Unambiguous:** Each step must be clear and leave no room for interpretation.
2. **Finite:** It must eventually stop and produce a result.
3. **Correct:** It must solve the problem for all valid inputs.
4. **Efficient:** It should use computer resources (time and memory) wisely.

---

## The Spec-First Discipline

In this program, we follow the **Spec-first discipline**: describe the solution in plain English before writing code. 

Why? Because if you cannot explain the steps in plain English, you cannot write code for it. This helps you separate the *logic* of the solution from the *syntax* of Python.

```python
# Spec: 
# 1. Take a list of numbers as input.
# 2. Keep track of the highest number seen so far.
# 3. Look at each number one by one.
# 4. If the current number is higher than the highest seen, update the highest.
# 5. Return the highest number.

# Code:
def find_maximum(numbers):
    highest = numbers[0]
    for num in numbers:
        if num > highest:
            highest = num
    return highest

print(find_maximum([4, 1, 9, -2, 5]))
```
```text
9
```

---

## Correctness vs. Efficiency

When evaluating an algorithm, we look at two main factors:

1. **Correctness:** Does it work? Does it handle all edge cases? An algorithm that is incredibly fast but gives the wrong answer is useless.
2. **Efficiency:** How much time and memory does it take? If an algorithm takes 10 years to run, it is technically correct but practically useless.

### Brute-Force to Optimized

When solving a new problem, it is perfectly fine to start with a **brute-force** solution. A brute-force algorithm is the most straightforward, obvious approach. It usually checks every single possibility. It is often slow, but it is *correct*.

Once you have a working brute-force solution, you can then analyze it and find ways to **optimize** it (make it faster or use less memory).

**Example —** Problem: *Find a specific word in a physical dictionary.*
- **Brute-Force:** Start at page 1 and read every single word until you find it. (Correct, but very slow).
- **Optimized:** Open the dictionary to the middle, see if the word is in the first half or second half, ignore the half where the word cannot be, and repeat this process. (Correct and very fast).

---

## Try it Yourself

**Scenario:** You need to find if the number `7` exists in the list `[2, 4, 6, 8, 10]`.

**(a)** Write the Spec (plain English steps) for a brute-force algorithm to solve this.

*Your Spec:*
1. Start at the first number in the list.
2. Check if it is equal to 7.
3. If yes, stop and say "Found it".
4. If no, move to the next number and repeat step 2.
5. If we reach the end and haven't found it, say "Not found".

**(b)** Translate your Spec into Python code and test it.

```python
def find_number(target, numbers):
    for num in numbers:
        if num == target:
            return "Found it"
    return "Not found"

print(find_number(7, [2, 4, 6, 8, 10]))
```
```text
Not found
```

---

## Common Mistakes

- **Writing code before thinking:** Jumping straight into Python without a Spec often leads to confusing, buggy code. Always plan the algorithm first.
- **Ignoring Edge Cases:** An algorithm might work for `[1, 2, 3]`, but crash if you give it an empty list `[]`. A correct algorithm handles edge cases.
- **Confusing an Algorithm with a Program:** Remember, an algorithm is language-independent. The same algorithm can be written in Python, Java, or C++.

---

## Key Takeaway

An algorithm is the logical, step-by-step plan to solve a problem. Before writing code, you should always design the algorithm first (Spec-first discipline). A good algorithm is not just correct; it is also efficient. Starting with a simple brute-force solution and optimizing it is a standard and effective problem-solving strategy.

---

## References
- [Khan Academy: What is an algorithm?](https://www.khanacademy.org/computing/computer-science/algorithms/intro-to-algorithms/v/what-are-algorithms)
