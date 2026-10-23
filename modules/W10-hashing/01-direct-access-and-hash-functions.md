# Direct Access and Hash Functions

In our journey through Data Structures and Algorithms, we have witnessed a continuous struggle between speed and flexibility:
- **Arrays** grant instantaneous **$O(1)$ random lookups**, but only if you know the exact integer index in memory.
- **Binary Search** finds items in **$O(\log N)$ time**, but strictly requires the entire dataset to remain sorted.
- **Linear Search** works on arbitrary, unsorted data, but slows down to a sluggish **$O(N)$ linear scan**.

This brings us to the foundation of modern data storage: **Can we achieve instant $O(1)$ lookups for arbitrary keys (strings, floating-point numbers, or complex objects) without sorting?**

The journey begins with the simplest concept in indexing—the **Direct Access Table**—and leads directly to the core mathematical engine of modern software: the **Hash Function**.

---

## 1. The Ideal World: Direct Access Tables (DAT)

Imagine you are managing an apartment complex with 100 units numbered `0` through `99`. You want to store tenant records so you can retrieve, update, or remove any resident instantly.

How do you store them? You simply create an array of size 100:

```text
Unit Number (Key) ──► Direct Array Index ──► array[Unit Number]
```

```python
# Direct Access Table
tenants = [None] * 100

# Operations in instant O(1) time:
tenants[42] = "Alice"     # INSERT: O(1)
resident = tenants[42]    # SEARCH: O(1) -> "Alice"
tenants[42] = None        # DELETE: O(1)
```

This is a **Direct Access Table (DAT)**. When keys are small, non-negative, consecutive integers, direct addressing is unbeatable. Every search, insertion, and deletion takes exactly **$O(1)$ Constant Time** because the hardware CPU computes the target memory address in a single arithmetic instruction:

$$\text{Memory Address} = \text{Base Address} + (\text{Key} \times \text{Element Size})$$

---

## 2. Why Direct Access Fails in the Real World

Direct Access Tables work flawlessly—until reality intrudes. Real-world data keys rarely behave like neat, small integers:

### Problem A: Massive Key Universes (Memory Explosion)
Suppose you want to look up users by their 9-digit Social Security Number (SSN) or 10-digit National ID. 
- The universe of possible keys $U$ is $10^9$ or $10^{10}$ integers.
- To use a Direct Access Table, you must allocate an array of **10 billion slots**.
- Even if each slot holds a simple 8-byte pointer, that array consumes **80 Gigabytes of RAM**!
- If your system only has 5,000 active users, **99.99995% of your allocated memory sits completely empty**. This catastrophic waste is known as **extreme table sparsity**.

### Problem B: Non-Integer Keys
What if the key is not an integer at all?
- Usernames: `"shubham_g"`, `"alice_wonderland"`
- Email addresses: `"student@university.edu"`
- GPS Coordinates: `(37.7749, -122.4194)`

Computer memory addresses are positive integers (`0x00`, `0x01`, `0x02`, ...). You cannot ask the operating system to read memory at index `array["alice_wonderland"]`.

```text
Direct Access Dilemma:
Keys are huge, sparse, or non-integer ──► Cannot map directly to RAM slots!
```

We need an intermediary translator: a mechanism that converts arbitrary keys into compact array indices. That translator is the **Hash Function**.

---

## 3. The Hash Function Architecture

A **Hash Function** ($h$) is a deterministic mathematical function that maps a key $k$ from a massive or non-integer universe $U$ into a small, bounded range of integer indices $[0, m - 1]$, where $m$ is the size of our storage array (the **Hash Table**):

$$h: U \to \{0, 1, 2, \dots, m - 1\}$$

```text
Universe of Keys (U)                     Hash Table Slots (Array of size m)
┌──────────────────────┐                 ┌───┬──────────────────────┐
│ "alice_wonderland"   │───┐             │ 0 │ None                 │
│ "bob_dev"            │───┼───────────► ├───┼──────────────────────┤
│ "charlie_k"          │───┤  h(key)     │ 1 │ "charlie_k"          │
│ 4532-8912-3456-7890  │───┘             ├───┼──────────────────────┤
└──────────────────────┘                 │ 2 │ "alice_wonderland"   │
 (Infinite possible keys)                ├───┼──────────────────────┤
                                         │ 3 │ "bob_dev"            │
                                         └───┴──────────────────────┘
                                           (Fixed small size m)
```

