# ⛓️ Constructor Chaining

## 📌 What is it?

> Calling **one constructor from another** — either within the same class (`this(...)`) or from a base class (`base(...)`) — to avoid duplicating initialization logic.

---

## 💻 Code Examples

**Basic — chaining within the same class using `this()`:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;
    public string Category;

    public Product() : this("Unnamed", 0, "General") { }

    public Product(string name, decimal price) : this(name, price, "General") { }

    public Product(string name, decimal price, string category)
    {
        Name = name;
        Price = price;
        Category = category;
    }
}

var p1 = new Product();                          // uses defaults via chaining
var p2 = new Product("Laptop", 55000m);           // Category defaults to "General"
var p3 = new Product("Phone", 20000m, "Mobile");  // fully specified
```

**Intermediate — chaining to a base class using `base()`:**

```csharp
public class Person
{
    public string Name;

    public Person(string name)
    {
        Name = name;
    }
}

public class Employee : Person
{
    public string Department;

    public Employee(string name, string department) : base(name)  // calls Person's constructor first
    {
        Department = department;
    }
}

var emp = new Employee("Sumit", "IT");
Console.WriteLine(emp.Name);        // "Sumit" — set via base constructor
Console.WriteLine(emp.Department);  // "IT"
```

> More on `base` in `04_Inheritance/05_Constructor_in_Inheritance.md`.

**Practical — real-world DTO/entity chaining:**

```csharp
public class BaseEntity
{
    public int Id { get; }
    public DateTime CreatedAt { get; }

    public BaseEntity(int id)
    {
        Id = id;
        CreatedAt = DateTime.Now;
    }
}

public class Order : BaseEntity
{
    public decimal Total { get; set; }

    public Order(int id, decimal total) : base(id)   // chains to BaseEntity, ensures Id/CreatedAt always set
    {
        Total = total;
    }
}
```

---

## 🖼 Execution Order Diagram

```
new Employee("Sumit", "IT")
        │
        ▼
Employee(name, department) : base(name)
        │
        ▼ (base constructor runs FIRST)
Person(name) executes  →  Name = "Sumit"
        │
        ▼ (then derived constructor body runs)
Employee body executes  →  Department = "IT"
```

> ⚠️ Key rule: base/chained constructor **always runs before** the current constructor's own body.

---

## 🚨 Common Mistakes

- ❌ Duplicating initialization logic across multiple constructors instead of chaining — leads to maintenance headaches (update one, forget the other)
- ❌ Assuming derived constructor logic runs before the base constructor — it's actually the opposite; base always runs first
- ❌ Circular chaining (`this()` calling itself indirectly) — compile error

---

## 🎤 Interview Questions

| Question                                      | Key Point                                                                                                         |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| What is constructor chaining?                 | Calling one constructor from another (`this(...)` same class, `base(...)` parent class) to reduce duplication |
| In inheritance, which constructor runs first? | The base class constructor always runs before the derived class constructor's body                                |
| Why use constructor chaining?                 | Avoids duplicating initialization logic across multiple overloaded constructors                                   |

---

## 📝 30-Second Revision Cheat Sheet

- `this(...)` → chains to another constructor in the SAME class
- `base(...)` → chains to the parent class's constructor
- Base/chained constructor ALWAYS executes before the current constructor's body
- Reduces duplicate initialization code across overloaded constructo

# Constructor Chaining

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
