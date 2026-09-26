# λ Lambda Expressions

## 📌 What is it?

A **Lambda Expression** is an anonymous, inline function — a shorthand way to write a small piece of executable code without formally declaring a named method.

```csharp
x => x * 2
```

Read as: *"take `x`, return `x * 2`."*

## 🤔 Why do we need it?

Before lambdas, passing behavior as data required writing a whole separate named method + wrapping it in a delegate (see `01_Delegates.md`). Lambdas let you write that logic **inline, right where you use it** — shorter, more readable, and keeps related logic close together.

```csharp
// Without lambda (old style)
bool IsEven(int x) => x % 2 == 0;
var evens = numbers.Where(IsEven);

// With lambda — no separate method needed
var evens = numbers.Where(x => x % 2 == 0);
```

## 🌍 Real-world analogy

Instead of writing someone a formal letter with a name and address every time you want to give a quick instruction ("please water the plants"), you just say it out loud on the spot. Lambdas are that spoken, disposable instruction — no formal paperwork (a named method) required.

## ⚙️ Syntax Forms

| Form                          | Example                               | When to use                                                 |
| ----------------------------- | ------------------------------------- | ----------------------------------------------------------- |
| **Expression lambda**   | `x => x * 2`                        | Single expression, implicit return                          |
| **Statement lambda**    | `x => { var y = x * 2; return y; }` | Multiple statements, needs`{ }` and explicit `return`   |
| **No parameters**       | `() => Console.WriteLine("Hi")`     | Action with no input                                        |
| **Multiple parameters** | `(x, y) => x + y`                   | Two or more inputs                                          |
| **Typed parameters**    | `(int x, int y) => x + y`           | Explicit typing (rarely needed — type is usually inferred) |

## 🧠 Lambdas Are Just Syntax Sugar Over Delegates

Under the hood, a lambda is compiled into a method and wrapped in a delegate — most commonly `Func<>` (returns a value) or `Action<>` (returns nothing).

```csharp
Func<int, int> square = x => x * 2;
Console.WriteLine(square(5));   // 10

Action<string> greet = name => Console.WriteLine($"Hello, {name}");
greet("Rohit");                 // Hello, Rohit

Predicate<int> isEven = x => x % 2 == 0;   // returns bool specifically
```

| Delegate Type        | Signature                        | Use case                   |
| -------------------- | -------------------------------- | -------------------------- |
| `Action<T>`        | Takes params, returns`void`    | Fire-and-forget logic      |
| `Func<T, TResult>` | Takes params, returns a value    | Transform/compute logic    |
| `Predicate<T>`     | Takes one param, returns`bool` | Filtering/condition checks |

## 💻 Real-World Usage — LINQ (the #1 use case)

```csharp
var employees = GetEmployees();

var seniorDevs = employees
    .Where(e => e.YearsOfExperience > 5)          // filter
    .Select(e => e.Name)                          // project
    .OrderBy(name => name)                        // sort
    .ToList();
```

Every one of `.Where()`, `.Select()`, `.OrderBy()` accepts a lambda — this is *the* everyday use case you'll see constantly in C#/LINQ code.

## 🧠 Closures — Lambdas Capture Outer Variables

A lambda can **capture** (remember) variables from its surrounding scope — this is called a **closure**.

```csharp
int threshold = 10;
Func<int, bool> isAboveThreshold = x => x > threshold;

Console.WriteLine(isAboveThreshold(15)); // true

threshold = 20; // change the outer variable
Console.WriteLine(isAboveThreshold(15)); // false — captured by REFERENCE, sees the update!
```

⚠️ The lambda doesn't capture the *value* of `threshold` at creation time — it captures the *variable itself*. If the outer variable changes later, the lambda sees the new value.

## 🚨 Common Mistakes

- ❌ **Closure-over-loop-variable bug** (mostly historical, fixed in modern C#, but still a classic interview trap):

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    actions.Add(() => Console.WriteLine(i));
}
foreach (var a in actions) a();
// C# 5+ : prints 0, 1, 2 (loop variable is now scoped per-iteration) ✅
// Pre-C# 5 : printed 3, 3, 3 (all lambdas shared the same variable) ❌
```

- ❌ Writing overly complex logic inside a lambda — hurts readability; extract to a named method if it grows beyond a line or two.
- ❌ Capturing large objects unnecessarily in a lambda that outlives them, causing memory to be held longer than expected.

## 💡 Best Practices

- Keep lambdas **short and simple** — one line ideally. If it needs multiple statements and gets complex, use a named method instead.
- Understand that lambdas capture variables **by reference**, not by value — be deliberate when using loop variables or mutable outer state.
- Prefer lambdas for **LINQ, event handlers, and short callbacks** — this is where they shine.

## 🎤 Interview Questions

1. What's the difference between a lambda expression and a regular method?
2. What is a closure, and does a C# lambda capture variables by value or by reference?
3. What's the difference between `Action<T>`, `Func<T, TResult>`, and `Predicate<T>`?
4. Why did the classic "loop variable capture" bug happen pre-C# 5, and how was it fixed?

## 📝 30-second Revision Cheat Sheet

- Lambda = anonymous inline function: `x => x * 2`.
- Compiles down to a delegate — usually `Func<>`, `Action<>`, or `Predicate<>`.
- Captures outer variables by **reference** (closures).
- Most common real-world use: **LINQ** (`Where`, `Select`, `OrderBy`, etc.).
- Keep them short — extract to a named method if logic grows complex.
