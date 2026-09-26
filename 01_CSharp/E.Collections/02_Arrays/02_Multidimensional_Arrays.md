# 🧮 Multidimensional Arrays

## 📌 What is it?

A **Multidimensional Array** is a single array with **more than one dimension** — most commonly 2D (rows/columns, like a grid or matrix), but C# supports any number of dimensions.

```csharp
int[,] grid = new int[2, 3];   // 2 rows, 3 columns — ONE rectangular array
grid[0, 0] = 1;
grid[1, 2] = 6;
```

This is called a **rectangular array** in C# — note the single `[,]` syntax, as opposed to a **jagged array** (`[][]`, covered in `03_Jagged_Arrays.md`), which is a different structure entirely.

## 🤔 Why do we need it?

Many real-world problems are naturally **grid-shaped** — a chessboard, a spreadsheet, pixel data in an image, a matrix in linear algebra. A multidimensional array models this directly, rather than manually simulating rows/columns using flat 1D array math.

## 🌍 Real-world analogy

A **spreadsheet** 📊 — cells are addressed by row AND column together (`B3`, `C7`), not by one single flat position. `grid[row, col]` mirrors this exact addressing scheme.

## ⚙️ 2D Array — Declaration & Initialization

```csharp
// Declare a 2x3 grid (2 rows, 3 columns), default values (0)
int[,] grid = new int[2, 3];

// Declare and initialize together
int[,] matrix = new int[,]
{
    { 1, 2, 3 },   // row 0
    { 4, 5, 6 }    // row 1
};

Console.WriteLine(matrix[1, 2]);   // 6 — row 1, column 2
```

## 🖼 Visualizing a 2D Array

```
matrix[,] =
        col0  col1  col2
row0  [  1     2     3  ]
row1  [  4     5     6  ]

matrix[0, 1] → 2
matrix[1, 0] → 4
```

## 💻 Iterating a 2D Array

```csharp
int[,] matrix = { { 1, 2, 3 }, { 4, 5, 6 } };

for (int row = 0; row < matrix.GetLength(0); row++)      // GetLength(0) = number of rows
{
    for (int col = 0; col < matrix.GetLength(1); col++)  // GetLength(1) = number of columns
    {
        Console.Write(matrix[row, col] + " ");
    }
    Console.WriteLine();
}
```

Note: `.GetLength(dimension)` is used here — **not** `.Length` (which would return the **total** element count across all dimensions: `6` for a 2×3 array) and **not** `.Count`.

## 📊 3D Arrays (and Beyond)

```csharp
int[,,] cube = new int[2, 2, 2];   // 2x2x2 — 8 total elements
cube[0, 1, 1] = 99;
```

Rarely used beyond 2D in typical business applications, but common in scientific computing, 3D graphics, and certain matrix-heavy algorithms.

## 🧠 Memory Layout

A rectangular multidimensional array is still stored as **one single contiguous block of memory** — the "2D" structure is really just a mathematical mapping over a flat block, computed internally by the runtime. This is why it's called a "rectangular" array — every row has exactly the same number of columns, forming a perfect grid with no gaps.

## 🚨 Common Mistakes

- ❌ Confusing `.Length` with `.GetLength(dimension)` — `.Length` on a multidimensional array gives the **total element count**, not the size of one specific dimension.
- ❌ Mixing up rectangular arrays (`int[,]`) with jagged arrays (`int[][]`) — they use **different syntax** and behave differently (see `03_Jagged_Arrays.md`) — this is a very common beginner confusion.
- ❌ Assuming every row can have a different length in a rectangular array — it **can't**; all rows must be the same length by definition. Use a jagged array if rows need varying lengths.
- ❌ Forgetting the initializer syntax requires matching brace structure (`{ {1,2,3}, {4,5,6} }`) — mismatched row lengths cause a compile error.

## 💡 Best Practices

- Use rectangular (`[,]`) arrays when your data is **truly grid-shaped with uniform row lengths** — e.g. a fixed-size game board, a mathematical matrix.
- Use `.GetLength(0)` / `.GetLength(1)` (not `.Length`) when iterating dimension-by-dimension.
- If rows genuinely need different lengths, reach for a **jagged array** instead — don't force irregular data into a rectangular shape.

## 🎤 Interview Questions

1. What's the difference between `array.Length` and `array.GetLength(dimension)` for a 2D array?
2. How is a rectangular multidimensional array actually stored in memory?
3. When would you choose a 2D rectangular array over a jagged array?
4. Can each row in a rectangular array have a different number of columns? Why or why not?

## 📝 30-second Revision Cheat Sheet

- Multidimensional (rectangular) array = `int[,]` — grid-shaped, all rows same length.
- Access via `array[row, col]`.
- Use `.GetLength(dim)` for per-dimension size, **not** `.Length` (total count) or `.Count`.
- Stored as one contiguous memory block internally.
- Different from jagged arrays (`int[][]`) — don't confuse the two syntaxes.
