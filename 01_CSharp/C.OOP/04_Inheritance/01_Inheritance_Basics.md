# 🧬 Inheritance Basics

## 📌 What is it?

> **Inheritance** allows a class (derived/child) to acquire fields, properties, and methods from another class (base/parent), promoting code reuse.

One of the 4 OOP pillars.

---

## 🌍 Real-world analogy

Think of a **Vehicle** as a general blueprint — it has an engine, wheels, and can move. A **Car** and a **Motorcycle** both **inherit** these common traits from `Vehicle`, then add their own specifics (Car has 4 doors, Motorcycle has a kickstand).

---

## 🧠 Intuition

```
        ┌────────────┐
        │  Vehicle    │   (base/parent class)
        │  - Speed    │
        │  - Move()   │
        └──────┬─────┘
               │ inherits
      ┌────────┴────────┐
      ▼                 ▼
┌──────────┐     ┌─────────────┐
│   Car     │     │ Motorcycle   │  (derived/child classes)
│ + Doors   │     │ + HasKickstand │
└──────────┘     └─────────────┘
```

---

## 💻 Code Examples

**Basic:**

```csharp
public class Vehicle
{
    public int Speed;
    public void Move() => Console.WriteLine("Vehicle is moving");
}

public class Car : Vehicle   // Car INHERITS from Vehicle
{
    public int Doors;
}

var car = new Car { Speed = 100, Doors = 4 };
car.Move();   // "Vehicle is moving" — inherited method works on Car too!
```

**Intermediate — adding new behavior on top of inherited:**

```csharp
public class Employee
{
    public string Name;
    public decimal BaseSalary;

    public decimal CalculateSalary() => BaseSalary;
}

public class Manager : Employee
{
    public decimal Bonus;

    public decimal CalculateTotalCompensation() => CalculateSalary() + Bonus;   // reuses base method
}
```

**Practical — real project scenario (common ADO.NET entity base class):**

```csharp
public class BaseEntity
{
    public int Id { get; set; }
    public DateTime CreatedDate { get; set; }
    public bool IsActive { get; set; } = true;
}

public class Product : BaseEntity   // inherits Id, CreatedDate, IsActive
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class Customer : BaseEntity   // also inherits the same base fields
{
    public string Email { get; set; }
}
```

---

## ⚙️ Key Rules

| Rule                                                | Detail                                                                                       |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Single inheritance only                             | A class can inherit from only ONE base class (C# doesn't support multiple class inheritance) |
| Multiple interfaces allowed                         | A class CAN implement multiple interfaces, though (see`06_Abstraction/02_Interfaces.md`)   |
| `sealed` classes can't be inherited               | See`08_Advanced_OOP/02_Sealed_Classes.md`                                                  |
| Everything is`public`/`protected` unless hidden | `private` members of the base class are NOT accessible in the derived class                |

---

## 🚨 Common Mistakes

- ❌ Trying to inherit from multiple classes (`class Car : Vehicle, Machine`) — not allowed in C#, use interfaces instead
- ❌ Assuming `private` base class members are accessible in derived classes — they're not; use `protected` if derived classes need access
- ❌ Overusing inheritance for code reuse when composition would be more appropriate (see `07_Object_Relationships/04_HAS_A_vs_IS_A.md`)

---

## 🎤 Interview Questions

| Question                                                      | Key Point                                                        |
| ------------------------------------------------------------- | ---------------------------------------------------------------- |
| What is inheritance?                                          | Mechanism allowing a class to acquire members from another class |
| Does C# support multiple inheritance?                         | No, for classes — but yes for interfaces                        |
| Can a derived class access private members of its base class? | No — only`protected` or `public` members are accessible     |

---

## 📝 30-Second Revision Cheat Sheet

- Inheritance = derived class acquires base class's members
- `class Derived : Base` syntax
- Single class inheritance only (multiple interface implementation allowed)
- `private` base members NOT accessible in derived class — use `protected`

# Inheritance Basics

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
