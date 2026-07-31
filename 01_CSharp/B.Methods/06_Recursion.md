# 🔄 Recursion

## 📌 What is it?

> A method that **calls itself** to solve a smaller instance of the same problem, until it reaches a **base case** that stops the recursion.

---

## 🧠 Intuition

Every recursive method needs two parts:

1. **Base case** — the stopping condition (prevents infinite recursion)
2. **Recursive case** — calls itself with a smaller/simpler input, moving toward the base case

```
Factorial(4)
  = 4 * Factorial(3)
        = 3 * Factorial(2)
              = 2 * Factorial(1)
                    = 1 * Factorial(0)
                          = 1   ← BASE CASE
              = 2 * 1 = 2
        = 3 * 2 = 6
  = 4 * 6 = 24
```

---

## 🖼 Call Stack Visualization

```
┌─────────────────┐
│ Factorial(0) → 1 │  ← base case, returns first
├─────────────────┤
│ Factorial(1)      │
├─────────────────┤
│ Factorial(2)      │
├─────────────────┤
│ Factorial(3)      │
├─────────────────┤
│ Factorial(4)      │  ← original call, stays on stack until inner calls resolve
└─────────────────┘
       STACK
```

> ⚠️ Each recursive call adds a frame to the **call stack**. Too many levels of recursion (no base case, or too deep) → `StackOverflowException`.

---

## 💻 Code Examples

**Basic — Factorial:**

```csharp
int Factorial(int n)
{
    if (n == 0) return 1;         // base case
    return n * Factorial(n - 1);  // recursive case
}

Console.WriteLine(Factorial(5));  // 120
```

**Intermediate — Recursive tree traversal (common real-world use case):**

```csharp
class Category
{
    public string Name;
    public List<Category> SubCategories = new();
}

void PrintCategoryTree(Category category, int depth = 0)
{
    Console.WriteLine(new string(' ', depth * 2) + category.Name);
    foreach (var sub in category.SubCategories)
        PrintCategoryTree(sub, depth + 1);   // recursive call for nested structure
}
```

**Practical — Fibonacci (classic interview question, shows inefficiency):**

```csharp
int Fibonacci(int n)
{
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
// ⚠️ Exponential time complexity — recalculates same values repeatedly
// Fix: memoization (cache results) — covered in DSA notes
```

---

## 📊 Recursion vs Iteration

| Aspect      | Recursion                                | Iteration (loop)                               |
| ----------- | ---------------------------------------- | ---------------------------------------------- |
| Readability | Often cleaner for tree/nested structures | Better for simple linear repetition            |
| Performance | Overhead per call (stack frame)          | Generally faster, no call overhead             |
| Risk        | `StackOverflowException` if too deep   | Risk of infinite loop if condition never false |
| Best for    | Trees, graphs, divide & conquer problems | Simple counted/conditioned repetition          |

---

## 🚨 Common Mistakes

- ❌ Missing or unreachable base case → infinite recursion → `StackOverflowException`
- ❌ Using recursion for simple linear tasks where a loop is clearer and faster
- ❌ Not considering performance — naive recursive Fibonacci is exponential; use memoization or iteration for large inputs

---

## 🎤 Interview Questions

| Question                                      | Key Point                                                                                |
| --------------------------------------------- | ---------------------------------------------------------------------------------------- |
| What are the two required parts of recursion? | Base case (stopping condition) + recursive case (moves toward base case)                 |
| What happens if there's no base case?         | Infinite recursion →`StackOverflowException`                                          |
| When is recursion preferred over iteration?   | Naturally recursive/nested data structures — trees, graphs, divide & conquer algorithms |

---

## 📝 30-Second Revision Cheat Sheet

- Recursion = method calling itself | needs **base case** + **recursive case**
- Each call adds a stack frame → deep/infinite recursion = `StackOverflowException`
- Best for tree/nested structures (e.g., category trees, file systems)
- Naive recursive Fibonacci = classic example of inefficient recursion (exponential tim

# Recursion

---

## 📌 Overview

> Write your notes here.

---

## 🔑 Key Concepts

---

## 💻 Code Example

```csharp
```

---

## ❓ Interview Questions

---

## 🔗 Related Topics

---

*Last updated: 2026-03-15*
