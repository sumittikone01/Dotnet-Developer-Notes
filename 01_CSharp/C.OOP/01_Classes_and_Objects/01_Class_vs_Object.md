# 🏛️ Class vs Object

## 📌 What is it?

> **Class** — a blueprint/template defining structure (fields, properties) and behavior (methods).
> **Object** — an actual instance created from that class, living in memory with real data.

---

## 🌍 Real-world analogy

A **class** is like a **house blueprint** — it defines rooms, doors, layout, but you can't live in a blueprint. An **object** is an **actual house built** from that blueprint — you can build many houses (objects) from the same blueprint (class), each with different paint colors, furniture (property values).

---

## 🖼 Visual

```
CLASS (Blueprint)                    OBJECTS (Instances)
┌───────────────────┐
│ class Car          │        car1 = new Car("Red", "Toyota")
│  - Color            │  ──►  car2 = new Car("Blue", "Honda")
│  - Brand            │        car3 = new Car("Black", "BMW")
│  + Drive()          │
└───────────────────┘        Each object has its OWN data,
                              but SHARES the same method logic
```

---

## 💻 Code Examples

**Basic:**

```csharp
class Car
{
    public string Color;
    public string Brand;

    public void Drive()
    {
        Console.WriteLine($"{Color} {Brand} is driving");
    }
}

// Creating objects (instances)
Car car1 = new Car { Color = "Red", Brand = "Toyota" };
Car car2 = new Car { Color = "Blue", Brand = "Honda" };

car1.Drive();   // Red Toyota is driving
car2.Drive();   // Blue Honda is driving
```

**Practical — real project example (BAL layer):**

```csharp
public class Customer
{
    public int Id;
    public string Name;
    public string Email;
}

// Each row from the database becomes a separate OBJECT of the Customer CLASS
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "Sumit", Email = "sumit@x.com" },
    new Customer { Id = 2, Name = "Rahul", Email = "rahul@x.com" }
};
```

---

## 📊 Class vs Object

| Aspect     | Class                              | Object                                    |
| ---------- | ---------------------------------- | ----------------------------------------- |
| Definition | Blueprint/template                 | Instance created from the class           |
| Memory     | No memory until instantiated       | Allocated on heap when created via`new` |
| Existence  | Exists at compile-time (as a type) | Exists at runtime                         |
| Count      | One class definition               | Many objects can be created from it       |

---

## 🚨 Common Mistakes

- ❌ Confusing a class declaration with an actual usable object — a class alone holds no data until instantiated
- ❌ Forgetting each object has independent field values (unless static — see `08_Advanced_OOP/01_Static_Classes_and_Members.md`)

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                          |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| What's the difference between a class and an object? | Class = blueprint/type; Object = runtime instance with actual data |
| How many objects can be created from one class?      | Unlimited — each with independent field values                    |
| Where are objects stored in memory?                  | Heap (class instances are reference types)                         |

---

## 📝 30-Second Revision Cheat Sheet

- Class = blueprint (compile-time) | Object = instance (runtime, on heap)
- One class → many independent objects
- `new` keyword creates an object from a cla

# Class vs Object

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
