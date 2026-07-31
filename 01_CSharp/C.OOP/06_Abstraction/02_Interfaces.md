# 🔌 Interfaces

## 📌 What is it?

> An **interface** defines a **contract** — a set of members (methods, properties, events) that implementing classes MUST provide, with (traditionally) no implementation of their own.

---

## 🌍 Real-world analogy

Think of an interface like a **USB port specification**. Any device (mouse, keyboard, flash drive) that follows the USB contract can plug in — the port doesn't care HOW the device works internally, only that it follows the agreed interface (shape, protocol).

---

## 💻 Code Examples

**Basic:**

```csharp
public interface IShape
{
    double GetArea();          // no implementation, no body
    double GetPerimeter();
}

public class Circle : IShape
{
    public double Radius;
    public double GetArea() => Math.PI * Radius * Radius;
    public double GetPerimeter() => 2 * Math.PI * Radius;
}
```

**Intermediate — multiple interface implementation (this is why interfaces matter):**

```csharp
public interface IPrintable { void Print(); }
public interface ISavable { void Save(); }

public class Report : IPrintable, ISavable   // implements MULTIPLE interfaces — classes can't do this with base classes!
{
    public void Print() => Console.WriteLine("Printing report");
    public void Save() => Console.WriteLine("Saving report");
}
```

**Practical — real project pattern: Dependency Injection with interfaces (very common in your MVC stack):**

```csharp
public interface IOrderRepository
{
    Order GetById(int id);
    void Add(Order order);
}

public class OrderRepository : IOrderRepository   // concrete implementation
{
    public Order GetById(int id) { /* ADO.NET call */ return null; }
    public void Add(Order order) { /* ADO.NET call */ }
}

public class OrderController : Controller
{
    private readonly IOrderRepository _repository;   // depends on the INTERFACE, not the concrete class

    public OrderController(IOrderRepository repository)   // injected via DI container
    {
        _repository = repository;
    }
}
```

> 💡 This interface-based pattern is the foundation of **Dependency Injection** — controllers depend on abstractions (`IOrderRepository`), not concrete implementations, making code testable and swappable.

---

## 📊 Interface Evolution in C#

| Version | Feature                                                                                                                                                      |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| C# 1–7 | Only method/property signatures, NO implementation, NO fields                                                                                                |
| C# 8+   | Default interface methods allowed (implementation IS possible now) — see`04_Explicit_Interface_Implementation.md` and `05_Interface_Default_Methods.md` |
| Always  | Still cannot have instance fields (only properties)                                                                                                          |

---

## 🚨 Common Mistakes

- ❌ Adding instance fields to an interface — never allowed (properties are fine, fields are not)
- ❌ Forgetting a class implementing an interface must implement ALL members, unless the interface provides a default implementation (C# 8+)
- ❌ Not leveraging interfaces for testability — coding directly against concrete classes makes unit testing/mocking much harder

---

## 🎤 Interview Questions

| Question                                               | Key Point                                                                                                                         |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| Can a class implement multiple interfaces?             | Yes — this is a key advantage over single class inheritance                                                                      |
| Can interfaces have fields?                            | No — only properties, methods, events, indexers                                                                                  |
| Why are interfaces important for Dependency Injection? | They let code depend on abstractions rather than concrete implementations, enabling mocking/testing and swappable implementations |

---

## 📝 30-Second Revision Cheat Sheet

- Interface = contract, no fields, (traditionally) no implementation
- A class can implement MULTIPLE interfaces (unlike single class inheritance)
- Foundation of Dependency Injection — depend on `IOrderRepository`, not `OrderRepository`
- C# 8+ allows default method implementations in interface

# Interfaces

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
