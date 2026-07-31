# 🆚 HAS-A vs IS-A

## 📌 What is it?

> Two fundamental ways to model relationships between classes:
>
> - **IS-A** → Inheritance (`class Dog : Animal`) — Dog IS-A Animal
> - **HAS-A** → Composition/Aggregation (`class Car { Engine engine; }`) — Car HAS-A Engine

---

## 🧠 Intuition — The Test

Ask: **"Is X truly a specialized type of Y, or does X just use/contain a Y?"**

```
Dog IS-A Animal          → true specialization → use INHERITANCE
Car HAS-A Engine         → Car contains/uses an Engine → use COMPOSITION
Employee IS-A Person     → true specialization → use INHERITANCE
Order HAS-A Customer     → Order references a Customer → use ASSOCIATION/AGGREGATION
```

---

## 💻 Code Comparison

**IS-A (Inheritance):**

```csharp
public class Animal { public virtual void Eat() { } }
public class Dog : Animal { }   // Dog IS-A Animal — inherits Animal's full contract
```

**HAS-A (Composition):**

```csharp
public class Engine { public void Start() { } }
public class Car
{
    private readonly Engine engine = new();   // Car HAS-A Engine — doesn't inherit from it
    public void StartCar() => engine.Start();
}
```

---

## 📊 Decision Table

| Question                                                                     | If YES →                           | If NO →          |
| ---------------------------------------------------------------------------- | ----------------------------------- | ----------------- |
| Is it truly a specialized subtype (passes "is a" test naturally in English)? | IS-A → Inheritance                 | Continue below    |
| Does it need to reuse behavior by containing/using another object?           | HAS-A → Composition/Aggregation    | Reconsider design |
| Would inheriting expose behavior that doesn't make sense for the subtype?    | Don't use inheritance — a red flag | —                |

---

## 🚨 Common Mistakes — The Classic Anti-Pattern

```csharp
// ❌ WRONG — modeling HAS-A as IS-A
public class Engine { public int Horsepower; }
public class Car : Engine { }   // "Car IS-A Engine"?? Doesn't make sense!

// ✅ CORRECT — Car HAS-A Engine
public class Car
{
    private Engine engine;
}
```

- ❌ Using inheritance purely for code reuse when the relationship isn't a true "is-a" — leads to fragile, illogical hierarchies
- ❌ The classic "Square IS-A Rectangle" trap — mathematically true, but often violates behavioral expectations in code (Liskov Substitution Principle issue — see `09_SOLID_Principles/03_Liskov_Substitution.md`)

---

## 🎤 Interview Questions

| Question                                                                          | Key Point                                                                           |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| How do you decide between inheritance and composition?                            | Ask if the relationship is truly "is-a" (specialization) or "has-a" (contains/uses) |
| Why is "favor composition over inheritance" good advice?                          | Composition is more flexible, less tightly coupled, and avoids fragile hierarchies  |
| Give an example of misusing inheritance where composition should be used instead. | Making`Car` inherit from `Engine` instead of containing an `Engine` instance  |

---

## 📝 30-Second Revision Cheat Sheet

- IS-A → Inheritance (`Dog : Animal`) — true specialization
- HAS-A → Composition/Aggregation (`Car` contains `Engine`) — contains/uses relationship
- Test: does the English sentence "X is a Y" sound natural and behaviorally correct? If not, use HAS-A
- Prefer composition when in doubt — more flexible, less fragil

# HAS-A vs IS-A

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
