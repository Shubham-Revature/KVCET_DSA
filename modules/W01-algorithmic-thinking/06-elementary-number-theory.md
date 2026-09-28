# Elementary Number Theory

---

## Learning Objectives

By the end of this topic, you will be able to:

- Master the **modulo operator (`%`)** and understand its common use cases.
- Use math to extract and manipulate the individual digits of a number.
- Distinguish between standard division (`/`) and integer division (`//`).
- Implement a basic algorithm to check if a number is prime.

---

## Overview

You do not need a degree in advanced mathematics to be a great software engineer. However, a huge portion of algorithmic coding interviews and platform challenges rely on **elementary number theory**—specifically, understanding how division and remainders work. Mastering these basic math operations in Python gives you the tools to solve problems involving digit manipulation, repeating patterns, and primes.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Modulo (`%`)** | An operator that returns the remainder of a division operation. |
| **Integer Division (`//`)** | An operator that divides two numbers and rounds down to the nearest whole number, dropping any decimal remainder. |
| **Prime Number** | A number greater than 1 that has exactly two factors: 1 and itself. |

---

## 1. The Magic of Modulo (`%`)

The modulo operator (`%`) calculates the remainder of division. It is arguably the most frequently used math operator in coding challenges.

```python
print(10 % 3)  # 10 divided by 3 is 3, with a remainder of 1.
print(15 % 5)  # 15 divided by 5 is 3, with a remainder of 0.
```

### Common Use Cases for Modulo:
**1. Checking Even/Odd:** Any number modulo 2 will be `0` if it is even, and `1` if it is odd.
```python
assert 4 % 2 == 0  # Even
assert 7 % 2 == 1  # Odd
```

**2. Cycling / Repeating Patterns:** Modulo wraps numbers around. If you have an array of 3 items (indices 0, 1, 2) and you want to continuously loop through them, modulo keeps your index in bounds.
```python
colors = ["Red", "Green", "Blue"]
for i in range(5):
    index = i % 3
    print(colors[index], end=" ")
```
```text
Red Green Blue Red Green 
```

---

## 2. Digit Extraction

A classic algorithmic problem asks you to sum the digits of a number, or reverse a number (e.g., turn `456` into `654`). You can do this by converting the number to a string, but the mathematical way is faster and heavily tested in interviews.

You use a combination of Modulo (`% 10`) and Integer Division (`// 10`).

- `number % 10` always gives you the **last digit**.
- `number // 10` **removes the last digit**.

**Example:**
```python
number = 456

# Get the last digit
last_digit = number % 10
print(f"Last digit: {last_digit}")

# Remove the last digit
number = number // 10
print(f"Number is now: {number}")
```
```text
Last digit: 6
Number is now: 45
```

---

## 3. Basic Prime Check

A prime number is a number greater than 1 that is only divisible by 1 and itself (e.g., 2, 3, 5, 7, 11).

How do we check if a number $N$ is prime? 
**The Brute-Force Algorithm:** Try dividing $N$ by every number from 2 up to $N-1$. If any of those numbers divide evenly (i.e., `N % i == 0`), then $N$ is not prime.

```python
def is_prime(n):
    if n <= 1:
        return False
        
    for i in range(2, n):
        if n % i == 0:
            return False  # We found a divisor!
            
    return True  # No divisors found, it must be prime

assert is_prime(7) == True
assert is_prime(10) == False
```

*(Note: In Week 2, we will look at how to optimize this algorithm from $O(N)$ to $O(\sqrt{N})$ to make it much faster!)*

---

## Try it Yourself

**Scenario:** Write an algorithm to find the sum of all the digits in the number `825`. (The answer should be $8 + 2 + 5 = 15$).

**Your Spec:**
1. Start a `total` variable at 0.
2. While the number is greater than 0:
    - Extract the last digit using `% 10`.
    - Add that digit to the `total`.
    - Remove the last digit from the number using `// 10`.
3. Return the total.

**Your Code:**
```python
def sum_digits(n):
    total = 0
    while n > 0:
        last_digit = n % 10
        total += last_digit
        n = n // 10
    return total

assert sum_digits(825) == 15
print("Digit sum test passed!")
```
```text
Digit sum test passed!
```

---

## Common Mistakes

- **Confusing `/` and `//`:** In Python, standard division (`/`) always returns a float (a decimal). `10 / 2` returns `5.0`. If you are extracting digits or doing exact integer math, you must use integer division (`//`). `10 // 2` returns `5`.
- **Negative Modulo:** In Python, modulo with negative numbers works slightly differently than in languages like C++ or Java. `-5 % 3` returns `1` in Python, not `-2`. Be extremely careful when using modulo on negative numbers.

---

## Key Takeaway

Elementary number theory forms the building blocks for countless algorithms. The modulo operator (`%`) helps identify patterns and remainders, while combining it with integer division (`//`) allows you to mathematically slice and manipulate digits. Understanding these concepts allows you to avoid messy string conversions and write highly efficient mathematical algorithms.

---

## References
- [GeeksforGeeks: Modulo Operator in Python](https://www.geeksforgeeks.org/what-is-a-modulo-operator-in-python/)
- [Khan Academy: Modular Arithmetic](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/what-is-modular-arithmetic)
