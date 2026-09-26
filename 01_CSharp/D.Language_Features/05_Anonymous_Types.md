# 🎭 Anonymous Types

## 📌 What is it?

An **Anonymous Type** is a simple, unnamed class that the compiler generates automatically, based on a set of properties you specify inline — used when you need a quick, throwaway data container without formally declaring a class.

```csharp
var person = new { Name = "Rohit", Age = 25 };
Console.WriteLine(person.Name);   // Rohit
```

There's no `Person` class anywhere in your code — the compiler invents one behind the scenes, just for this.

## 🤔 Why do we need it?

Sometimes you need to bundle a few values together **temporarily** — for a LINQ projection, a quick return value, or test data — and creating a whole formal class just for that one use feels like overkill.

```csharp
// Without anonymous type — need a whole separate class just for this
class EmployeeSummary { public string Name; public int YearsWorked; }

// With anonymous type — no class declaration needed
var summary = new { Name = "Priya", YearsWorked = 3 };
```

## 🌍 Real-world analogy

A **sticky note** 📝 you jot down for a quick, one-time reminder — you're not going to design and print a formal labeled form just to write "pick up milk." Anonymous types are the sticky-note equivalent of a class.

## ⚙️ Key Characteristics

| Property                 | Behavior                                                                                                        |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- |
| **Type name**      | Compiler-generated, hidden — you can't name it explicitly                                                      |
| **Properties**     | Read-only (get-only) — set once, at creation                                                                   |
| **Type inference** | Must be assigned to`var`                                                                                      |
| **Equality**       | Two anonymous type instances are equal if all their properties are equal (value-based equality, auto-generated) |
| **Scope**          | Best suited for local, short-lived use (can't be a method's return type or public API — see 🚨 below)          |

## 💻 Real-World Usage — LINQ Projections (the #1 use case)

```csharp
var employees = GetEmployees();

var summary = employees.Select(e => new
{
    e.Name,
    IsSenior = e.YearsOfExperience > 5
});

foreach (var s in summary)
    Console.WriteLine($"{s.Name} - Senior: {s.IsSenior}");
```

This is by far the most common real-world use: **projecting only the fields you need**, without creating a formal class for a query result you'll use once and discard.

## 📊 Value-Based Equality Example

```csharp
var a = new { Name = "Rohit", Age = 25 };
var b = new { Name = "Rohit", Age = 25 };

Console.WriteLine(a == b);        // false — reference types, no operator overload
Console.WriteLine(a.Equals(b));   // true  — auto-generated Equals() compares property values
```

## 🚨 Common Mistakes / Limitations

- ❌ Trying to return an anonymous type from a public method — **the calling code has no way to name/reference the type**, so this only works within the same method/local scope (or via `dynamic`, which loses compile-time safety).
- ❌ Trying to modify a property after creation — anonymous type properties are **read-only**; you must create a new instance instead.
- ❌ Using anonymous types for data that needs to cross method/API boundaries — use a proper named class, a `record` (see `07_Records.md`), or a tuple (see `06_Tuples_and_ValueTuple.md`) instead.
- ❌ Confusing anonymous types with `dynamic` — anonymous types are still **strongly typed** at compile time (via `var` inference); `dynamic` bypasses compile-time type checking entirely.

## 💡 Best Practices

- Use only for **short-lived, local, throwaway** data groupings — especially LINQ projections.
- If the same shape of data is used in multiple places, or crosses a method boundary, promote it to a proper named class or a `record`.
- Don't rely on anonymous types for anything that needs to be serialized/passed across process boundaries.

## 🎤 Interview Questions

1. Why must anonymous type variables be declared with `var`?
2. Can you return an anonymous type from a method? Why or why not?
3. How does equality comparison work for anonymous types?
4. When would you choose an anonymous type over a `record` or a tuple?

## 📝 30-second Revision Cheat Sheet

- Anonymous type = compiler-generated, unnamed class: `new { Name = "X", Age = 25 }`.
- Must use `var` — properties are **read-only**.
- Equality is **value-based**, auto-generated.
- Best use case: **LINQ projections** for quick, local, throwaway data shapes.
- Can't cross method/API boundaries meaningfully — use a `record` or named class for that.