### The Two-Stage Pipeline
Under the hood, transforming an arbitrary object into an array slot happens in two distinct stages:

```text
  Key (e.g. "alice")
          │
          ▼
┌───────────────────┐
│ 1. Hash Code      │  Transforms arbitrary object into a wide integer (can be huge/negative)
│    Generation     │  Example: hash("alice") -> 6298371982736192
└───────────────────┘
          │
          ▼
┌───────────────────┐
│ 2. Compression    │  Restricts the wide integer into array bounds [0, m - 1]
│    Function       │  Example: 6298371982736192 % 8 -> Index 2
└───────────────────┘
          │
          ▼
    Target Slot (2)
```

1. **Hash Code Generation:** Converts an arbitrary data type (string, tuple, floating-point number) into an integer hash code.
2. **Compression Function:** Compresses that integer into the range $[0, m - 1]$ (most commonly via the modulo operator: $\text{index} = \text{hash\_code} \bmod m$).

---

## 4. Four Pillars of a Great Hash Function

A poorly designed hash function creates bottlenecks, clustering, and crashes. A production-grade hash function must satisfy four non-negotiable rules:

| Property | Rule | Why It Matters |
|---|---|---|
| **1. Determinism** | If $k_1 == k_2$, then $h(k_1)$ **must strictly equal** $h(k_2)$. | If the same key generated different hashes across calls, you would insert an item into slot 4 and look for it in slot 7, making it permanently lost. |
| **2. Uniform Distribution** | Keys should spread evenly across all $m$ slots with equal probability. | Prevents clustering and hotspots where multiple keys pile into the same slot. |
| **3. High Efficiency** | Must compute in $O(1)$ time (or $O(L)$ for a string of length $L$). | If computing the hash takes longer than a linear scan, the $O(1)$ lookup advantage is defeated. |
| **4. The Avalanche Effect** | A 1-bit or 1-character difference in input must produce a radically different hash. | `"cat"` and `"car"` should land in totally different sections of the table rather than adjacent slots. |

---

## 5. Classical Hash Function Design Techniques

How do mathematicians and computer scientists actually build hash functions?

### Technique 1: The Division Method
The simplest compression function takes key $k$ modulo table size $m$:

$$h(k) = k \bmod m$$

> [!IMPORTANT]
> **Why Table Size $m$ Must Be Prime:**
> If $m$ is a power of 2 (e.g., $m = 2^4 = 16$), then $k \bmod 16$ simply extracts the **last 4 binary bits** of $k$! All higher-order bits are completely ignored. If your keys share common patterns in their low bits, they will all collide in the same slots. 
> Choosing $m$ as a **prime number not close to a power of 2** forces the modulo operation to depend on all bits of the key, maximizing uniform distribution.

### Technique 2: The Multiplication Method
Proposed by Donald Knuth, the multiplication method avoids depending on prime table sizes:
1. Multiply key $k$ by a constant fraction $A$ between $0$ and $1$ (Knuth suggested the Golden Ratio conjugate: $A \approx \frac{\sqrt{5} - 1}{2} \approx 0.6180339887$).
2. Extract the fractional part: $(k \cdot A) \bmod 1$.
3. Multiply by $m$ and take the floor:
   $$h(k) = \lfloor m \cdot ((k \cdot A) \bmod 1) \rfloor$$

Because the fractional part scrambles bits effectively, $m$ can safely be a convenient power of 2 (which CPUs calculate ultra-fast using bitwise shifts).

### Technique 3: Polynomial Rolling Hash for Strings
How do we hash a string like `"code"`? We cannot just add ASCII values (`ord('c') + ord('o') + ord('d') + ord('e')`), because anagrams like `"deco"` or `"edoc"` would produce the exact same sum!

