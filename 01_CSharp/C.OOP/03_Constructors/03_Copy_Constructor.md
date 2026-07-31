# 📋 Copy Constructor

## 📌 What is it?

> A constructor that creates a **new object by copying values from an existing object** of the same class. Unlike some languages (C++), C# doesn't auto-generate one — you write it manually.

---

## 🤔 Why do we need it?

Since classes are reference types, simple assignment (`obj2 = obj1`) only copies the **reference**, not the data (see `05_Value_vs_Reference_Types.md`). A copy constructor lets you create a truly **independent duplicate**.

---

## 💻 Code Examples

**Basic:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }

    // Copy constructor
    public Product(Product source)
    {
        Name = source.Name;
        Price = source.Price;
    }
}

var original = new Product("Laptop", 55000m);
var copy = new Product(original);   // independent copy

copy.Price = 60000m;
Console.WriteLine(original.Price);  // 55000 — unaffected!
Console.WriteLine(copy.Price);      // 60000
```

**Intermediate — the problem it solves (reference copy pitfall):**

```csharp
var original = new Product("Laptop", 55000m);
var reference = original;          // NOT a copy — same object!
reference.Price = 60000m;

Console.WriteLine(original.Price); // 60000 — original changed too! ❌
```

**Practical — shallow vs deep copy consideration:**

```csharp
public class Order
{
    public string CustomerName;
    public List<string> Items;

    public Order(Order source)
    {
        CustomerName = source.CustomerName;          // value-like copy — safe
        Items = new List<string>(source.Items);       // ⚠️ must explicitly copy the list too!
                                                        // Items = source.Items; would share the SAME list (shallow copy)
    }
}
```

---

## 📊 Shallow Copy vs Deep Copy

| Aspect                                                   | Shallow Copy                                              | Deep Copy                                                 |
| -------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------- |
| Value type fields                                        | Copied correctly (independent)                            | Copied correctly (independent)                            |
| Reference type fields (e.g.,`List<T>`, nested objects) | Only the reference is copied — both point to SAME object | New independent copies are created for nested objects too |
| Risk                                                     | Mutating a nested object in the copy affects the original | Fully independent — safest                               |

---

## 🚨 Common Mistakes

- ❌ Writing a copy constructor that only does a shallow copy when nested reference types need independence — must manually deep-copy collections/nested objects
- ❌ Assuming `obj2 = obj1` creates a copy — it just copies the reference (both point to the same object)
- ❌ Forgetting C# has no built-in copy constructor syntax like C++ — you must write it yourself (or use `MemberwiseClone()` for shallow copies)

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                                                                              |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Why doesn't`obj2 = obj1` create a copy?            | Classes are reference types — assignment copies the reference, not the underlying data                                |
| What's the difference between shallow and deep copy? | Shallow copies reference-type fields by reference (shared); deep copy creates independent copies of nested objects too |
| Does C# auto-generate copy constructors?             | No — must be written manually, unlike C++                                                                             |

---

## 📝 30-Second Revision Cheat Sheet

- Copy constructor = manually written, creates independent duplicate object
- `obj2 = obj1` ≠ copy — just copies the reference (same object)
- Watch out for shallow copy pitfalls with nested reference-type fields (lists, nested objects)
- C# has no automatic copy constructor — write it yourse

# Copy Constructor

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
