# Backtracking

The final recursive pattern we will study is **Backtracking**. 

Backtracking is a specialized form of Tree Recursion used when you need to explore all possible paths to find a solution (like solving a maze or Sudoku puzzle), but you want the ability to "turn around" the exact moment you realize you went down a dead end.

---

## 1. What is Backtracking?

Imagine you are physically standing in a maze. 
1. You come to a fork with 3 paths: Left, Straight, and Right. 
2. You *choose* to go Left. 
3. After walking a bit, you hit a solid brick wall (a dead end). 
4. Do you stay there forever? No! You walk backwards to the fork, *undo* your choice to go Left, and try the Straight path instead.

This is exactly how Backtracking works in code. It builds a solution incrementally, one piece at a time. If it realizes the current path is invalid (or if it successfully finds a solution and wants to find more), it abandons that path, steps back, and tries the next available option.

---

## 2. The Three Steps

Every Backtracking algorithm follows a strict 3-step pattern inside its recursive `for` loop:

1. **Choose:** Pick a single option and add it to your `current_path` list.
2. **Explore:** Recursively call the function to continue exploring down this new path.
3. **Un-choose (Backtrack):** After returning from the recursion, remove the choice you just made from your `current_path`. This wipes the slate clean so the loop can try the *next* option on a fresh path.

**Famous Use Cases:**
- Finding all Combinations and Permutations of a string.
- The N-Queens problem (placing 8 queens on a chessboard so none can attack each other).
- Solving Sudoku.

---

## Try it Yourself

**Scenario:** Let's write the conceptual backtracking code to generate all permutations of a string. 

If given `"AB"`, the output should be `["AB", "BA"]`.

*Your Spec:*
1. **Base Case:** If our `current_path` is the same length as the input string, we have a complete word! Add it to our `results` and `return`.
2. **Loop:** Iterate through all available letters. If the letter is already in our path, `continue` (skip it).
3. **Choose:** Add the letter to `current_path`.
4. **Explore:** Call `backtrack()` recursively.
5. **Un-choose:** Remove the letter from `current_path`.

```python
def get_permutations(text):
    results = []
    
    def backtrack(current_path):
        # 1. Base Case: The word is complete
        if len(current_path) == len(text):
            results.append("".join(current_path))
            return
            
        for letter in text:
            # Prevent using the same letter twice
            if letter in current_path:
                continue
                
            # 2. CHOOSE
            current_path.append(letter)
            
            # 3. EXPLORE
            backtrack(current_path)
            
            # 4. UN-CHOOSE (Backtrack!)
            current_path.pop()
            
    backtrack([])
    return results

assert get_permutations("AB") == ["AB", "BA"]
print("Backtracking permutations successful!")
```
```text
Backtracking permutations successful!
```

---

## Common Mistakes

- **Forgetting the "Un-choose" step:** This is the defining feature of Backtracking. If you use a single list to track your path and you forget to call `.pop()` after the recursive step, your path gets polluted with dead-ends. The next time the loop runs, the old garbage choices will still be stuck in the list!
- **Appending a Reference instead of a Copy:** In Python, if you do `results.append(current_path)` inside your base case, you are appending a *reference* to the list. When you backtrack and `.pop()` items later, it will also erase the answers you already saved in `results`! Always append a copy of the list (e.g., `results.append(list(current_path))` or `results.append("".join(current_path))`).

---

## Key Takeaway

**Backtracking** is a brute-force recursive pattern used to explore all possible combinations. It relies heavily on a 3-step engine inside a loop: **Choose, Explore, Un-choose**. By exploring a path and then methodically undoing that choice (backtracking), the algorithm can exhaustively check every possible solution to a complex problem without permanently modifying the core data.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **Backtracking** | An algorithmic technique that incrementally builds solutions and abandons (backtracks) paths that fail to meet conditions. |
| **Permutation** | A specific ordering of a set of items. |
| **Un-choose** | The act of removing the most recent addition to your path, resetting the state for the next possible choice. |

---

## References
- [GeeksforGeeks: Backtracking Algorithms](https://www.geeksforgeeks.org/backtracking-algorithms/) 
