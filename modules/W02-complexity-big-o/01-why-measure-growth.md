# Why Measure Growth?

Imagine you write a Python script to sort a list of numbers, and your friend writes a different script to do the exact same thing. You want to know whose code is "better". Your first instinct might be to grab a stopwatch (or use Python's `time` module), run both scripts, and see which one finishes first. 

While this sounds logical, **measuring raw execution time is a terrible way to judge an algorithm.** Your code might run in 0.1 seconds on a brand-new gaming PC, while your friend's code takes 2.0 seconds on an old, dusty laptop. Did you write a better algorithm, or do you just have better hardware? To evaluate algorithms fairly and scientifically, we cannot rely on time. Instead, we measure **the rate of growth**: how the number of operations increases as the size of the input gets larger.

---

## 1. The Problem with Timers

Raw time depends on too many external variables:
1. **Hardware:** A faster CPU will always execute code faster.
2. **Current Load:** If your computer is running a heavy video game in the background, your Python script will suddenly run slower.
3. **Language/Compiler:** The exact same logic written in C++ will almost always run faster than in Python because of how the languages are built.

Because of this, stating "my algorithm takes 5 seconds" is meaningless to another software engineer unless they have your exact computer in the exact same state.

---

## 2. Counting Operations Instead of Seconds

If we can't measure seconds, what do we measure? We measure **operations**. 

An operation is a basic step the computer takes, such as:
- Assigning a value to a variable (`x = 5`)
- Doing basic math (`x + y`)
- Comparing two values (`if a > b:`)

Instead of asking "How many seconds does this take?", computer scientists ask: **"How many operations does this algorithm require to finish?"**

---

## 3. The Importance of Input Size ($N$)

Counting operations only makes sense in relation to the **input size**, commonly referred to as $N$. 

If your input is a list of 5 items ($N = 5$), almost any algorithm will finish instantly. A bad algorithm and a great algorithm look exactly the same when $N$ is small. 

The true test of an algorithm is its **Rate of Growth**: how rapidly the number of operations shoots up as $N$ grows from 5, to 5,000, to 5,000,000. 

**Example:**
- **Algorithm A (Good):** The number of operations is equal to $N$. If $N=10$, it takes 10 operations. If $N=1,000,000$, it takes 1,000,000 operations.
- **Algorithm B (Bad):** The number of operations is equal to $N \times N$ ($N^2$). If $N=10$, it takes 100 operations. But if $N=1,000,000$, it takes **1,000,000,000,000 (1 trillion) operations**.

No matter how fast your hardware is, Algorithm A will eventually destroy Algorithm B as the data gets larger.

---

## Try it Yourself

**Scenario:** Below are two simple functions. Both print numbers, but they scale differently based on the input size `n`. 

**(a)** If you pass `n = 10` into both functions, how many times will the word "Hello" be printed for each?

```python
def function_one(n):
    for i in range(n):
        print("Hello")

def function_two(n):
    for i in range(n):
        for j in range(n):
            print("Hello")
```

*Answer:* 
- `function_one(10)` will print "Hello" exactly 10 times. (Growth is $N$).
- `function_two(10)` has a nested loop. For every 1 of the 10 outer loops, the inner loop runs 10 times. It will print "Hello" $10 \times 10 = 100$ times. (Growth is $N^2$).

**(b)** What if `n = 1000`?
- `function_one` prints 1,000 times.
- `function_two` prints 1,000,000 times! You can easily see how `function_two` grows significantly worse.

---

## Common Mistakes

- **Thinking a fast computer saves a bad algorithm:** A common beginner trap is thinking, "My code is slow, I just need a better processor." Hardware improvements are linear (e.g., 2x faster). Bad algorithms grow exponentially or quadratically. Hardware can never outpace a bad algorithm on a large dataset.
- **Worrying about exact operation counts:** You don't need to count exactly 43 operations vs 45 operations. We only care about the massive trends as $N$ gets huge. (We will formalize this concept in the next lesson with Big-O notation).

---

## Key Takeaway

We do not measure code efficiency with a stopwatch because time is dependent on hardware and environmental factors. Instead, we evaluate algorithms based on their **rate of growth**: how the number of required computational operations increases relative to the size of the input data ($N$). 

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Input Size ($N$)** | The amount of data an algorithm needs to process (e.g., the length of a list). |
| **Operation** | A basic step performed by a CPU, such as addition, assignment, or comparison. |
| **Rate of Growth** | A description of how the number of operations required by an algorithm scales as the input size $N$ gets infinitely larger. |

---

## References
- [Khan Academy: Asymptotic Notation](https://www.khanacademy.org/computing/computer-science/algorithms/asymptotic-notation/a/asymptotic-notation)
