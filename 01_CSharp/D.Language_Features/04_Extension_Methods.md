# 🧩 Extension Methods

## 📌 What is it?

An **Extension Method** lets you **add new methods to an existing type** — even one you don't own the source code for (like `string` or `List<T>`) — without modifying its original class, using inheritance, or wrapping it.

```csharp
string name = "rohit";
bool result = name.IsNullOrWhitespace();  // "IsNullOrWhitespace" isn't a built-in string method!
```

## 🤔 Why do we need it?

Sometimes you want to add convenient, reusable behavior to a type you **can't modify** — a .NET built-in type (`string`, `int`, `IEnumerable<T>`), or a type from a third-party library. Extension methods solve this by letting you "attach" new methods from the outside.

This is also **exactly how LINQ itself works** — `.Where()`, `.Select()`, `.OrderBy()` are all extension methods on `IEnumerable<T>`, not methods built into the interface itself!

## 🌍 Real-world analogy

Think of a **phone case with a built-in wallet slot**. You didn't modify the phone itself (you can't — it's sealed, manufactured) — you attached an accessory that adds new functionality from the outside, and it *feels* like it was part of the phone all along.

## ⚙️ How to Define One

**Rules:**

1. Must be in a **static class**.
2. Must be a **static method**.
3. First parameter must use the `this` keyword, followed by the type being extended.

```csharp
public static class StringExtensions
{
    public static bool IsNullOrWhitespace(this string value)
    {
        return string.IsNullOrEmpty(value) || value.Trim().Length == 0;
    }
}
```

**Usage** — looks exactly like a normal instance method:

```csharp
string input = "   ";
bool empty = input.IsNullOrWhitespace();   // true

// Behind the scenes, the compiler actually rewrites this to:
bool empty = StringExtensions.IsNullOrWhitespace(input);
```

> 💡 This is the key insight: extension methods are **syntactic sugar**. The compiler transforms `obj.Method()` into `StaticClass.Method(obj)` at compile time — nothing is actually added to the original type at runtime.

## 💻 More Practical Examples

**Extending a custom domain type:**

```csharp
public static class OrderExtensions
{
    public static bool IsHighValue(this Order order) => order.Total > 10000;
}

// Usage:
if (myOrder.IsHighValue()) { ApplyPriorityShipping(); }
```

**Extending `IEnumerable<T>` (how LINQ itself is built):**

```csharp
public static class EnumerableExtensions
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T> source) 
        where T : class
    {
        foreach (var item in source)
            if (item != null) yield return item;
    }
}

// Usage:
var validItems = myList.WhereNotNull();
```

## 📊 Extension Method vs Instance Method

| Aspect                                 | Instance Method         | Extension Method                    |
| -------------------------------------- | ----------------------- | ----------------------------------- |
| **Requires source code access?** | ✅ Yes                  | ❌ No                               |
| **Can access private members?**  | ✅ Yes                  | ❌ No — only public API            |
| **Can override in subclass?**    | ✅ Yes (virtual)        | ❌ No — always resolved statically |
| **Where defined**                | Inside the class itself | Separate static class               |

⚠️ If a type later adds a **real instance method** with the same name/signature as your extension method, the real instance method always **wins** — extension methods have lower priority in method resolution.

## 🚨 Common Mistakes

- ❌ Overusing extension methods for logic that really belongs as a proper instance method on a class you *do* own — extension methods are for types you *can't* modify, not a replacement for good OOP design.
- ❌ Putting extension methods in random/unclear namespaces — makes them hard for teammates to discover (they only appear via IntelliSense if the namespace is imported).
- ❌ Writing extension methods with side effects that surprise callers (e.g. `list.RemoveDuplicates()` that mutates instead of returning a new collection) — should behave predictably, ideally without hidden mutation.
- ❌ Calling an extension method on a `null` reference and being surprised it doesn't throw immediately — extension methods CAN be called on `null` (since they're really just static method calls), so you must handle `null` explicitly inside the method if needed.

## 💡 Best Practices

- Group extension methods into clearly-named static classes (e.g. `StringExtensions`, `EnumerableExtensions`) in a dedicated `Extensions` folder/namespace.
- Use them to add small, focused, reusable helper behavior — not core business logic.
- Keep them **pure** (no unexpected side effects) where possible — a caller expects `obj.DoSomething()` to behave predictably like a normal method.
- Remember: they only work if the containing namespace is imported (`using`) wherever you call them.

## 🎤 Interview Questions

1. What are the syntax requirements for defining an extension method?
2. How does the compiler actually resolve `obj.ExtensionMethod()` under the hood?
3. Why can't extension methods access private members of the type they extend?
4. What happens if a class later adds a real instance method with the same signature as an existing extension method?
5. Name a widely-used C# feature that is entirely built using extension methods.

## 📝 30-second Revision Cheat Sheet

- Extension method = static method in a static class, first param uses `this TypeName`.
- Lets you add methods to types you don't own (built-in types, third-party libraries).
- Compiler rewrites `obj.Method()` → `StaticClass.Method(obj)` — pure syntax sugar.
- Real instance methods always take priority over extension methods with the same signature.
- **LINQ is built entirely on extension methods** over `IEnumerable<T>`.
