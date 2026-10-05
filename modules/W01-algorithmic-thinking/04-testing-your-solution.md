# Testing Your Solution

A common trap for junior engineers is the belief that "if the code runs without an error message, it must be correct." In reality, a program that runs perfectly can still produce completely wrong answers (a logical error). 

In the real world, you don't submit code to your manager and hope it works. You submit code *along with proof* that it works. This is the **test-your-solution discipline**. By thoroughly testing your own code, you catch embarrassing bugs before anyone else sees them.

---

## 1. The Manual Dry Run

Before you even press the "Run" button on your code, you should perform a **dry run**. This means pretending to be the computer and tracing the values of your variables on a piece of paper as you step through the logic.

**Example Code:**
```python
def sum_to_n(n):
    total = 0
    for i in range(n):
        total += i
    return total
```

**Dry Run with `n = 3`:**
1. `total` starts at 0.
2. `range(3)` produces `0, 1, 2`.
3. Loop 1: `i = 0`. `total` becomes `0 + 0 = 0`.
4. Loop 2: `i = 1`. `total` becomes `0 + 1 = 1`.
5. Loop 3: `i = 2`. `total` becomes `1 + 2 = 3`.
6. Loop ends. Returns `3`.

Wait! The mathematical sum of numbers up to 3 is $1 + 2 + 3 = 6$. Our dry run returned `3`. We just found a bug *without* running the code! (The fix is `range(1, n + 1)`).

---

## 2. Automating Tests with `assert`

Instead of using `print()` to test your functions and checking the terminal with your eyes, you can automate testing using Python's `assert` keyword.

An `assert` statement tells Python: *"I expect this condition to be True. If it is False, stop the program immediately and throw an error."*

```python
def multiply(a, b):
    return a * b

# Happy path tests
assert multiply(3, 4) == 12
assert multiply(5, 5) == 25

# Edge case tests
assert multiply(10, 0) == 0    # Testing zero
assert multiply(-2, 3) == -6   # Testing negative numbers

print("All tests passed!")
```
```text
All tests passed!
```

If any of those assertions were wrong (e.g., if you had a bug in your function), Python would immediately stop and print an `AssertionError`, pointing directly to the line that failed.

---

## 3. Happy Path vs. Edge Cases

When you write tests for your code, you must cover two distinct categories:

1. **Happy Path:** These are the inputs the problem was designed for. If you are writing a sorting function, a happy path test is `[4, 1, 3, 2]`. 
2. **Edge Cases:** These are the extreme boundaries of the problem's constraints. You actively want to try to break your code.
   * What if the list is empty `[]`?
   * What if the list has only one item `[5]`?
   * What if the list is already sorted `[1, 2, 3, 4]`?
   * What if the list has duplicates `[2, 1, 2, 1]`?

**Daily-life example:** Testing an elevator. 
- *Happy path:* 3 people get in, press floor 5, and it goes up.
- *Edge case:* 20 people get in (over weight limit). Someone presses all the buttons. The power goes out mid-ride. A good engineer tests all of these.

---

## Try it Yourself

**Scenario:** You have written a function `is_even(number)` that should return `True` if a number is even, and `False` if it is odd.

Write three `assert` statements to test this function. Include one happy path test for an even number, one happy path test for an odd number, and one edge case test for zero.

```python
def is_even(number):
    return number % 2 == 0

# (a) Happy path for an even number
assert is_even(4) == True

# (b) Happy path for an odd number
assert is_even(7) == False

# (c) Edge case for zero
assert is_even(0) == True

print("All tests passed!")
```
```text
All tests passed!
```

---

## Common Mistakes

- **Testing with Prints:** Writing `print(my_function(5))` forces you to look at the terminal and mentally verify the answer. If you have 20 tests, your terminal will be flooded. Use `assert` instead—if the code finishes silently, you know everything is correct.
- **Assuming the Grader is Wrong:** If your code fails a hidden test case on HackerRank or LeetCode, beginners often assume the platform's test case is broken. 99.9% of the time, the platform is right. You missed an edge case. Go back to the constraints!

---

## Key Takeaway

Never submit code without testing it first. Use the **dry run** technique to mentally trace your logic and find bugs early. When translating logic to code, use Python's `assert` statements to build automated tests covering both the expected **happy path** and the tricky **edge cases**.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Test-Your-Solution Discipline** | The professional habit of actively trying to break your own code to prove it works. |
| **Happy Path** | The default, expected scenario where normal inputs are provided. |
| **Dry Run / Desk Checking** | Manually stepping through your code line-by-line with a pen and paper. |
| **`assert` Statement** | A Python keyword used to test if a condition is `True`. If it is `False`, the program crashes with an `AssertionError`. |

---

## References
- [Python Documentation: The assert statement](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement)
