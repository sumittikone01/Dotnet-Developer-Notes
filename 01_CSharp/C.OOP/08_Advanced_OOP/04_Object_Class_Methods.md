# 🎯 Object Class Methods (ToString, Equals, GetHashCode)

## 📌 What is it?

> Every class in C# implicitly inherits from `System.Object` (or `object`), which provides several fundamental methods that can be overridden: `ToString()`, `Equals()`, `GetHashCode()`, and `GetType()`.

---

## 📊 Core Object Methods

| Method                 | Default Behavior                                         | Common Override Reason                              |
| ---------------------- | -------------------------------------------------------- | --------------------------------------------------- |
| `ToString()`         | Returns the full type name (e.g.,`MyApp.Product`)      | Provide a readable string representation            |
| `Equals(object obj)` | Reference equality (are they the SAME object in memory?) | Value-based equality (are they logically equal?)    |
| `GetHashCode()`      | Based on object's memory reference (roughly)             | MUST override alongside`Equals()` for consistency |
| `GetType()`          | Returns runtime type info                                | Rarely overridden                                   |

---

## 💻 Code Examples

**Basic — overriding ToString():**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public override string ToString() => $"{Name} - ${Price}";
}

var p = new Product { Name = "Laptop", Price = 55000 };
Console.WriteLine(p);           // "Laptop - $55000" — automatically calls ToString()
Console.WriteLine(p.ToString());  // same result
```

**Intermediate — overriding Equals() and GetHashCode() together (MUST be done together):**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public override bool Equals(object obj)
    {
        if (obj is not Product other) return false;
        return Name == other.Name && Price == other.Price;
    }

    public override int GetHashCode() => HashCode.Combine(Name, Price);
}

var p1 = new Product { Name = "Laptop", Price = 55000 };
var p2 = new Product { Name = "Laptop", Price = 55000 };

Console.WriteLine(p1 == p2);        // false — '==' still uses reference equality by default (unless operator overloaded)
Console.WriteLine(p1.Equals(p2));   // true — because we overrode Equals() for value-based comparison
```

**Practical — why this matters for collections (real-world impact):**

```csharp
var products = new HashSet<Product>();   // HashSet uses GetHashCode() + Equals() internally
products.Add(p1);
products.Add(p2);

Console.WriteLine(products.Count);  // 1 — because p1 and p2 are considered EQUAL (if Equals/GetHashCode overridden correctly)
                                     // Without overriding, this would be 2 (different reference identities)
```

---

## ⚙️ The Golden Rule

> ⚠️ **If you override `Equals()`, you MUST also override `GetHashCode()`** — and vice versa. Objects considered "equal" MUST produce the SAME hash code, or collections like `Dictionary`/`HashSet` will behave incorrectly (lookups fail unpredictably).

```
Equals() says A == B    →    GetHashCode() MUST return the SAME value for A and B
```

---

## 🚨 Common Mistakes

- ❌ Overriding `Equals()` without overriding `GetHashCode()` — breaks dictionary/hashset behavior, compiler even gives a warning
- ❌ Confusing `Equals()` (instance method, value comparison after override) with `==` (operator, still reference comparison unless explicitly overloaded)
- ❌ Not calling `base.ToString()` when extending, or forgetting `ToString()` is automatically called by `Console.WriteLine()`, string interpolation, and debugger displays

---

## 🎤 Interview Questions

| Question                                                           | Key Point                                                                                  |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| Why must`Equals()` and `GetHashCode()` be overridden together? | Objects considered equal MUST have the same hash code, or hash-based collections break     |
| What's the default behavior of`Equals()`?                        | Reference equality — checks if both variables point to the exact same object in memory    |
| When is`ToString()` automatically called?                        | By`Console.WriteLine()`, string interpolation (`$"{obj}"`), and debugger watch windows |

---

## 📝 30-Second Revision Cheat Sheet

- `ToString()` → readable string representation, auto-called by WriteLine/interpolation
- `Equals()` → default is reference equality; override for value-based equality
- `GetHashCode()` → MUST be overridden together with `Equals()` — golden rule
- Critical for correct behavior in `HashSet<T>`, `Dictionary<TKey, TValue>`

# Object Class Methods

---

## 📌 Overview

> Write your notes here.

---

## 🔑 Key Concepts

---

## 💻 Code Example

```csharp
```

---

## ❓ Interview Questions

---

## 🔗 Related Topics

---

*Last updated: 2026-03-15*
