# Hash Collisions and Resolution Strategies

In Lesson 01, we saw how a hash function translates a massive universe of keys into a compact array of size $m$. 

However, this compression comes with an unavoidable mathematical truth: **No matter how brilliant your hash function is, collisions are mathematically guaranteed to happen.**

A **Hash Collision** occurs when two distinct keys, $k_1 \neq k_2$, produce the exact same array index:

$$h(k_1) = h(k_2)$$

If `"alice"` and `"bob"` both hash to index `4`, which one gets to sit in slot 4? Where does the other one go? 

In this lesson, we explore why collisions are inevitable and master the two primary paradigms computer scientists use to resolve them: **Separate Chaining** and **Open Addressing**.

---

## 1. The Inevitability of Collisions

Beginners often ask: *"Can't we just design a perfect hash function that never collides?"*

The answer is a hard mathematical **no**. Collisions are enforced by two fundamental principles of mathematics:

### Principle A: The Pigeonhole Principle
If you have **10 pigeons** and only **9 pigeonholes**, at least one hole must contain more than one pigeon.

```text
Keys (Pigeons):         [ "alice",  "bob",  "charlie",  "david",  "eve" ]  (5 keys)
Slots (Pigeonholes):    [ Slot 0,   Slot 1,  Slot 2,   Slot 3 ]             (4 slots)
                                      ▲
                    At least one slot MUST hold 2+ keys!
```

In any real-world application:
- The universe of possible keys $U$ is practically infinite (all possible strings, names, email addresses).
- The table size $m$ is finite (e.g., an array of 10,000 slots).
- Because $|U| \gg m$, collisions are a certainty.

### Principle B: The Birthday Paradox
Even when the number of stored keys $N$ is vastly smaller than the table size $m$, collisions happen **far earlier** than human intuition expects!

Consider a room of people:
- There are **365 days** in a year ($m = 365$).
- How many people ($N$) must be in the room before there is a **50% probability** that two of them share the exact same birthday?
- Most people guess $365 / 2 \approx 180$.
- The real mathematical answer is just **23 people**!

By the time your hash table has only $\sqrt{m}$ items, the chance of a collision exceeds 50%. A hash table cannot rely on avoiding collisions; it must have an explicit, rock-solid **collision resolution strategy**.

---

## 2. The Load Factor ($\alpha$)

Before examining collision strategies, we must define the vital health metric of any hash table: the **Load Factor ($\alpha$)**.

$$\alpha = \frac{N}{m} = \frac{\text{Number of elements stored}}{\text{Total number of array slots}}$$

The Load Factor measures how "crowded" the hash table is:
- If $\alpha$ is near `0.0`, the table is mostly empty (fast lookups, but wasted memory).
- If $\alpha$ approaches `1.0` or higher, collisions skyrocket, and performance can collapse from $O(1)$ down to $O(N)$!

---

## 3. Strategy 1: Separate Chaining (Open Hashing)

In **Separate Chaining**, array slots do not store values directly. Instead, every slot in the array points to an independent **Linked List** (or "bucket"):

```text
Array Slots               Buckets (Linked Lists)
[ 0 ] ─────────────► None
[ 1 ] ─────────────► [ ("charlie", 40) ] ──► None
[ 2 ] ─────────────► [ ("alice", 25) ] ───► [ ("eve", 90) ] ──► None  <-- COLLISION!
[ 3 ] ─────────────► [ ("bob", 30) ] ─────► None
```

### How Operations Work:
1. **Insert:** Hash the key to find index `i = h(key) % m`. Traverse the linked list at `buckets[i]`. If the key already exists, update its value. If not, append a new node to the front (or end) of the list.
2. **Search:** Hash the key to find index `i`. Walk the linked list comparing keys with `==` until the target is found.
3. **Delete:** Hash the key, locate the node in the list, and update pointers to remove it.

### Performance & Load Factor:
- In Separate Chaining, the load factor $\alpha$ can exceed $1.0$ (e.g., storing 200 items in 50 buckets $\implies \alpha = 4.0$).
- Under uniform hashing, the average length of each chain is exactly $\alpha$.
- **Average Search Time:** **$\Theta(1 + \alpha)$**. As long as we keep $\alpha \le 1.0$, lookups execute in instant **$O(1)$ time**.
- **Worst-Case Degradation:** **$O(N)$**. If an adversary feeds keys that all hash to the same bucket, the table degenerates into a single linked list!