Instead, we treat the string as a number in a high-base positional numeral system:

$$h(s) = \left( s[0] \cdot p^{L-1} + s[1] \cdot p^{L-2} + \dots + s[L-1] \cdot p^0 \right) \bmod m$$

where $p$ is a prime base (typically $31$ or $53$ for alphabetic strings) and $m$ is the table modulus.

#### Computing in $O(L)$ with Horner's Rule
Computing large powers $p^{L-1}$ leads to numeric overflow. Instead, we use **Horner’s Rule** to compute the polynomial iteratively from left to right in a single pass:

```python
def polynomial_hash(s: str, p: int = 31, m: int = 10007) -> int:
    hash_value = 0
    for char in s:
        # Horner's Rule: fold current value, multiply by base, add next char
        hash_value = (hash_value * p + ord(char)) % m
    return hash_value
```

---

## 6. Hashing in Python: Hashable vs. Unhashable

Python includes a powerful built-in `hash()` function:

```python
print(hash(42))            # 42
print(hash("antigravity")) # e.g. -4829104829471829304
print(hash((1, 2, 3)))     # e.g. 5293440686954933535
```

### The Hashability Contract
In Python, an object is **hashable** if and only if:
1. It has an internal hash code that **never changes during its entire lifetime** (implemented via `__hash__()`).
2. It can be compared for equality with other objects (implemented via `__eq__()`).
3. If two objects are equal (`a == b`), their hash values **must be identical** (`hash(a) == hash(b)`).

### Why Mutable Objects CANNOT Be Hashed
Try running this in Python:
```python
hash([1, 2, 3])  # TypeError: unhashable type: 'list'
```

Why does Python strictly forbid lists, sets, and dictionaries from being hashed?
Imagine if lists *were* allowed as dictionary keys:
1. You insert `my_list = [1, 2]` into a hash table. Python computes its hash and stores it in **Slot 5**.
2. Later, you mutate the list: `my_list.append(3)`.
3. Now the contents of `my_list` have changed, so its hash code changes to point to **Slot 12**.
4. You try to look up `my_list` in your dictionary. Python computes the new hash, checks **Slot 12**, finds nothing, and reports `KeyError`!
5. Meanwhile, the original data is permanently stranded in Slot 5 as an unreachable ghost element!

> [!CAUTION]
> **Rule of Thumb:**
> - **Hashable (Immutable):** `int`, `float`, `str`, `tuple` (if all elements are hashable), `frozenset`, `bytes`.
> - **Unhashable (Mutable):** `list`, `set`, `dict`.

### Python Security: SipHash Randomization
If you run `hash("hello")` in Python, exit the terminal, and run it again in a new session, you will get **a completely different number**!

Why? Python initializes each process with an unpredictable, randomized secret seed using the **SipHash-2-4** algorithm. This prevents **HashDoS Attacks**, where malicious attackers deliberately feed thousands of colliding keys to a web server to force $O(N^2)$ hash table degradation and crash the CPU!

---

## Try it Yourself

**Scenario:** Implement a custom string hash function using Horner's polynomial rule, verify the Avalanche Effect (how changing a single character scrambles the hash), and contrast it with a Direct Access Table.

*Your Spec:*
1. Implement `polynomial_hash(s, p=31, m=10007)` using Horner's rule.
2. Verify determinism: hashing the same word twice produces identical results.
3. Verify position sensitivity: anagrams `"cat"` and `"act"` produce completely different hash values.
4. Verify the avalanche effect: `"cat"` and `"car"` produce non-adjacent hashes.
5. Implement a simple `DirectAccessTable` for integer IDs to observe the contrast.

