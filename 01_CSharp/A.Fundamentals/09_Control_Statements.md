# 🔀 Control Statements

## 📌 What is it?

> Statements that control the **flow of execution** — deciding which code runs, how many times, and under what conditions.

---

## 📊 Categories

| Category              | Statements                                                  |
| --------------------- | ----------------------------------------------------------- |
| **Conditional** | `if / else if / else`, `switch`                         |
| **Looping**     | `for`, `while`, `do-while`, `foreach`               |
| **Jump**        | `break`, `continue`, `return`, `goto` (rarely used) |

---

## 💻 Code Examples

**Conditional — if/else vs switch:**

```csharp
// if / else
int score = 85;
if (score >= 90)
    Console.WriteLine("A");
else if (score >= 75)
    Console.WriteLine("B");
else
    Console.WriteLine("C");

// switch expression (modern, C# 8+) — preferred for cleaner multi-branch logic
string grade = score switch
{
    >= 90 => "A",
    >= 75 => "B",
    _ => "C"
};
```

**Looping — choosing the right loop:**

```csharp
// for — when you know the exact number of iterations
for (int i = 0; i < 5; i++)
    Console.WriteLine(i);

// foreach — when iterating a collection (most common in real code)
List<string> names = new() { "Sumit", "Rahul", "Priya" };
foreach (var name in names)
    Console.WriteLine(name);

// while — condition checked BEFORE each iteration
int retries = 0;
while (retries < 3)
{
    Console.WriteLine("Attempt " + retries);
    retries++;
}

// do-while — condition checked AFTER each iteration (runs at least once)
int input;
do
{
    input = GetUserInput();
} while (input != -1);
```

**Jump statements — break/continue in practice:**

```csharp
foreach (var item in items)
{
    if (item.IsDeleted)
        continue;          // skip this item, move to next

    if (item.Id == targetId)
        break;              // stop the loop entirely

    Process(item);
}
```

---

## 📊 Loop Comparison

| Loop         | Condition checked              | Runs at least once?          | Best for                                    |
| ------------ | ------------------------------ | ---------------------------- | ------------------------------------------- |
| `for`      | Before each iteration          | ❌ No (if false initially)   | Known iteration count                       |
| `while`    | Before each iteration          | ❌ No                        | Unknown iteration count, condition-driven   |
| `do-while` | After each iteration           | ✅ Yes                       | Must run at least once (e.g., menu prompts) |
| `foreach`  | N/A (iterates full collection) | Only if collection non-empty | Iterating collections/arrays                |

---

## 🌍 Real-world usage (your stack)

```csharp
// Common in Kendo Grid server-side data shaping
foreach (var row in gridData)
{
    if (row.Quantity <= 0)
        continue;   // skip invalid rows

    row.Total = row.Quantity * row.UnitPrice;
}

// switch expression for status badges (common in MVC Views/ViewModels)
string badgeColor = order.Status switch
{
    "Pending" => "orange",
    "Delivered" => "green",
    "Cancelled" => "red",
    _ => "gray"
};
```

---

## 🚨 Common Mistakes

- ❌ Using `foreach` when you need to modify the collection while iterating → throws `InvalidOperationException` (use a `for` loop or iterate a copy instead)
- ❌ Infinite loops from forgetting to update the loop variable/condition in `while`
- ❌ Using `do-while` when `while` was intended — `do-while` always executes the body at least once, even if the condition is false from the start

---

## 🎤 Interview Questions

| Question                                                   | Key Point                                                                                                              |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Difference between`while` and `do-while`?              | `do-while` guarantees at least one execution; `while` may execute zero times                                       |
| When would you use`foreach` vs `for`?                  | `foreach` for simple collection iteration (read-only); `for` when you need the index or must modify the collection |
| What does`continue` do vs `break`?                     | `continue` skips to the next iteration; `break` exits the loop entirely                                            |
| Can you modify a`List<T>` while using `foreach` on it? | No — throws`InvalidOperationException`; use a `for` loop or `.ToList()` a copy first                            |

---

## 📝 30-Second Revision Cheat Sheet

- `if/else` → simple branching | `switch` → clean multi-branch (prefer `switch` expression for value mapping)
- `for` → known count | `while` → condition-driven | `do-while` → runs at least once | `foreach` → collection iteration
- `break` → exit loop | `continue` → skip to next iteration
- Never modify a collection while `foreach`-ing 

# Control Statements

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