> [!TIP]
> **Modern Industry Evolution (Treeification):**
> To defend against $O(N)$ malicious worst-case chaining, **Java 8+ `HashMap`** introduced *Treeification*. When any single bucket chain exceeds 8 elements, Java converts that linked list into a **Red-Black Balanced BST**. This upgrades the worst-case search from $O(N)$ to **$O(\log N)$**!

---

## 4. Strategy 2: Open Addressing (Closed Hashing)

In **Open Addressing**, there are **no linked lists**. Every element lives directly inside the table array itself!

```text
All elements reside directly within array slots:
Index:   [ 0 ]    [ 1 ]         [ 2 ]        [ 3 ]
Value:   None   ("charlie")   ("alice")    ("bob")
```

If a collision occurs at slot `h(key)`, the algorithm does not create a list. Instead, it systematically **probes** (searches) adjacent array slots until it discovers an empty slot.

Because every item takes up a slot, **Open Addressing strictly enforces $\alpha < 1.0$**. The table can never hold more items than its array capacity!

### Probing Techniques

How do we decide which slot to inspect next when a collision occurs?

#### 1. Linear Probing ($h(k, i) = (h(k) + i) \bmod m$)
If slot $h(k)$ is occupied, check the immediate next slot: $h(k) + 1$, then $h(k) + 2$, then $h(k) + 3$, wrapping around the array.

```text
Insert "apple" -> hashes to slot 2 (Empty -> Placed at 2)
Insert "apricot" -> hashes to slot 2 (Occupied! Try 2 + 1 = 3 -> Placed at 3)
Insert "avocado" -> hashes to slot 2 (Occupied! Try 3 Occupied! Try 4 -> Placed at 4)
```

- **Pros:** Phenomenal **CPU Cache Locality**. Because elements sit right next to each other in memory, the CPU hardware prefetcher loads them in a single clock cycle.
- **Cons: Primary Clustering.** Occupied slots cluster together into long, contiguous blocks (like a traffic jam on a highway). Any new key that hashes anywhere into the cluster makes the cluster even larger, causing search times to degrade rapidly!

#### 2. Quadratic Probing ($h(k, i) = (h(k) + c_1 i + c_2 i^2) \bmod m$)
Instead of stepping by 1, jump by quadratic increments: $+1^2, +2^2, +3^2, +4^2 \dots$
- **Pros:** Eliminates primary clustering by jumping away from crowded regions.
- **Cons: Secondary Clustering.** Keys that have the same initial hash $h(k)$ will still trace the exact same probe sequence.

#### 3. Double Hashing ($h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$)
The gold standard of open addressing. It employs **two independent hash functions**:
- $h_1(k)$ determines the **initial slot**.
- $h_2(k)$ determines the **step size** between probes.

```text
Key A: initial = 2, step = 3 ──► Probes: 2, 5, 8, 11...
Key B: initial = 2, step = 7 ──► Probes: 2, 9, 16, 23...
```

Even if Key A and Key B collide at slot 2, their subsequent paths diverge completely! This totally eliminates both primary and secondary clustering.
*(Note: $h_2(k)$ must never return 0, and must be relatively prime to $m$).*

---

## 5. The Tombstone Problem: Deletion in Open Addressing

Deleting an element from an Open Addressing table introduces a subtle, dangerous bug.

Imagine a table with Linear Probing:
1. `"alice"` hashes to **Slot 2** and is placed there.
2. `"bob"` also hashes to **Slot 2**. Slot 2 is full, so linear probing places `"bob"` in **Slot 3**.
3. Now, you delete `"alice"` from Slot 2 and reset it to `None`.
4. Next, you search for `"bob"`:
   - Python hashes `"bob"` $\implies$ starts at **Slot 2**.
   - The algorithm sees `None` in Slot 2!
   - Because an open-addressing search **stops when it hits an empty slot**, it assumes `"bob"` does not exist and returns `KeyError`! `"bob"` has become an unreachable ghost!

