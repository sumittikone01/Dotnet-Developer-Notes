# 🧩 Pattern Matching

## 📌 What is it?

**Pattern Matching** lets you check whether a value has a certain "shape" (type, structure, or value) and **extract data from it in one step**, instead of writing separate type-checks and manual casting.

```csharp
object obj = "Hello";

if (obj is string s)
{
    Console.WriteLine(s.Length);   // "s" is already cast and ready to use
}
```

## 🤔 Why do we need it?

Before pattern matching, checking a type and using it required two separate steps:

```csharp
// Old way
if (obj is string)
{
    string s = (string)obj;   // manual cast, redundant
    Console.WriteLine(s.Length);
}
```

Pattern matching combines the **check** and the **extraction** into a single, safer expression.

## 🌍 Real-world analogy

A **key-shaped lock** 🔑 — you don't first check "is this a key?" and then separately "does it fit?"; inserting it either matches the pattern (unlocks) or it doesn't, in one motion. Pattern matching checks shape and structure in one unified step.

## ⚙️ Pattern Matching Evolution (C# 7 → 11+)

| Pattern Type                                        | Example                                       | Introduced |
| --------------------------------------------------- | --------------------------------------------- | ---------- |
| **Type pattern**                              | `obj is string s`                           | C# 7       |
| **Constant pattern**                          | `x is 5`                                    | C# 7       |
| **Switch expression**                         | `x switch { 1 => "one", _ => "other" }`     | C# 8       |
| **Property pattern**                          | `person is { Age: > 18 }`                   | C# 8       |
| **Tuple pattern**                             | `(x, y) switch { (0, 0) => "origin", ... }` | C# 8       |
| **Relational pattern**                        | `age is >= 18 and < 65`                     | C# 9       |
| **Logical patterns (`and`/`or`/`not`)** | `x is not null`                             | C# 9       |
| **List patterns**                             | `arr is [1, 2, ..]`                         | C# 11      |

## 💻 Switch Expression (Modern, Concise)

```csharp
string GetSeason(int month) => month switch
{
    12 or 1 or 2 => "Winter",
    3 or 4 or 5  => "Spring",
    6 or 7 or 8  => "Summer",
    _            => "Autumn"
};
```

Compare to the old-style `switch` **statement**, which needed `case`, `break`, and far more lines for the same logic.

## 💻 Property Pattern — Matching on Object Shape

```csharp
public record Order(decimal Total, string Status);

string GetShippingPriority(Order order) => order switch
{
    { Status: "VIP", Total: > 1000 } => "Priority Shipping",
    { Status: "VIP" }                => "Fast Shipping",
    { Total: > 500 }                 => "Standard Plus",
    _                                => "Standard"
};
```

This checks **both the type's shape and specific property values** in one readable expression — no chained `if/else if` needed.

## 💻 Tuple Pattern — Matching Multiple Values Together

```csharp
string ClassifyPoint(int x, int y) => (x, y) switch
{
    (0, 0)          => "Origin",
    (0, _)          => "On Y-axis",
    (_, 0)          => "On X-axis",
    var (a, b) when a == b => "On diagonal",
    _               => "Somewhere else"
};
```

## 💻 List Pattern (C# 11) — Matching Array/List Shapes

```csharp
int[] numbers = { 1, 2, 3 };

string Describe(int[] arr) => arr switch
{
    []              => "Empty",
    [var only]      => $"Single: {only}",
    [var first, .., var last] => $"First: {first}, Last: {last}",
    _               => "Other"
};
```

## 📊 Logical & Relational Patterns

```csharp
bool IsValidAge(int age) => age is >= 0 and < 120;

bool IsMiddle(int score) => score is not (< 0 or > 100);
```

## 🚨 Common Mistakes

- ❌ Forgetting the discard pattern `_` in a `switch` expression — missing a default case causes a runtime exception if no pattern matches.
- ❌ Overcomplicating a switch expression with deeply nested property patterns — readability suffers past a certain complexity; consider splitting logic.
- ❌ Using old-style `is` + manual cast when a type pattern (`is string s`) does both steps more safely and concisely.
- ❌ Not realizing pattern matching **works great with records** — records' positional deconstruction pairs naturally with pattern matching (`order is Order(> 1000, "VIP")`).

## 💡 Best Practices

- Prefer `switch` **expressions** over old `switch` **statements** for anything that just returns a value.
- Use **property patterns** to replace long `if/else` chains checking multiple conditions on an object.
- Combine pattern matching with `record` types for very expressive, declarative code (see `07_Records.md`).
- Always include a `_` discard case unless you're deliberately certain all cases are covered (e.g. matching over a closed set like an `enum`).

## 🎤 Interview Questions

1. What's the difference between a `switch` statement and a `switch` expression?
2. How does a property pattern let you match on an object's shape, and how is it more concise than nested `if` statements?
3. What happens if a `switch` expression doesn't have a matching pattern and no discard (`_`) case?
4. How do pattern matching and records complement each other in modern C#?

## 📝 30-second Revision Cheat Sheet

- Pattern Matching = check shape/type/value AND extract data, in one step.
- `obj is string s` → type pattern, combines check + cast.
- `switch` **expression** (not statement) is the modern, concise way to branch on patterns.
- Property patterns (`{ Status: "VIP" }`) replace long `if/else` chains.
- Pairs naturally with `record` types for expressive, declarative code.
