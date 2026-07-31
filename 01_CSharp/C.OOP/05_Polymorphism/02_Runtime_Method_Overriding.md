# 🏃 Runtime Polymorphism (Method Overriding)

## 📌 What is it?

> **Runtime polymorphism** (dynamic polymorphism) — the specific method implementation to call is determined at **runtime**, based on the **actual object type**, not the reference type. Achieved via `virtual` + `override`.

---

## 🧠 Intuition

```
Animal a = new Dog();
a.Speak();

At COMPILE time: compiler only knows 'a' is typed as Animal
At RUNTIME: CLR looks at the ACTUAL object (Dog) and calls Dog's Speak()

This is "late binding" — the decision is deferred until the program runs.
```

---

## 💻 Code Examples

**Basic — classic runtime polymorphism:**

```csharp
public class Shape
{
    public virtual double GetArea() => 0;
}

public class Circle : Shape
{
    public double Radius;
    public override double GetArea() => Math.PI * Radius * Radius;
}

public class Square : Shape
{
    public double Side;
    public override double GetArea() => Side * Side;
}

List<Shape> shapes = new List<Shape> { new Circle { Radius = 5 }, new Square { Side = 4 } };

foreach (Shape shape in shapes)
{
    Console.WriteLine(shape.GetArea());   // calls the CORRECT overridden version for each actual object
}
// Output: 78.54 (Circle), 16 (Square)
```

**Practical — real-world use case, polymorphic processing (matches your daily patterns):**

```csharp
public abstract class NotificationSender
{
    public abstract void Send(string message);
}

public class EmailSender : NotificationSender
{
    public override void Send(string message) => Console.WriteLine($"Email: {message}");
}

public class SmsSender : NotificationSender
{
    public override void Send(string message) => Console.WriteLine($"SMS: {message}");
}

void NotifyUser(NotificationSender sender, string message)
{
    sender.Send(message);   // doesn't care WHICH sender it is — runtime resolves the correct Send()
}

NotifyUser(new EmailSender(), "Order confirmed");   // "Email: Order confirmed"
NotifyUser(new SmsSender(), "Order confirmed");     // "SMS: Order confirmed"
```

> 💡 This pattern — coding against a base type/interface and letting runtime polymorphism handle the specifics — is the foundation of writing extensible, maintainable business logic.

---

## 📊 Requirements for Runtime Polymorphism

| Requirement                                      | Detail                                                              |
| ------------------------------------------------ | ------------------------------------------------------------------- |
| Base method must be`virtual` (or `abstract`) | Signals it CAN be overridden                                        |
| Derived method must use`override`              | Explicitly replaces the base behavior                               |
| Access via base type reference or collection     | The "polymorphic" behavior shows when working through the base type |

---

## 🚨 Common Mistakes

- ❌ Forgetting `virtual` on the base method — without it, `override` won't compile, and you're forced into hiding (`new`) which breaks polymorphism (see `04_Inheritance/03_Method_Hiding_vs_Overriding.md`)
- ❌ Not realizing polymorphism's value until working with **collections of a base type** — the real payoff is processing mixed derived types uniformly (like the `shapes` list example)

---

## 🎤 Interview Questions

| Question                              | Key Point                                                                                                      |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| What is runtime polymorphism?         | The correct overridden method is determined at runtime based on the actual object type, not the reference type |
| What's required to achieve it?        | `virtual` (or `abstract`) in base class + `override` in derived class                                    |
| Why is it also called "late binding"? | Method resolution is deferred until runtime, unlike overloading which is resolved at compile-time              |

---

## 📝 30-Second Revision Cheat Sheet

- Runtime polymorphism = correct overridden method chosen at runtime, based on actual object type
- Requires: `virtual`/`abstract` in base + `override` in derived
- Also called: dynamic polymorphism / late binding
- Real power: process a list of mixed derived types uniformly through the base typ

# Runtime Method Overriding

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
