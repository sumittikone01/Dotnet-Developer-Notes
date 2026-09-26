# 🗂️ Collection Interfaces

## 📌 What is it?

.NET's collection types are built around a set of **interfaces** that define common contracts — what operations a collection supports (adding, counting, iterating, indexing) — regardless of the concrete type actually used (`List<T>`, `Dictionary<K,V>`, `HashSet<T>`, etc.).

## 🤔 Why do we need it?

Interfaces let you write code that works with **any** collection satisfying a contract, without caring about the specific implementation:

```csharp
void PrintAll(IEnumerable<string> items)
{
    foreach (var item in items) Console.WriteLine(item);
}

PrintAll(new List<string> { "a", "b" });     // works
PrintAll(new HashSet<string> { "c", "d" });  // also works — same method, different collection
```

This is the foundation of **polymorphism for data structures** — write once, use with any compatible collection.

## 🌍 Real-world analogy

A **universal power socket adapter** 🔌. It doesn't matter what specific device you plug in (phone charger, laptop, hairdryer) — as long as it fits the standard plug shape (the interface), it works. Collection interfaces are that standard shape for data structures.

## 🖼 The Interface Hierarchy

```
IEnumerable<T>          ← most basic: "can be iterated over" (foreach)
      │
      ▼
ICollection<T>          ← adds: Count, Add, Remove, Contains
      │
      ▼
IList<T>                ← adds: indexer access (list[i]), Insert, RemoveAt
      │
      ▼
   List<T>               ← concrete implementation


IDictionary<TKey,TValue> ← key-value pair access, extends ICollection<KeyValuePair<K,V>>
      │
      ▼
Dictionary<TKey,TValue>   ← concrete implementation
```

## 📊 Core Interfaces At a Glance

| Interface                  | Adds                      | Example capability                                     |
| -------------------------- | ------------------------- | ------------------------------------------------------ |
| `IEnumerable<T>`         | Iteration only            | `foreach (var x in collection)`                      |
| `ICollection<T>`         | Size + mutation           | `.Count`, `.Add()`, `.Remove()`, `.Contains()` |
| `IList<T>`               | Indexed access            | `list[2]`, `.Insert(index, item)`                  |
| `IDictionary<K,V>`       | Key-based lookup          | `dict["key"]`, `.ContainsKey()`                    |
| `IReadOnlyCollection<T>` | Read-only count/iteration | Exposes size without allowing mutation                 |
| `IReadOnlyList<T>`       | Read-only indexed access  | Exposes`list[i]` without allowing mutation           |

## 💻 Why This Matters: Method Signature Design

```csharp
// ❌ Too restrictive — forces caller to use List<T> specifically
void Process(List<int> numbers) { ... }

// ✅ Better — accepts anything enumerable: List, array, HashSet, LINQ result, etc.
void Process(IEnumerable<int> numbers) { ... }

// ✅ Good when you need Count/indexing but not mutation, and want to signal "don't modify this"
void Process(IReadOnlyList<int> numbers) { ... }
```

**Rule of thumb**: accept the **most general interface** that gives you the capabilities you actually need. This makes your method flexible and reusable with any compatible collection type.

## 🧠 `IEnumerable<T>` and Deferred Execution (connects to LINQ)

`IEnumerable<T>` is also the foundation of **LINQ** (see `F.LINQ/01_LINQ_Overview.md`) — every LINQ method (`.Where()`, `.Select()`, etc.) is an extension method on `IEnumerable<T>`, which is why LINQ works uniformly across arrays, lists, sets, and more.

## 🚨 Common Mistakes

- ❌ Accepting `List<T>` as a parameter type when `IEnumerable<T>` or `IReadOnlyList<T>` would be flexible enough — unnecessarily locks callers into one specific collection type.
- ❌ Returning a mutable interface (`IList<T>`) from a public API when you meant to expose read-only data — callers can then mutate your internal collection unexpectedly. Use `IReadOnlyList<T>` / `IReadOnlyCollection<T>` for exposed data you don't want changed externally.
- ❌ Assuming all `IEnumerable<T>` sources support indexing (`[i]`) — only `IList<T>`/`IReadOnlyList<T>` guarantee that.

## 💡 Best Practices

- **Accept broad interfaces, return specific types** (or read-only interfaces) — a well-known API design principle: "be liberal in what you accept, specific in what you return."
- Use `IReadOnlyList<T>` / `IReadOnlyCollection<T>` for return types when callers shouldn't mutate the collection you're exposing.
- Reach for `IEnumerable<T>` for simple iteration-only needs — it's the most flexible, works with lazy/streamed data too.

## 🎤 Interview Questions

1. What's the difference between `IEnumerable<T>` and `ICollection<T>`?
2. Why should method parameters generally use interfaces rather than concrete collection types?
3. What's the risk of returning `IList<T>` from a public API instead of `IReadOnlyList<T>`?
4. How does `IEnumerable<T>` relate to how LINQ works across different collection types?

## 📝 30-second Revision Cheat Sheet

- Interfaces define **contracts**: `IEnumerable<T>` (iterate) → `ICollection<T>` (+ size/mutate) → `IList<T>` (+ indexing).
- `IDictionary<K,V>` for key-value lookup structures.
- Accept the **most general interface** needed in method parameters; use **read-only interfaces** for exposed return data.
- `IEnumerable<T>` is the foundation LINQ is built on.
