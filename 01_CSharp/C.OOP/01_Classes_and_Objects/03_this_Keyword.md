# 👉 this Keyword

## 📌 What is it?

> `this` refers to the **current instance** of the class — the specific object the method/constructor is running on.

---

## 🤔 Why do we need it?

Mainly to resolve **naming conflicts** between a class field/property and a parameter/local variable with the same name.

---

## 💻 Code Examples

**Basic — resolving name conflict:**

```csharp
public class Product
{
    private string name;

    public Product(string name)
    {
        this.name = name;   // 'this.name' = field, 'name' = parameter
    }
}
```

**Intermediate — constructor chaining with `this()`:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public Product() : this("Unnamed", 0) { }   // calls the other constructor

    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}
```

> More on this in `03_Constructors/05_Constructor_Chaining.md`.

**Practical — passing current instance to another method:**

```csharp
public class OrderValidator
{
    public void Validate(Order order)
    {
        if (order.Quantity <= 0)
            throw new ArgumentException("Invalid quantity");
    }
}

public class Order
{
    public int Quantity;

    public void Save()
    {
        var validator = new OrderValidator();
        validator.Validate(this);   // pass current object instance
    }
}
```

---

## 🚨 Common Mistakes

- ❌ Using `this` unnecessarily when there's no naming conflict — adds clutter (some teams prefer always using it for consistency; follow your team's convention)
- ❌ Trying to use `this` inside a `static` method — not allowed, since static methods don't belong to an instance

---

## 🎤 Interview Questions

| Question                                | Key Point                                                                         |
| --------------------------------------- | --------------------------------------------------------------------------------- |
| What does`this` refer to?             | The current object instance the method/constructor is executing on                |
| Can you use`this` in a static method? | No — static members don't belong to any instance                                 |
| Why use`this()` in a constructor?     | To chain to another constructor in the same class, avoiding duplicated init logic |

---

## 📝 30-Second Revision Cheat Sheet

- `this` = reference to the current object instance
- Common use: resolve field vs parameter naming conflicts
- `this(...)` = constructor chaining within the same class
- Not usable inside `static` metho

# this Keyword

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
