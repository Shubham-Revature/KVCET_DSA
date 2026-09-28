# Number Systems and Bits

---

## Learning Objectives

By the end of this topic, you will be able to:

- Understand the difference between the **Base-10 (Decimal)** and **Base-2 (Binary)** number systems.
- Define what **bits** and **bytes** are in computer science.
- Manually convert simple numbers between binary and decimal.
- Use Python's built-in functions to handle binary conversions.

---

## Overview

When you type `x = 5` in Python, you understand it as the number five. But the computer does not. At the hardware level, a computer is just a massive collection of microscopic electronic switches. A switch can only be in one of two states: **ON** or **OFF**. 

To make a computer do math, we have to translate our human numbers into a language of ONs and OFFs. This language is the **Binary number system**. Understanding how computers represent data is a fundamental part of algorithmic thinking, especially when you start dealing with memory constraints and bitwise algorithms later on.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Base-10 (Decimal)** | The number system humans use, containing 10 digits (0 through 9). |
| **Base-2 (Binary)** | The number system computers use, containing 2 digits (0 and 1). |
| **Bit** | Short for "Binary Digit". A single 0 or 1. The smallest unit of data in a computer. |
| **Byte** | A group of 8 bits. |

---

## Human Counting vs. Computer Counting

### Base-10 (Decimal)
Humans have 10 fingers, so we naturally count in Base-10. Each position in a number represents a power of 10 (1s, 10s, 100s, 1000s).

Take the number **342**:
*   $2 \times 1 = 2$
*   $4 \times 10 = 40$
*   $3 \times 100 = 300$
*   **Total = 342**

### Base-2 (Binary)
Computers only have two "fingers" (ON/1 and OFF/0), so they count in Base-2. Each position in a binary number represents a power of 2 (1, 2, 4, 8, 16, 32...).

Take the binary number **1011**:
*   Rightmost bit: $1 \times 1 = 1$
*   Next bit: $1 \times 2 = 2$
*   Next bit: $0 \times 4 = 0$
*   Leftmost bit: $1 \times 8 = 8$
*   **Total = 8 + 0 + 2 + 1 = 11 (in decimal)**

**Daily-life example:** Think of binary like a row of lightbulbs. Each bulb has a specific value (8, 4, 2, 1). To make the number 11, you turn ON the '8' bulb, the '2' bulb, and the '1' bulb ($8+2+1=11$). You leave the '4' bulb OFF.

---

## Binary in Python

You don't always have to convert numbers by hand. Python has built-in tools to handle binary.

### 1. Decimal to Binary
Use the `bin()` function to convert a normal integer to its binary string representation. Python adds an `0b` prefix to show that it is a binary string.

```python
number = 11
binary_string = bin(number)

print(binary_string)
```
```text
0b1011
```

### 2. Binary to Decimal
Use the `int()` function, but pass a second argument `2` to tell Python that the string you are providing is in Base-2.

```python
decimal_number = int("1011", 2)

print(decimal_number)
```
```text
11
```

---

## Try it Yourself

**Scenario:** You need to represent the decimal number **13** in binary. 

**(a) Manual Conversion:** 
*Your Spec:* 
Look at the powers of 2: [8, 4, 2, 1].
- Do we need an 8? Yes. (We have 5 left over).  -> `1`
- Do we need a 4? Yes. (We have 1 left over).  -> `1`
- Do we need a 2? No. (1 is less than 2).     -> `0`
- Do we need a 1? Yes. (We have 0 left over).  -> `1`
Result: `1101`

**(b) Verify with Python:**
Write a quick script using `assert` to prove your manual conversion is correct.

```python
# Convert 13 to binary
assert bin(13) == "0b1101"

# Convert binary "1101" back to decimal
assert int("1101", 2) == 13

print("Binary conversions are correct!")
```
```text
Binary conversions are correct!
```

---

## Common Mistakes

- **Treating binary strings as regular numbers:** If you type `x = 1011` in Python, Python thinks you mean the decimal number one thousand and eleven. To write a binary number directly in Python code, you must use the `0b` prefix: `x = 0b1011`.
- **Forgetting the `0b` prefix:** When using `bin()`, remember that the first two characters are `0b`. If you only want the digits, you need to slice the string: `bin(13)[2:]` will give you `"1101"`.

---

## Key Takeaway

Computers process everything in binary (Base-2), a system of 0s and 1s representing powers of 2. A single 0 or 1 is a bit, and 8 bits make a byte. While Python handles the translation for you automatically, knowing how numbers are stored under the hood is critical for understanding memory limits and writing advanced algorithms later in your career.

---

## References
- [Math is Fun: Binary Number System](https://www.mathsisfun.com/binary-number-system.html)
- [Python Documentation: `bin()`](https://docs.python.org/3/library/functions.html#bin)