```text
Before Deletion:   [ Slot 2: "alice" ]  ──►  [ Slot 3: "bob" ]
Naively Setting None: [ Slot 2: None ]   ──►  [ Slot 3: "bob" ]
                                ▲
        Search for "bob" starts at 2, sees None, and STOPS! (Bug!)
```

### The Solution: The "Tombstone" (DELETED Sentinel)
When deleting an item in open addressing, you **never** reset the slot to `None`. Instead, you replace it with a special marker called a **Tombstone** (or `DELETED` object):

- **During Search:** If a probe encounters a `Tombstone`, it **keeps searching** down the chain.
- **During Insertion:** If a probe encounters a `Tombstone`, it can **recycle** that slot and overwrite it with the new key!

---

## 6. The Grand Showdown: Chaining vs. Open Addressing

| Dimension | Separate Chaining | Open Addressing (Linear / Double) |
|---|---|---|
| **Storage Model** | Array of Linked Lists / Trees | Flat contiguous array |
| **Load Factor Limit** | $\alpha$ can exceed $1.0$ | Must strictly keep $\alpha < 1.0$ (typically $\le 0.7$) |
| **Cache Locality** | Poor (nodes scattered across heap) | **Superior (sequential memory access)** |
| **Deletion** | Simple linked list pointer update | Complex (requires Tombstones) |
| **Memory Overhead** | High (stores extra pointer per node) | **Low (no node/pointer overhead)** |
| **Performance Degradation** | Graceful degradation as $\alpha$ grows | Severe cliff: freezes if $\alpha \to 1.0$ |
| **Production Usage** | Java `HashMap`, C++ `std::unordered_map` | **Python `dict`**, Redis, Go `map` |

---

## Try it Yourself

**Scenario:** Implement an Open Addressing Hash Table with **Linear Probing** and verify that the **Tombstone mechanism** prevents deleted keys from breaking subsequent lookups.

*Your Spec:*
1. Implement `LinearProbingHashTable` with fixed capacity 7.
2. Store keys `"A"` (ASCII sum 65 $\equiv$ 2 mod 7) and `"H"` (ASCII sum 72 $\equiv$ 2 mod 7) to force a collision at Slot 2.
3. Delete `"A"` and replace it with a `Tombstone`.
4. Assert that searching for `"H"` still succeeds.
5. Insert `"O"` (ASCII sum 79 $\equiv$ 2 mod 7) and verify that it recycles the tombstone slot!

```python
class LinearProbingHashTable:
    # Unique sentinel object to mark deleted slots
    TOMBSTONE = object()

    def __init__(self, capacity: int = 7):
        self.capacity = capacity
        self.keys = [None] * capacity
        self.values = [None] * capacity

    def _hash(self, key: str) -> int:
        # Sum of characters mod capacity (deterministic demo)
        return sum(ord(c) for c in key) % self.capacity

    def insert(self, key: str, value: int) -> None:
        idx = self._hash(key)
        first_tombstone = None

        for i in range(self.capacity):
            slot = (idx + i) % self.capacity

            if self.keys[slot] is None:
                # Place in first available tombstone or this empty slot
                target = first_tombstone if first_tombstone is not None else slot
                self.keys[target] = key
                self.values[target] = value
                return
            elif self.keys[slot] is self.TOMBSTONE:
                if first_tombstone is None:
                    first_tombstone = slot
            elif self.keys[slot] == key:
                self.values[slot] = value
                return

        # If table was searched and a tombstone was seen
        if first_tombstone is not None:
            self.keys[first_tombstone] = key
            self.values[first_tombstone] = value
            return

        raise RuntimeError("Hash table is completely full!")

    def get(self, key: str) -> int | None:
        idx = self._hash(key)

        for i in range(self.capacity):
            slot = (idx + i) % self.capacity

            if self.keys[slot] is None:
                return None  # Unallocated slot: key definitely does not exist
            elif self.keys[slot] == key:
                return self.values[slot]

        return None

    def delete(self, key: str) -> bool:
        idx = self._hash(key)

        for i in range(self.capacity):
            slot = (idx + i) % self.capacity

            if self.keys[slot] is None:
                return False  # Not found
            elif self.keys[slot] == key:
                self.keys[slot] = self.TOMBSTONE
                self.values[slot] = None
                return True

        return False


# --- Test Execution ---
table = LinearProbingHashTable(7)

# 'A' (65 % 7 = 2) and 'H' (72 % 7 = 2) collide!
table.insert("A", 100)  # Lands at Slot 2
table.insert("H", 200)  # Probes to Slot 3

assert table.get("A") == 100
assert table.get("H") == 200

# Delete 'A' -> Slot 2 becomes TOMBSTONE
assert table.delete("A") is True

# Critical test: 'H' MUST still be findable at Slot 3!
assert table.get("H") == 200

# Insert 'O' (79 % 7 = 2) -> Should recycle Slot 2!
table.insert("O", 300)
assert table.get("O") == 300
assert table.keys[2] == "O"

print("Linear Probing with Tombstone resolution executed successfully!")
print("Slots:", [k if k is not table.TOMBSTONE else "DELETED" for k in table.keys])
```
```text
Linear Probing with Tombstone resolution executed successfully!
Slots: [None, None, 'O', 'H', None, None, None]
```

