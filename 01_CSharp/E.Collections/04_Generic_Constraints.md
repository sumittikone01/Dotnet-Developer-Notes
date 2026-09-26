
# 🚧 Generic Constraints

## 📌 What is it?

A **Generic Constraint** (`where T : ...`) restricts what types can be used as the type argument `T` for a generic class or method — letting you call **specific members** on `T` inside the generic code, which wouldn't otherwise be possible if `T` could be *literally anything*.

```csharp
public T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) > 0 ? a : b;   // ✅ allowed — constraint guarantees CompareTo() exists
}
```

## 🤔 Why do we need it?

Without any constraint, the compiler has to assume `T` could be **any type whatsoever** — so it won't let you call `.CompareTo()`, access a constructor, or assume `T` is a reference/value type, because those assumptions don't hold for *every possible* type.

```csharp
// ❌ Without a constraint — compile error
public T Max<T>(T a, T b)
{
    return a.CompareTo(b) > 0 ? a : b;   // Error: 'T' has no CompareTo() — compiler can't assume this exists
}
```

Constraints are how you tell the compiler: *"T won't be just anything — it will specifically support this capability,"* unlocking the ability to use that capability inside the method/class body.

## 🌍 Real-world analogy

A **"must have a driver's license" requirement** for a car rental 🚗 — the rental company doesn't need to know *who* you are specifically, but they DO need to guarantee you have at least *this one capability* (a valid license) before handing over the keys. Constraints work the same way: you don't need to know the *exact* type `T`, just guarantee it has certain capabilities.

## 📊 Available Constraint Types

| Constraint                  | Meaning                                                      | Example                      |
| --------------------------- | ------------------------------------------------------------ | ---------------------------- |
| `where T : struct`        | `T` must be a **value type**                         | `where T : struct`         |
| `where T : class`         | `T` must be a **reference type**                     | `where T : class`          |
| `where T : new()`         | `T` must have a **public parameterless constructor** | `where T : new()`          |
| `where T : BaseClass`     | `T` must be `BaseClass` or derive from it                | `where T : Animal`         |
| `where T : InterfaceName` | `T` must implement a specific interface                    | `where T : IComparable<T>` |
| `where T : U`             | `T` must be, or derive from, another type parameter `U`  | `where T : U`              |
| `where T : notnull`       | `T` must be a non-nullable type                            | `where T : notnull`        |

## 💻 Combining Multiple Constraints

```csharp
public class Repository<T> where T : class, new()
{
    public T CreateNew() => new T();   // ✅ allowed — "new()" constraint guarantees this works
}
```

Multiple constraints on the same type parameter are comma-separated. Note: if combining `class`/`struct` with other constraints, `class`/`struct` must come **first**, and `new()` must always come **last**.

## 💻 Constraining with an Interface — The Most Common Real-World Case

```csharp
public class Repository<T> where T : IEntity   // custom interface, e.g. requires an "Id" property
{
    private List<T> _items = new List<T>();

    public T GetById(int id) => _items.FirstOrDefault(item => item.Id == id);
    // ✅ allowed — the constraint guarantees every T has an "Id" property, since IEntity requires it
}

public interface IEntity
{
    int Id { get; }
}

public class Employee : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
}

var repo = new Repository<Employee>();   // ✅ valid — Employee implements IEntity
// var badRepo = new Repository<string>(); ❌ compile error — string doesn't implement IEntity
```

## 💻 `where T : new()` — Requiring a Parameterless Constructor

```csharp
public class Factory<T> where T : new()
{
    public T CreateInstance() => new T();
}

var factory = new Factory<Employee>();   // works only if Employee has a public parameterless constructor
```

## 💻 `where T : struct` vs `where T : class`

```csharp
public class ValueBox<T> where T : struct   // only value types allowed: int, bool, custom structs, etc.
{
    public T Value;
}

public class RefBox<T> where T : class      // only reference types allowed: classes, interfaces, delegates
{
    public T Value;
}

var vb = new ValueBox<int>();     // ✅ valid
// var vb2 = new ValueBox<string>();  ❌ compile error — string is a reference type
```

## 📊 Multiple Type Parameters, Each With Their Own Constraint

```csharp
public class Pair<TKey, TValue> 
    where TKey : IComparable<TKey> 
    where TValue : class
{
    public TKey Key { get; set; }
    public TValue Value { get; set; }
}
```

Each type parameter gets its **own** `where` clause — constraints don't have to match between parameters.

## 🚨 Common Mistakes

- ❌ Trying to call a member of `T` (like `.CompareTo()`, `new T()`, or a custom interface method) without a constraint that guarantees it exists — results in a compile error, since the compiler assumes `T` could be *anything* by default.
- ❌ Forgetting constraint ordering rules: `class`/`struct` must come first, `new()` must come last, when combining multiple constraints.
- ❌ Over-constraining a generic type unnecessarily — adding constraints the method/class body doesn't actually need limits what types callers can use it with, for no real benefit.
- ❌ Confusing `where T : SomeClass` (inheritance constraint) with `where T : new()` (constructor constraint) — they serve very different purposes.

## 💡 Best Practices

- Add exactly the constraints your generic code **actually needs** — no more, no less. Over-constraining reduces flexibility unnecessarily.
- Use interface constraints (`where T : ISomeInterface`) as the most common, flexible way to guarantee specific capabilities without tying `T` to one concrete base class.
- Combine constraints thoughtfully (e.g. `where T : class, new()`) when a generic factory/repository pattern genuinely needs both.

## 🎤 Interview Questions

1. Why does the compiler reject `a.CompareTo(b)` inside an unconstrained generic method, even though many types have a `CompareTo()` method?
2. What's the difference between `where T : struct` and `where T : class`?
3. What does the `new()` constraint guarantee, and why might a generic factory method need it?
4. Can you apply different constraints to different type parameters in the same generic class? Give an example.

## 📝 30-second Revision Cheat Sheet

- Generic constraints (`where T : ...`) restrict what `T` can be, unlocking specific capabilities inside generic code.
- Common constraints: `struct`, `class`, `new()`, a base class, or an interface (most common/flexible).
- Ordering rule when combining: `class`/`struct` first, `new()` last.
- Without a constraint, the compiler assumes `T` could be *anything* — very limited operations allowed.
- Add only the constraints your code actually needs — avoid unnecessary over-constraining.
