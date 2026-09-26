# 🧬 Generics Overview

## 📌 What is it?

**Generics** let you write a class, method, or interface that works with **any type**, decided at the point of use — instead of writing separate, near-identical code for every specific type (`int`, `string`, `Employee`, etc.), you write it **once** with a type placeholder.

```csharp
public class Box<T>
{
    public T Value { get; set; }
}

Box<int> intBox = new Box<int> { Value = 42 };
Box<string> stringBox = new Box<string> { Value = "Hello" };
```

`T` here is a **type parameter** — a placeholder that gets replaced with a real type (`int`, `string`, etc.) when the generic class is actually used.

## 🤔 Why do we need it?

Without generics, you'd need to either:

1. Write a **separate class per type** — massive code duplication.
2. Use `object` for everything — loses type safety and requires manual, error-prone casting.

```csharp
// ❌ Without generics — using object loses type safety
public class ObjectBox
{
    public object Value { get; set; }
}

var box = new ObjectBox { Value = 42 };
string text = (string)box.Value;   // 💥 InvalidCastException at RUNTIME — no compile-time safety!

// ✅ With generics — type safety enforced at COMPILE TIME
var typedBox = new Box<int> { Value = 42 };
// typedBox.Value = "text";   ❌ Compile error — caught immediately, not at runtime
```

## 🌍 Real-world analogy

A **universal shipping container** 📦 that can be configured to carry **any type of cargo** — electronics, furniture, food — but once you configure it for "electronics only," it enforces that consistently; you can't accidentally load furniture into an electronics-labeled container. Generics give you that same "flexible template, but strictly enforced once chosen" behavior.

## ⚙️ Where Generics Show Up (You've Already Been Using Them!)

Every collection covered earlier in this chapter is **already generic**:

```csharp
List<int> numbers;                          // T = int
Dictionary<string, int> ages;                // TKey = string, TValue = int
Queue<Employee> tasks;                        // T = Employee
```

Generics aren't a separate, rarely-used feature — they're the foundation of nearly every collection type you use daily in C#.

## 📊 Generics vs `object` vs Non-Generic Duplication

| Approach                          | Type Safety                             | Performance                          | Code Duplication              |
| --------------------------------- | --------------------------------------- | ------------------------------------ | ----------------------------- |
| **Separate class per type** | ✅ Full                                 | ✅ Fast                              | ❌ High — repeated code      |
| **Using `object`**        | ❌ None — runtime cast errors possible | ⚠️ Slower (boxing for value types) | ✅ None                       |
| **Generics** ⭐             | ✅ Full, compile-time checked           | ✅ Fast — no boxing for value types | ✅ None — one implementation |

Generics give you the **best of both worlds**: one implementation (no duplication) with full compile-time type safety and good performance.

## 🧠 Boxing/Unboxing — Why Generics Matter for Performance Too

```csharp
// Non-generic ArrayList (legacy, avoid) — stores everything as "object"
ArrayList list = new ArrayList();
list.Add(42);           // int gets BOXED (wrapped in an object) — extra allocation + performance cost
int x = (int)list[0];    // UNBOXED back — extra cast

// Generic List<int> — no boxing needed at all
List<int> genericList = new List<int>();
genericList.Add(42);     // stored directly as int, no boxing
int y = genericList[0];   // direct access, no cast needed
```

This is a major reason generics were introduced in C# 2.0 — avoiding the **boxing/unboxing overhead** that plagued older non-generic collections like `ArrayList` and `Hashtable`.

## 📊 What Can Be Generic?

| Construct            | Example                                                |
| -------------------- | ------------------------------------------------------ |
| **Classes**    | `class Box<T>` (see `02_Generic_Classes.md`)       |
| **Methods**    | `T Max<T>(T a, T b)` (see `03_Generic_Methods.md`) |
| **Interfaces** | `IEnumerable<T>`, `IComparable<T>`                 |
| **Delegates**  | `Func<T, TResult>`, `Action<T>`                    |

## 🚨 Common Mistakes

- ❌ Using `object` (or legacy non-generic collections like `ArrayList`) for new code — loses compile-time type safety and incurs boxing overhead for value types.
- ❌ Thinking generics are some rarely-used advanced feature — in practice, you're using generics constantly through `List<T>`, `Dictionary<K,V>`, `Func<>`, etc.
- ❌ Not realizing that a runtime `InvalidCastException` from an `object`-based approach is a bug that generics would have caught at **compile time** instead.

## 💡 Best Practices

- Prefer generic collections (`List<T>`, `Dictionary<K,V>`) over legacy non-generic ones (`ArrayList`, `Hashtable`) — always, in modern code.
- Reach for generics whenever you notice yourself writing **near-identical code** for multiple different types.
- Understand generics as fundamentally a **compile-time type safety + performance** tool, not just "syntax with angle brackets."

## 🎤 Interview Questions

1. What problem do generics solve compared to using `object` for a general-purpose container?
2. What is "boxing," and how do generics help avoid it?
3. Why is `List<T>` preferred over the legacy `ArrayList` in modern C# code?
4. Name three built-in .NET types that are already generic, which you've likely used without thinking of them as "generics."

## 📝 30-second Revision Cheat Sheet

- Generics = write code once, work with **any type**, decided at usage — `T` is a placeholder type parameter.
- Solves: code duplication (vs per-type classes) AND type safety/performance (vs `object`/`ArrayList`).
- Avoids **boxing/unboxing** overhead for value types compared to non-generic collections.
- You already use generics constantly: `List<T>`, `Dictionary<K,V>`, `Func<>`, `Action<>`.
- Can apply to classes, methods, interfaces, and delegates.
