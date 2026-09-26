# 📇 Records

## 📌 What is it?

A **Record** (introduced in C# 9) is a reference type designed specifically for modeling **immutable data** — it gives you value-based equality, built-in `ToString()`, and easy "modified copies" **automatically**, without writing all that boilerplate yourself.

```csharp
public record Person(string Name, int Age);

var p1 = new Person("Rohit", 25);
var p2 = new Person("Rohit", 25);

Console.WriteLine(p1 == p2);   // true! (value-based equality, unlike a normal class)
```

## 🤔 Why do we need it?

With a regular `class`, comparing two objects with identical data returns `false` unless you manually override `Equals()`, `GetHashCode()`, and `ToString()` yourself — every single time.

```csharp
// Regular class — needs manual boilerplate for value equality
class PersonClass
{
    public string Name; public int Age;
}
var a = new PersonClass { Name = "Rohit", Age = 25 };
var b = new PersonClass { Name = "Rohit", Age = 25 };
Console.WriteLine(a == b);  // false — compares REFERENCES, not values!
```

Records give you this "compare by value" behavior **for free**, since that's usually what you actually want for data-holding models (DTOs, API payloads, domain value objects).

## 🌍 Real-world analogy

Two **$10 bills** are considered "the same" in value even though they are physically different pieces of paper — you don't care *which* bill you have, only that it's worth $10. Records model this "compare by value, not identity" idea directly.

## ⚙️ Records vs Classes vs Structs

| Aspect                   | `class`                                                   | `record`                                              | `struct`                    |
| ------------------------ | ----------------------------------------------------------- | ------------------------------------------------------- | ----------------------------- |
| **Type category**  | Reference type                                              | Reference type (by default)                             | Value type                    |
| **Equality**       | Reference-based (default)                                   | **Value-based** (auto-generated)                  | Value-based                   |
| **Mutability**     | Mutable by default                                          | **Immutable by default** (with positional syntax) | Mutable by default            |
| **`ToString()`** | Manual override needed                                      | Auto-generated, readable output                         | Manual override needed        |
| **Best for**       | Behavior-focused objects (services, entities with identity) | Data-focused, immutable models                          | Small, lightweight value data |

## 💻 Two Ways to Declare a Record

**Positional syntax (concise, most common):**

```csharp
public record Person(string Name, int Age);
```

**Traditional syntax (more control, similar to a class):**

```csharp
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}
```

Both generate the same value-equality, `ToString()`, and copy behavior under the hood.

## 🧠 `with` Expressions — Non-Destructive Mutation

Since records are designed to be immutable, you don't modify them directly — you create a **modified copy**:

```csharp
var original = new Person("Rohit", 25);
var older = original with { Age = 26 };

Console.WriteLine(original.Age);  // 25 — unchanged!
Console.WriteLine(older.Age);     // 26 — new copy with the update
```

This is the record equivalent of "change one field, keep everything else the same" — extremely common when working with immutable data models.

## 📊 Auto-Generated `ToString()` Example

```csharp
var p = new Person("Rohit", 25);
Console.WriteLine(p);  
// Output: Person { Name = Rohit, Age = 25 }   ← readable, automatic!
```

Compare this to a regular class, which would print the unhelpful `Namespace.Person` (the type name) unless you manually override `ToString()`.

## 📊 `record class` vs `record struct` (C# 10+)

```csharp
public record class Person(string Name, int Age);   // reference type (default)
public record struct Point(int X, int Y);            // value type
```

`record struct` combines the value-type performance of structs with the auto-generated equality/`ToString()`/`with` benefits of records.

## 🚨 Common Mistakes

- ❌ Assuming records are always **immutable by default** — the positional syntax makes properties `init`-only (see `10_init_Only_Setters.md`), but the traditional syntax with plain `{ get; set; }` is fully mutable — immutability is a *convention* records encourage, not an absolute guarantee.
- ❌ Using records for objects with real **identity and behavior** (e.g. an `Order` entity with methods that mutate internal state over time) — records are meant for data, not behavior-heavy domain objects; use a `class` there.
- ❌ Forgetting that `with` creates a **shallow copy** — nested reference-type properties are shared between the original and the copy, not deep-cloned.

## 💡 Best Practices

- Use records for **DTOs, API request/response models, and immutable domain value objects**.
- Prefer positional syntax for simple data shapes — it's the most concise.
- Use `with` expressions instead of manually constructing a new instance field-by-field.
- Use `record struct` when the data is small and you want value-type (stack) performance.

## 🎤 Interview Questions

1. What's the core difference in default equality behavior between a `class` and a `record`?
2. What does a `with` expression do, and why is it useful for immutable data?
3. When would you choose a `record struct` over a `record class`?
4. Are records always fully immutable? Explain the nuance.

## 📝 30-second Revision Cheat Sheet

- Record = reference type built for immutable data — auto value-equality, `ToString()`, and `with` copying.
- `record Person(string Name, int Age);` — positional syntax, most common.
- `with` expression → creates a modified **copy**, doesn't mutate the original.
- `record struct` = value-type version (C# 10+).
- Use for DTOs/data models; use `class` for behavior-heavy objects with identity.
