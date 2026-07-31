
# ⚖️ Equality & IEquatable

## 📌 What is it?

> Deep dive into how C# handles equality comparisons: `==` operator, `.Equals()` method, and the `IEquatable<T>` interface for type-safe, performant equality.

---

## 📊 Equality Mechanisms Compared

| Mechanism                             | Default behavior                              | Can be customized?                   |
| ------------------------------------- | --------------------------------------------- | ------------------------------------ |
| `==` operator (reference types)     | Reference equality                            | Yes — via operator overloading      |
| `==` operator (value types/structs) | Value equality (field-by-field)               | Yes — via operator overloading      |
| `.Equals()`                         | Reference equality (inherited from`object`) | Yes — via override                  |
| `IEquatable<T>.Equals(T other)`     | N/A — must implement                         | Type-safe, avoids boxing for structs |

---

## 💻 Code Examples

**Basic — the `==` vs `.Equals()` distinction:**

```csharp
public class Product
{
    public string Name;
}

var p1 = new Product { Name = "Laptop" };
var p2 = new Product { Name = "Laptop" };
var p3 = p1;

Console.WriteLine(p1 == p2);        // false — different objects, reference comparison
Console.WriteLine(p1 == p3);        // true — same object reference
Console.WriteLine(p1.Equals(p2));   // false — default Equals() is also reference-based
```

**Intermediate — implementing `IEquatable<T>` (recommended for value comparisons):**

```csharp
public class Product : IEquatable<Product>
{
    public string Name;
    public decimal Price;

    public bool Equals(Product other)   // type-safe, no boxing, no casting needed
    {
        if (other is null) return false;
        return Name == other.Name && Price == other.Price;
    }

    public override bool Equals(object obj) => Equals(obj as Product);   // delegate to typed version
    public override int GetHashCode() => HashCode.Combine(Name, Price);
}

var p1 = new Product { Name = "Laptop", Price = 55000 };
var p2 = new Product { Name = "Laptop", Price = 55000 };
Console.WriteLine(p1.Equals(p2));   // true — value-based comparison now
```

**Practical — string equality (a special case worth knowing):**

```csharp
string s1 = "Hello";
string s2 = "Hello";
Console.WriteLine(s1 == s2);        // true — string overloads '==' for value comparison, even though it's a reference type!

string s3 = new string("Hello".ToCharArray());
Console.WriteLine(s1 == s3);        // true — still value comparison (operator overload)
Console.WriteLine(ReferenceEquals(s1, s3));  // false — different objects in memory
```

---

## 🌍 Why `IEquatable<T>` Matters (Performance)

- Without `IEquatable<T>`: comparing structs via `Equals(object)` causes **boxing** (converting value type to reference type) — performance cost
- With `IEquatable<T>`: type-safe comparison, no boxing, faster — especially important for structs used heavily in collections

---

## 🚨 Common Mistakes

- ❌ Assuming `==` always does value comparison — only true for value types by default, and for `string` (special-cased), NOT for regular reference types unless explicitly overloaded
- ❌ Implementing `IEquatable<T>.Equals()` but forgetting to also override `object.Equals()` and `GetHashCode()` — leads to inconsistent equality behavior across different code paths
- ❌ Using `ReferenceEquals()` when logical/value equality was intended, or vice versa

---

## 🎤 Interview Questions

| Question                                           | Key Point                                                                                                                      |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Difference between`==` and `.Equals()`?        | For reference types, both default to reference equality — but`==` can be operator-overloaded independently of `.Equals()` |
| Why implement`IEquatable<T>`?                    | Type-safe, avoids boxing (crucial for structs), better performance in collections                                              |
| Why does`string` behave differently with `==`? | `string` overloads the `==` operator to perform value comparison, despite being a reference type                           |

---

## 📝 30-Second Revision Cheat Sheet

- `==` default = reference equality (except structs, and `string` which overloads it)
- `.Equals()` default = reference equality (inherited from `object`)
- `IEquatable<T>` = type-safe equality, avoids boxing, recommended for custom value comparisons
- Overriding `Equals()` → must also override `GetHashCode()` (see `04_Object_Class_Methods.md`)
