# 🔩 Composition

## 📌 What is it?

> **Composition** is a "HAS-A" relationship with **strong ownership** — the contained object's lifecycle is completely tied to the container. If the parent is destroyed, the child is destroyed too.

---

## 🌍 Real-world analogy

A **Car** HAS an **Engine**. The Engine doesn't exist independently as a meaningful concept outside a specific car in this context — it's created WITH the car and destroyed WITH the car. Unlike aggregation, you don't take an Engine from a scrapped Car and casually plug it into another unrelated system.

---

## 💻 Code Example

```csharp
public class Engine
{
    public int Horsepower;
}

public class Car
{
    private readonly Engine engine;   // composition — Car OWNS the Engine

    public Car(int horsepower)
    {
        engine = new Engine { Horsepower = horsepower };   // Engine is CREATED inside Car
    }

    public void Start() => Console.WriteLine($"Starting engine with {engine.Horsepower} HP");
}

var car = new Car(300);
car.Start();
// The Engine object has NO existence outside this Car — created and destroyed with it
```

---

## 📊 Composition vs Aggregation — The Key Test

| Question                                             | Aggregation                     | Composition                               |
| ---------------------------------------------------- | ------------------------------- | ----------------------------------------- |
| Is the "part" created INSIDE the container?          | ❌ No — passed in from outside | ✅ Yes — created by the container itself |
| Can the "part" be shared across multiple containers? | ✅ Yes                          | ❌ No — exclusively owned                |
| Does destroying the container destroy the part?      | ❌ No                           | ✅ Yes                                    |

---

## 🌍 "HAS-A" vs "IS-A" — Composition vs Inheritance (Design Principle)

> 💡 **"Favor composition over inheritance"** — a well-known OOP design principle. Composition offers more flexibility (can change behavior at runtime by swapping components) compared to inheritance's rigid, compile-time hierarchy.

```csharp
// Inheritance approach (rigid)
public class FlyingCar : Car { public void Fly() { } }

// Composition approach (flexible)
public class Car
{
    private IFlightCapability flightModule;   // can be null, swapped, or added later
    public void SetFlightModule(IFlightCapability module) => flightModule = module;
}
```

---

## 🚨 Common Mistakes

- ❌ Choosing inheritance for code reuse when composition would be more flexible and less tightly coupled
- ❌ Not properly disposing composed objects that implement `IDisposable` — since the container owns them, it's responsible for their cleanup too (see `J.Memory_and_Runtime/02_IDisposable_and_using.md`)

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                                                                             |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| What is composition?                                 | A HAS-A relationship with strong ownership — the "part" cannot exist independently of the "whole"                    |
| What does "favor composition over inheritance" mean? | Prefer building behavior by combining smaller objects rather than rigid inheritance hierarchies, for more flexibility |
| Real-world example of composition?                   | A Car and its Engine — Engine has no independent existence outside that Car                                          |

---

## 📝 30-Second Revision Cheat Sheet

- Composition = HAS-A, strong ownership, "part" dies with the "whole"
- Part is created INSIDE the container, not passed in from outside
- "Favor composition over inheritance" — more flexible than rigid class hierarchies
- If the container is `IDisposable`-aware, it should dispose composed objects to

# Composition

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
