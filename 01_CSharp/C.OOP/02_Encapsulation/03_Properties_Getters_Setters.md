# 🎚️ Properties: Getters & Setters

## 📌 What is it?

> `get` and `set` are **accessors** inside a property that control how a value is read and written — allowing you to add logic, validation, or restrict access direction.

---

## 💻 Code Examples

**Basic — full property with backing field:**

```csharp
public class Product
{
    private string name;

    public string Name
    {
        get { return name; }
        set { name = value; }   // 'value' is the implicit parameter passed in
    }
}
```

**Intermediate — asymmetric access (different modifiers per accessor):**

```csharp
public class Order
{
    public int Id { get; private set; }   // readable everywhere, settable only inside class

    public Order(int id)
    {
        Id = id;   // allowed — inside the class
    }
}

var order = new Order(101);
// order.Id = 200;  ❌ Compile error — setter is private
Console.WriteLine(order.Id);  // ✅ getter is public
```

**Practical — validation logic in setter (common in ViewModels):**

```csharp
public class ProductViewModel
{
    private int quantity;

    public int Quantity
    {
        get => quantity;
        set
        {
            if (value < 0)
                throw new ArgumentException("Quantity cannot be negative");
            quantity = value;
        }
    }
}
```

**init-only setter (C# 9+) — set once at object creation, then immutable:**

```csharp
public class Customer
{
    public string Name { get; init; }
}

var customer = new Customer { Name = "Sumit" };
// customer.Name = "Rahul";  ❌ Compile error — init-only, can't modify after creation
```

> Covered in more depth in `D.Language_Features/10_init_Only_Setters.md`.

---

## 📊 Accessor Combinations

| Pattern                   | Meaning                                                    |
| ------------------------- | ---------------------------------------------------------- |
| `{ get; set; }`         | Fully read/write                                           |
| `{ get; }`              | Read-only (must be set in constructor)                     |
| `{ get; private set; }` | Publicly readable, only settable inside the class          |
| `{ get; init; }`        | Settable only during object initialization, then immutable |
| `{ set; }` only         | Write-only (rare, unusual design)                          |

---

## 🚨 Common Mistakes

- ❌ Making setters fully public when the value should only change via a controlled method (e.g., `Balance` should change via `Deposit()`/`Withdraw()`, not a raw setter)
- ❌ Putting expensive computation inside a getter — getters are expected to be fast, side-effect-free
- ❌ Forgetting `value` is the implicit keyword representing the incoming value inside a `set` block

---

## 🎤 Interview Questions

| Question                                                     | Key Point                                                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| What does`value` represent inside a setter?                | The value being assigned to the property (implicit parameter)                             |
| How do you make a property read-only from outside the class? | `{ get; private set; }` or just `{ get; }` (set only in constructor)                  |
| What's the difference between`set` and `init`?           | `set` allows changes anytime; `init` only allows setting during object initialization |

---

## 📝 30-Second Revision Cheat Sheet

- `get`/`set` = controlled read/write access to a property
- `value` = implicit keyword inside `set`, representing incoming value
- `{ get; private set; }` = public read, internal-only write
- `init` (C# 9+) = settable only at creation time, then immutab

# Properties Getters Setters

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
