# 🧱 Fields, Properties & Methods

## 📌 What is it?

> **Field** — a variable declared directly inside a class (raw storage).
> **Property** — a controlled wrapper around a field, exposing `get`/`set` accessors with optional logic.
> **Method** — a function that defines the class's behavior.

---

## 🧠 Intuition

```
Field:     private string _name;              ← raw storage, no control
Property:  public string Name { get; set; }   ← controlled access, can add validation
Method:    public void Save() { ... }          ← behavior
```

---

## 📊 Field vs Property

| Aspect                   | Field                       | Property                                     |
| ------------------------ | --------------------------- | -------------------------------------------- |
| Access control           | Direct, no validation       | Can validate/transform in`get`/`set`     |
| Convention               | `private`, `_camelCase` | `public`, `PascalCase`                   |
| Data binding (Kendo/MVC) | ❌ Doesn't work well        | ✅ Required for model binding, serialization |
| Encapsulation            | Poor if public              | Good — hides internal representation        |

---

## 💻 Code Examples

**Basic — auto-implemented property (most common):**

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

**Intermediate — property with validation logic (backing field):**

```csharp
public class Product
{
    private decimal _price;

    public decimal Price
    {
        get => _price;
        set
        {
            if (value < 0)
                throw new ArgumentException("Price cannot be negative");
            _price = value;
        }
    }
}

var p = new Product();
// p.Price = -10;   ❌ throws ArgumentException
p.Price = 99.99m;    // ✅ works
```

**Practical — read-only computed property (common in ViewModels):**

```csharp
public class OrderViewModel
{
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }

    public decimal Total => Quantity * UnitPrice;   // computed, no setter needed
}
```

---

## ⚙️ Property Types

| Type                         | Syntax                                   | Use Case                                   |
| ---------------------------- | ---------------------------------------- | ------------------------------------------ |
| Auto-implemented             | `public string Name { get; set; }`     | Simple data holder, no extra logic         |
| Full property                | `get { } set { }` with backing field   | Validation, transformation logic           |
| Read-only (init-time)        | `public string Name { get; }`          | Set only in constructor, immutable after   |
| Expression-bodied (computed) | `public decimal Total => Qty * Price;` | Derived/calculated value, no backing field |

---

## 🚨 Common Mistakes

- ❌ Exposing public fields directly instead of properties — breaks encapsulation, can't add validation later without breaking API
- ❌ Forgetting Kendo UI Grid / MVC model binding requires **properties**, not raw public fields, to work correctly
- ❌ Adding heavy logic inside a property getter — getters should be fast/cheap; use a method if computation is expensive

---

## 🎤 Interview Questions

| Question                                     | Key Point                                                                         |
| -------------------------------------------- | --------------------------------------------------------------------------------- |
| Why use properties instead of public fields? | Encapsulation — allows validation/transformation without breaking the public API |
| What is an auto-implemented property?        | Compiler-generated backing field, shorthand`{ get; set; }` syntax               |
| Can a property have only a getter?           | Yes — makes it effectively read-only from outside the class                      |

---

## 📝 30-Second Revision Cheat Sheet

- Field = raw storage (usually private) | Property = controlled access (usually public)
- Auto-property `{ get; set; }` = most common for simple data
- Full property with backing field = needed for validation logic
- Kendo/MVC binding requires properties, not public fiel

# Fields Properties Methods

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