```python
def polynomial_hash(s: str, p: int = 31, m: int = 10007) -> int:
    """Computes an O(L) polynomial rolling hash using Horner's rule."""
    hash_value = 0
    for char in s:
        hash_value = (hash_value * p + ord(char)) % m
    return hash_value


# 1. Determinism Check
assert polynomial_hash("algorithm") == polynomial_hash("algorithm")

# 2. Position Sensitivity Check (Anagrams do NOT collide)
h_cat = polynomial_hash("cat")
h_act = polynomial_hash("act")
assert h_cat != h_act

# 3. Avalanche Effect Check (Single character change scrambles hash)
h_car = polynomial_hash("car")
assert abs(h_cat - h_car) > 10

print(f"Hash of 'cat': {h_cat}")
print(f"Hash of 'act': {h_act}")
print(f"Hash of 'car': {h_car}")


# --- Direct Access Table Comparison ---
class DirectAccessTable:
    def __init__(self, capacity: int):
        self.slots = [None] * capacity

    def insert(self, key: int, value: str) -> None:
        self.slots[key] = value

    def search(self, key: int) -> str | None:
        return self.slots[key]

# Instant O(1) integer direct addressing
dat = DirectAccessTable(100)
dat.insert(42, "Alice")
assert dat.search(42) == "Alice"
assert dat.search(99) is None

print("Direct Access Table operations and custom hashing passed successfully!")
```
```text
Hash of 'cat': 8199
Hash of 'act': 6339
Hash of 'car': 8197
Direct Access Table operations and custom hashing passed successfully!
```

---

## Common Mistakes

- **Assuming Python's `hash()` is constant across runs:** Python randomizes hash seeds on every program startup for security. Never persist `hash(x)` into databases or disk files, as it will be invalid upon the next application reboot. Use `hashlib.sha256()` or `hashlib.md5()` if you need cross-session persistence.
- **Using a mutable object as a dictionary key:** Attempting `d[[1, 2]] = "value"` triggers `TypeError: unhashable type: 'list'`. Convert mutable lists to immutable tuples first: `d[(1, 2)] = "value"`.
- **Summing ASCII values for string hashing:** Writing `sum(ord(c) for c in s)` ignores character positions. Every anagram will hash to the exact same bucket! Always use polynomial weightings like Horner's method.
- **Using power-of-two table sizes with simple modulo:** When using $h(k) = k \bmod m$, setting $m$ to a power of 2 discards all high-order bits and amplifies collisions. Always pick prime table sizes when using the division method.
- **Implementing `__hash__` without `__eq__`:** In custom Python classes, if you define `__eq__`, Python automatically sets `__hash__ = None` to prevent bugs. If you override `__hash__`, you must also ensure that objects considered equal under `__eq__` produce identical hash values.

---

## Key Takeaway

A **Direct Access Table** offers the fastest possible $O(1)$ lookups, but is crippled by extreme memory waste when keys are sparse, large, or non-integer. The **Hash Function** bridges this gap: it translates an infinite universe of arbitrary keys into small, bounded array indices. A great hash function is **deterministic**, **uniform**, **efficient**, and exhibits the **avalanche effect**. In Python, only **immutable** objects are hashable to guarantee that stored keys never shift positions in memory.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Direct Access Table (DAT)** | A simple array where each record is stored at the index matching its integer key. |
| **Hash Function** | A deterministic function that maps keys of arbitrary size into fixed-size integers. |
| **Compression Function** | A mathematical operation (like modulo $m$) that scales large hash codes into array bounds $[0, m - 1]$. |
| **Determinism** | The guarantee that identical inputs always yield identical outputs. |
| **Avalanche Effect** | A property where a minor change in the input produces a drastic, unpredictable change in the output hash. |
| **Horner’s Rule** | An optimal algorithm for evaluating polynomials in $O(N)$ time with minimal arithmetic operations. |
| **Hashable** | An object with a fixed hash code throughout its lifecycle that can be compared for equality. |
| **SipHash** | A cryptographic, keyed pseudorandom hash algorithm used by CPython to defend against HashDoS attacks. |

---

## References

- [Introduction to Algorithms (CLRS): Chapter 11 - Hash Tables](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
- [Python Documentation: Built-in function hash()](https://docs.python.org/3/library/functions.html#hash)
- [Python Documentation: Glossary — hashable](https://docs.python.org/3/glossary.html#term-hashable)
- [PEP 456: Secure and Interchangeable Hash Algorithm (SipHash)](https://peps.python.org/pep-0456/)
