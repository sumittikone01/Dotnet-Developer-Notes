
# 🏗️ Generic Classes

## 📌 What is it?

A **Generic Class** is a class defined with one or more **type parameters** (placeholders like `T`), letting the same class definition work with different concrete types while keeping full compile-time type safety.

```csharp
public class Box<T>
{
    private T _value;
  
    public void Set(T value) => _value = value;
    public T Get() => _value;
}

var intBox = new Box<int>();
intBox.Set(42);
Console.WriteLine(intBox.Get());   // 42, strongly typed as int

var stringBox = new Box<string>();
stringBox.Set("Hello");
```

## 🤔 Why do we need it?

Without generics, supporting multiple types would mean either duplicating the entire class per type, or using `object` and losing type safety (see `01_Generics_Overview.md` for the full comparison). A generic class lets you define the structure **once**, and the compiler generates type-safe behavior for each specific type used.

## 🌍 Real-world analogy

A **cookie cutter** 🍪 — one single cutter (the generic class template) can be used to make cookies from any dough (`int` dough, `string` dough, `Employee` dough) — the *shape* of the operation stays the same, but the *material* changes each time you use it.

## ⚙️ Multiple Type Parameters

A generic class isn't limited to one type parameter — you can define as many as needed:

```csharp
public class Pair<TFirst, TSecond>
{
    public TFirst First { get; set; }
    public TSecond Second { get; set; }
  
    public Pair(TFirst first, TSecond second)
    {
        First = first;
        Second = second;
    }
}

var pair = new Pair<string, int>("Age", 25);
Console.WriteLine($"{pair.First}: {pair.Second}");   // "Age: 25"
```

This is exactly how `Dictionary<TKey, TValue>` itself is built — two independent type parameters, each substituted separately at usage.

## 💻 A Practical Example: Generic Repository Pattern

A very common real-world use of generic classes — a reusable data-access layer that works with **any** entity type:

```csharp
public class Repository<T> where T : class
{
    private List<T> _items = new List<T>();

    public void Add(T item) => _items.Add(item);
    public T GetById(int index) => _items[index];
    public List<T> GetAll() => _items;
}

var employeeRepo = new Repository<Employee>();
employeeRepo.Add(new Employee { Name = "Rohit" });

var orderRepo = new Repository<Order>();
orderRepo.Add(new Order { Id = 1 });
```

One `Repository<T>` class definition now serves **any** entity type in the application, fully type-safe, with zero duplicated logic.

## 📊 Generic Class vs Non-Generic Class With Inheritance

You might think "why not just use a base class and inheritance instead of generics?" — here's the key difference:

| Approach                             | Type returned from methods | Casting needed?                                |
| ------------------------------------ | -------------------------- | ---------------------------------------------- |
| **Base class (`object`)**    | `object`                 | ✅ Yes — manual cast required, runtime risk   |
| **Generic class (`Box<T>`)** | The exact type`T`        | ❌ No — compiler already knows the exact type |

```csharp
// Base class approach — loses specific type info
public class ObjectBox { public object Value; }
var box = new ObjectBox { Value = 42 };
int x = (int)box.Value;   // manual cast — risk of runtime InvalidCastException

// Generic approach — type flows through automatically
var genericBox = new Box<int>();
genericBox.Set(42);
int y = genericBox.Get();   // already an int — no cast needed at all
```

## 🧠 Default Values with `default(T)` / `default`

```csharp
public class Box<T>
{
    private T _value = default;   // works for ANY type T
    // For int → 0, for string → null, for bool → false, etc.
}
```

`default` (or `default(T)`) gives you the type's default value regardless of what `T` ends up being — essential since you can't hardcode `0` or `null` in a class body that must work for any type.

## 🚨 Common Mistakes

- ❌ Forgetting `default(T)` (or the shorthand `default`) when you need a placeholder/initial value that must work for **any** type `T` — you can't just write `null` if `T` might be a value type like `int`.
- ❌ Overcomplicating a class with generics when it's genuinely only ever going to be used with one specific type — adds unnecessary complexity for no real benefit.
- ❌ Not applying **constraints** (see `04_Generic_Constraints.md`) when the generic class body actually needs to call specific methods on `T` (e.g. comparing, or requiring a parameterless constructor) — without constraints, the compiler only allows operations valid for *any* possible type.

## 💡 Best Practices

- Use generic classes when the same **structural logic** needs to apply across multiple, otherwise-unrelated types — repositories, containers, wrappers, caches.
- Use multiple type parameters (`<TFirst, TSecond>`) when a class genuinely needs to relate two independent types together.
- Add constraints (`where T : ...`) whenever the class body needs to do more than just store/retrieve `T` generically — see the next note for full detail.

## 🎤 Interview Questions

1. What's the practical benefit of a generic `Repository<T>` class over a non-generic one using `object`?
2. Why does a generic class avoid the manual casting that an `object`-based equivalent requires?
3. What does `default(T)` do, and why is it necessary in a generic class?
4. Can a generic class have more than one type parameter? Give a built-in .NET example that does.

## 📝 30-second Revision Cheat Sheet

- Generic class = `class Box<T> { ... }` — one definition, works with any type, fully type-safe.
- Supports multiple type parameters: `class Pair<TFirst, TSecond>`.
- Avoids manual casting and runtime `InvalidCastException` risk compared to `object`-based designs.
- `default(T)` gives a safe default value for any type `T` (0, null, false, etc. as appropriate).
- Real-world use case: generic repository/data-access pattern.
