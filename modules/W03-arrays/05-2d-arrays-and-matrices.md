# 2D Arrays and Matrices

So far, we have only worked with flat, 1-dimensional arrays. However, many real-world engineering problems—like building a chessboard, processing pixel grids in images, or writing an Excel clone—require data to be organized in a grid of rows and columns. 

In computer science, we represent this grid using a **2D Array** (often called a **Matrix**).

---

## 1. What is a 2D Array?

A 2D Array is simply an array where every element inside of it is *another array*. 

In Python, we create this by writing a list of lists.
```python
grid = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```
Even though we typed it out nicely across multiple lines to look like a box, Python just sees a list containing three inner lists.

---

## 2. Accessing Elements (Row, then Column)

To read or change a value inside a 2D array, you must use two sets of brackets: `matrix[row][col]`.
- The first bracket selects the **Row** (the inner list).
- The second bracket selects the **Column** (the specific item inside that inner list).

*Remember: In computer science, everything is 0-indexed!*

```python
# From the grid above:
print(grid[0][0])  # Prints 1 (Top-Left corner)
print(grid[1][2])  # Prints 6 (Middle row, far right)
print(grid[2][1])  # Prints 8 (Bottom row, middle)
```

---

## 3. Finding the Dimensions

When given a matrix in an interview, you often need to know exactly how big it is before you start looping.

- **Rows:** The number of rows is just the length of the outer list: `len(grid)`
- **Columns:** The number of columns is the length of the first inner list: `len(grid[0])`

*(Note: We assume the matrix is a perfect rectangle, meaning all rows have the exact same number of columns).*

---

## 4. Traversing a Matrix

To visit every single item in a 1D array, we use a single `for` loop. To visit every single item in a 2D array, we must use a **Nested Loop**.

The outer loop goes down row by row. The inner loop goes across column by column.

```python
def print_all(matrix):
    rows = len(matrix)
    cols = len(matrix[0])
    
    for r in range(rows):           # Outer loop: Rows
        for c in range(cols):       # Inner loop: Columns
            print(matrix[r][c])
```

**Time Complexity:** If the matrix is a perfect square of size $N \times N$, this nested loop takes $O(N^2)$ Quadratic Time. However, if the matrix is a rectangle with $R$ rows and $C$ columns, the time complexity is accurately stated as **$O(R \times C)$**.

---

## Try it Yourself

**Scenario:** You are building a Tic-Tac-Toe game. You want to write a function that checks if the middle square is currently taken by "X".

*Your Spec:*
1. Create a 3x3 matrix representing a game board.
2. The middle row is index 1.
3. The middle column is index 1.
4. Write an `assert` statement that checks `board[1][1]`.

```python
# 1. Create the board
board = [
    ["O", "Empty", "X"],
    ["Empty", "X", "Empty"],
    ["O", "Empty", "O"]
]

# 2. Check the middle square
assert board[1][1] == "X"
print("The middle square is taken by X!")
```
```text
The middle square is taken by X!
```

---

## Common Mistakes

- **Confusing Matrix Indices with Math Coordinates:** In traditional mathematics graphing (Cartesian planes), you write $(x, y)$ where $x$ is horizontal and $y$ is vertical. **Matrices are the exact opposite!** `matrix[r][c]` means the first number is vertical (the row), and the second number is horizontal (the column). This trips up many beginners.
- **Out of Bounds Errors (`IndexError`):** Because indices start at 0, the last row is `len(matrix) - 1`. If you try to access `matrix[len(matrix)][0]`, your program will instantly crash.

---

## Key Takeaway

A 2D Array (Matrix) is an array of arrays used to represent grids. You access elements using `matrix[row][column]`, which requires careful attention because the vertical Row is always selected *before* the horizontal Column. To process all elements in a matrix, you use a nested loop which runs in $O(R \times C)$ time.

---

## Quick Glossary

| Term | Meaning |
|---|---|
| **2D Array / Matrix** | A data structure consisting of an array of arrays, representing a grid of rows and columns. |
| **Row** | A horizontal sequence of items in a matrix (selected by the first index). |
| **Column** | A vertical sequence of items in a matrix (selected by the second index). |
| **IndexError** | An error thrown by Python when you try to access a row or column index that does not exist. |

---

## References
- [GeeksforGeeks: Multidimensional Lists in Python](https://www.geeksforgeeks.org/multi-dimensional-lists-in-python/)
