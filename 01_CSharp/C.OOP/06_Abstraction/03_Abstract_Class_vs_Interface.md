# ⚖️ Abstract Class vs Interface

## 📌 What is it?

> The most-asked OOP interview question in C#. Both achieve abstraction, but serve different design purposes.

---

## 📊 Full Comparison Table

| Aspect                      | Abstract Class                                          | Interface                                                                          |
| --------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Instantiable?               | ❌ Never                                                | ❌ Never                                                                           |
| Multiple inheritance        | ❌ Only one abstract class can be inherited             | ✅ A class can implement multiple interfaces                                       |
| Fields                      | ✅ Allowed                                              | ❌ Not allowed (only properties)                                                   |
| Constructors                | ✅ Allowed                                              | ❌ Not allowed                                                                     |
| Access modifiers on members | ✅ Can use`public`, `protected`, `private`        | ⚠️ Members implicitly`public` (until explicitly implemented)                   |
| Method implementation       | ✅ Mix of abstract + concrete methods                   | ✅ (C# 8+) Default implementations allowed, but less common                        |
| Represents                  | "IS-A" relationship with SHARED implementation          | A capability/contract — "CAN-DO" relationship                                     |
| Versioning                  | Adding a new abstract method breaks all derived classes | (C# 8+) Can add default-implemented methods without breaking existing implementers |

---

## 🌍 Real-world analogy

- **Abstract class** = "Animal" — has shared traits (Eat, Sleep) all animals do the same way, PLUS some things each animal does differently (Speak). You can't have a generic "Animal" instance, only specific animals.
- **Interface** = "IFlyable" — a CAPABILITY. A `Bird` and an `Airplane` are completely unrelated in hierarchy, but BOTH can implement `IFlyable`. Interfaces cut ACROSS unrelated class hierarchies.

---

## 💻 Code Example — Choosing the Right One

```csharp
// Abstract class — shared implementation + IS-A relationship
public abstract class Employee
{
    public string Name;
    public decimal BaseSalary;

    public decimal CalculateBaseSalary() => BaseSalary;   // shared logic
    public abstract decimal CalculateBonus();              // must differ per employee type
}

public class Manager : Employee
{
    public override decimal CalculateBonus() => BaseSalary * 0.20m;
}

// Interface — capability, cuts across unrelated hierarchies
public interface ILoggable
{
    void LogActivity(string action);
}

public class Manager : Employee, ILoggable   // Manager IS-A Employee, but CAN-DO logging too
{
    public void LogActivity(string action) => Console.WriteLine($"Manager action: {action}");
}

public class OrderProcessor : ILoggable   // completely unrelated to Employee, but ALSO can log
{
    public void LogActivity(string action) => Console.WriteLine($"Process action: {action}");
}
```

---

## 🖼 Decision Flowchart

```
Do multiple, UNRELATED classes need to share this behavior?
        │
        ├── Yes → use INTERFACE (cuts across hierarchies)
        │
        └── No, only related classes in one hierarchy →
                    │
                    Do they share COMMON implementation code?
                    │
                    ├── Yes → use ABSTRACT CLASS
                    │
                    └── No, just a contract → INTERFACE still works fine
```

---

## 🚨 Common Mistakes

- ❌ Using an abstract class when the relationship is really a "capability" (CAN-DO) rather than "IS-A" — limits flexibility since a class can only inherit one abstract class
- ❌ Giving an interview answer that ONLY lists syntax differences — always mention the **design intent** difference (IS-A + shared code vs CAN-DO contract)
- ❌ Forgetting interfaces can be implemented by structs too, but abstract classes cannot be inherited by structs

---

## 🎤 Interview Questions

| Question                                                   | Key Point                                                                                                        |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| When would you choose an abstract class over an interface? | When derived classes share common implementation code and have a true IS-A hierarchical relationship             |
| When would you choose an interface over an abstract class? | When unrelated classes need to share a capability/contract, or when multiple "inheritance" of behavior is needed |
| Can an abstract class implement an interface?              | Yes — and it can choose to leave some interface members abstract for derived classes to implement               |
| Can a struct implement an interface?                       | Yes. Can a struct inherit an abstract class? No                                                                  |

---

## 📝 30-Second Revision Cheat Sheet

- Abstract class → IS-A + shared implementation, single inheritance only
- Interface → CAN-DO contract, multiple implementation allowed, (mostly) no fields/constructors
- Rule of thumb: shared code + hierarchy → abstract class; cross-cutting capability → interface
- A class can implement many interfaces but inherit only ONE abstract clas

# Abstract Class vs Interface

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