---

## Common Mistakes

- **Clearing slots to `None` upon deletion in Open Addressing:** As proved above, setting an open-addressing slot to `None` breaks the probe chain for all subsequent colliding keys, rendering them permanently inaccessible. Always use a Tombstone sentinel.
- **Ignoring the Load Factor in Open Addressing:** If an open-addressing table approaches $\alpha \approx 0.9$ or $1.0$, insertion times skyrocket exponentially, and an infinite loop will occur if inserting into a 100% full table. You must resize the table when $\alpha \ge 0.7$.
- **Believing collisions can be 100% prevented:** By the Pigeonhole Principle and Birthday Paradox, collisions are a physical certainty. Never write code that assumes collisions will not occur.
- **Using a zero step size in Double Hashing:** If $h_2(k) == 0$, the probing formula $h_1(k) + i \cdot 0$ probes the exact same slot repeatedly in an infinite loop! Always ensure $h_2(k) \ge 1$.

---

## Key Takeaway

A **Hash Collision** occurs when two distinct keys yield the same index. They are guaranteed by the **Pigeonhole Principle** and occur early due to the **Birthday Paradox**. 
- **Separate Chaining** stores collisions in external linked lists or trees, supporting load factors $\alpha > 1.0$ at the cost of pointer memory and cache misses.
- **Open Addressing** stores all data inside the array, using **Linear Probing**, **Quadratic Probing**, or **Double Hashing** to search for open slots. It provides superior cache performance but requires strict load factor limits ($\alpha \le 0.7$) and **Tombstone markers** for safe deletions.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Hash Collision** | The event where two distinct keys produce the exact same hash table index. |
| **Pigeonhole Principle** | A mathematical rule stating that if $N$ items are put into $m$ slots where $N > m$, at least one slot must hold multiple items. |
| **Birthday Paradox** | The probability phenomenon showing that collisions occur far sooner than intuitive uniform expectations. |
| **Load Factor ($\alpha$)** | The ratio of stored items to available slots ($\alpha = N / m$). |
| **Separate Chaining** | Resolving collisions by storing colliding elements in linked lists or trees attached to each bucket. |
| **Open Addressing** | Resolving collisions by probing alternative array slots within the main table. |
| **Linear Probing** | An open addressing strategy that checks sequential adjacent slots ($+1, +2, +3 \dots$). |
| **Primary Clustering** | The tendency of linear probing to create long contiguous blocks of occupied slots that worsen collision chains. |
| **Double Hashing** | An open addressing strategy where a second hash function calculates the step size between probes. |
| **Tombstone** | A sentinel value placed in a deleted open-addressing slot to prevent probe chains from terminating early. |

---

## References

- [Introduction to Algorithms (CLRS): Chapter 11.2 - Hash Tables & Collision Resolution](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)
- [GeeksforGeeks: Collision Resolution Techniques](https://www.geeksforgeeks.org/collision-resolution-techniques-in-hashing/)
- [OpenDSA: Hash Table Collision Resolution](https://opendsa-server.cs.vt.edu/ODSA/Books/CS3/html/HashCComb.html)
