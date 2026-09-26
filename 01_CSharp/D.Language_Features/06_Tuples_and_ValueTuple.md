# 🎁 Tuples and ValueTuple

## 📌 What is it?

A **Tuple** groups multiple values together into a single object **without** creating a formal class — useful for returning or passing around a small, fixed set of related values.

```csharp
(string Name, int Age) person = ("Rohit", 25);
Console.WriteLine(person.Name);  // Rohit
Console.WriteLine(person.Age);   // 25
```

## 🤔 Why do we need it?

The most common use case: a method needs to return **more than one value**. Before tuples, your options were:

- Use `out` parameters (clunky syntax).
- Create a whole custom class just to hold 2-3 return values.

`ValueTuple` gives you a lightweight, built-in way to return multiple values cleanly.

```csharp
// Old way — needs a class just to return two values
class MinMaxResult { public int Min; public int Max; }

// With ValueTuple — no class needed
(int Min, int Max) GetMinMax(int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}

var result = GetMinMax(new[] { 3, 7, 1, 9 });
Console.WriteLine($"{result.Min} - {result.Max}");  // 1 - 9
```

## 📊 Tuple (class, old — .NET 4.0) vs ValueTuple (struct, modern — C# 7+)

| Aspect                 | `Tuple<T1,T2,...>` (old)     | `ValueTuple` (modern) ⭐                    |
| ---------------------- | ------------------------------ | --------------------------------------------- |
| **Type**         | Reference type (class)         | Value type (struct)                           |
| **Performance**  | Heap allocation — slower      | Stack allocation — faster, less GC pressure  |
| **Member names** | Generic:`.Item1`, `.Item2` | **Named** elements: `.Name`, `.Age` |
| **Mutability**   | Immutable                      | Mutable (fields can be reassigned)            |
| **Syntax**       | `Tuple.Create("Rohit", 25)`  | `("Rohit", 25)` — much simpler             |

> 💡 In modern C#, always prefer **`ValueTuple`** (the `(...)` syntax) — `Tuple<>` is essentially legacy at this point.

## ⚙️ Named Elements — Big Readability Win

```csharp
// Without names — unclear what Item1/Item2 mean
var result = (10, 20);
Console.WriteLine(result.Item1);  // what is this? unclear

// With names — self-documenting
var result = (Width: 10, Height: 20);
Console.WriteLine(result.Width);  // clear!
```

## 💻 Deconstruction — Unpacking a Tuple

```csharp
(string name, int age) = GetPerson();
Console.WriteLine(name);
Console.WriteLine(age);

// Discard a value you don't need with "_"
(string name, _) = GetPerson();
```

**Deconstructing a custom class** (define your own `Deconstruct` method):

```csharp
public class Point
{
    public int X { get; }
    public int Y { get; }
    public Point(int x, int y) { X = x; Y = y; }

    public void Deconstruct(out int x, out int y)
    {
        x = X;
        y = Y;
    }
}

var p = new Point(3, 4);
var (x, y) = p;   // works because of the custom Deconstruct method
```

## 📊 Tuple vs Anonymous Type vs Record — When to Use What

| Need                                                                                       | Use                                                      |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| Quick multi-value return from a method, local use                                          | **ValueTuple**                                     |
| Quick LINQ projection, purely local                                                        | **Anonymous Type** (see `05_Anonymous_Types.md`) |
| Reusable, named data model with identity/equality semantics, crosses method/API boundaries | **Record** (see `07_Records.md`)                 |

## 🚨 Common Mistakes

- ❌ Using tuples for data that's used **widely across the codebase** — if the same shape of data appears in many places, promote it to a proper named class/record instead (tuples with unnamed/inconsistent element names hurt readability at scale).
- ❌ Forgetting `ValueTuple` is a **mutable struct** — passing it around by value means copies, and mutating a copy won't affect the original.
- ❌ Overusing tuples with 4+ elements — becomes hard to read; consider a proper type at that point.
- ❌ Mixing named and unnamed tuple elements inconsistently across a codebase — pick a convention.

## 💡 Best Practices

- Use **named tuple elements** always (`(string Name, int Age)`) — never rely on `.Item1`/`.Item2` in new code.
- Use tuples for **quick, local, small (2-3 element)** multi-value returns.
- Use **deconstruction** to unpack tuples cleanly at the call site.
- If a tuple shape starts getting reused in multiple places, that's a signal to promote it to a `record`.

## 🎤 Interview Questions

1. What's the difference between `Tuple<T>` and `ValueTuple`, and why is `ValueTuple` generally preferred?
2. Why is `ValueTuple` a struct instead of a class, and what performance implication does that have?
3. How does tuple deconstruction work, and how would you make a custom class support it?
4. When would you choose a tuple over a record for a method's return type?

## 📝 30-second Revision Cheat Sheet

- Tuple = group multiple values without a formal class: `(string Name, int Age)`.
- Prefer modern **`ValueTuple`** (struct, named elements) over legacy `Tuple<>` (class, `.Item1`/`.Item2`).
- Use **deconstruction** to unpack: `var (name, age) = GetPerson();`
- Best for: quick local multi-value returns.
- If reused widely across the codebase → promote to a `record` instead.
