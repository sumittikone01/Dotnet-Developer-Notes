# 🪚 Jagged Arrays

## 📌 What is it?

A **Jagged Array** is an **array of arrays** — each "row" is actually its own **independent array object**, which means each row can have a **different length**. This is different from a rectangular multidimensional array (`int[,]`), where every row must be the same length (see `02_Multidimensional_Arrays.md`).

```csharp
int[][] jagged = new int[3][];   // 3 rows, but each row's size is NOT yet defined

jagged[0] = new int[] { 1, 2 };        // row 0 has 2 elements
jagged[1] = new int[] { 3, 4, 5 };     // row 1 has 3 elements
jagged[2] = new int[] { 6 };           // row 2 has 1 element
```

## 🤔 Why do we need it?

Real-world data isn't always a perfect grid. Sometimes each "row" naturally has a **different number of items** — and forcing that into a rectangular array would waste memory (padding shorter rows with unused default values) or simply not model the data correctly.

## 🌍 Real-world analogy

A **bookshelf** 📚 where each shelf can hold a **different number of books** — shelf 1 has 5 books, shelf 2 has 2 books, shelf 3 has 8 books. Unlike a rectangular array (every shelf forced to hold exactly the same number of slots), a jagged array lets each shelf be exactly as long as it needs to be.

## 🖼 Visualizing the Difference

```
Rectangular array (int[,]) — every row MUST be the same length:
[ 1  2  3 ]
[ 4  5  6 ]

Jagged array (int[][]) — each row is its OWN independent array, any length:
[ 1  2 ]
[ 3  4  5 ]
[ 6 ]
```

## ⚙️ Key Characteristic: Each Row is a Separate Array Object

```csharp
int[][] jagged = new int[2][];
jagged[0] = new int[] { 1, 2, 3 };
jagged[1] = new int[] { 4 };

Console.WriteLine(jagged[0].Length);   // 3 — each row has its OWN .Length
Console.WriteLine(jagged[1].Length);   // 1
Console.WriteLine(jagged.Length);      // 2 — outer array = number of rows
```

Note the syntax difference from multidimensional arrays: `jagged[0][1]` (two separate brackets) instead of `matrix[0, 1]` (one bracket, comma-separated).

## 💻 Full Initialization Syntax

```csharp
int[][] jagged = new int[][]
{
    new int[] { 1, 2 },
    new int[] { 3, 4, 5 },
    new int[] { 6 }
};

foreach (int[] row in jagged)
{
    foreach (int val in row)
        Console.Write(val + " ");
    Console.WriteLine();
}
```

## 📊 Rectangular vs Jagged — Comparison

| Aspect                  | Rectangular (`int[,]`)                                 | Jagged (`int[][]`)                                                 |
| ----------------------- | -------------------------------------------------------- | -------------------------------------------------------------------- |
| **Row lengths**   | All rows must match                                      | Each row can differ                                                  |
| **Memory layout** | Single contiguous block                                  | Outer array of**references** to separate inner arrays          |
| **Syntax**        | `matrix[row, col]`                                     | `jagged[row][col]`                                                 |
| **Performance**   | Slightly faster (one contiguous block, one bounds check) | Slightly slower (extra indirection — each row is a separate object) |
| **Best for**      | True uniform grid data (matrices)                        | Variable-length row data (e.g. adjacency lists in graphs)            |

## 💻 Real-World Use Case: Graph Adjacency List

```csharp
// Each node connects to a different number of other nodes — perfect fit for jagged arrays
int[][] adjacencyList = new int[4][];
adjacencyList[0] = new int[] { 1, 2 };      // Node 0 connects to nodes 1, 2
adjacencyList[1] = new int[] { 0 };          // Node 1 connects to node 0
adjacencyList[2] = new int[] { 0, 3 };       // Node 2 connects to nodes 0, 3
adjacencyList[3] = new int[] { 2 };          // Node 3 connects to node 2
```

This is one of the most common real-world jagged array use cases — representing graphs, trees, or any inherently irregular structured data.

## 🚨 Common Mistakes

- ❌ Forgetting to initialize each inner row array before use — `jagged[0]` starts as `null` until you assign it an actual array; accessing it before that throws `NullReferenceException`.
- ❌ Confusing the syntax with rectangular arrays — `jagged[0, 1]` is **invalid** for a jagged array; must use `jagged[0][1]`.
- ❌ Assuming `.Length` on the outer array tells you anything about inner row sizes — it only tells you the **number of rows**; each row's own `.Length` must be checked separately.
- ❌ Using a jagged array when data really is uniform/rectangular — adds unnecessary indirection/complexity for no benefit.

## 💡 Best Practices

- Use jagged arrays when row lengths **genuinely vary** — e.g. graph adjacency lists, triangular data, variable-length per-item data.
- Use rectangular arrays instead when data is a **true, uniform grid** — slightly better performance and simpler mental model.
- Always initialize every inner array explicitly before accessing it, to avoid null reference errors.

## 🎤 Interview Questions

1. What's the fundamental structural difference between a jagged array and a rectangular multidimensional array?
2. Why might a jagged array be less performant than a rectangular array for the same logical data?
3. Give a real-world example where a jagged array is a more natural fit than a rectangular array.
4. What happens if you try to access `jagged[0][0]` before assigning anything to `jagged[0]`?

## 📝 30-second Revision Cheat Sheet

- Jagged array = `int[][]` — an array of independently-sized arrays; rows can have different lengths.
- Syntax: `jagged[row][col]` (two brackets) vs rectangular's `matrix[row, col]` (one bracket, comma).
- Each row must be individually initialized (`jagged[0] = new int[]{...}`) before use.
- Best for irregular data: graph adjacency lists, variable-length rows.
- Rectangular array is faster/simpler for true uniform grid data.
