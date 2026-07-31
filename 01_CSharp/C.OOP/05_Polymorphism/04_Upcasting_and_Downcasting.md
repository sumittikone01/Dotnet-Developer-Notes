# ⬆️⬇️ Upcasting & Downcasting

## 📌 What is it?

> **Upcasting** — treating a derived object as its base type (implicit, always safe).
> **Downcasting** — treating a base-typed reference back as its derived type (explicit, can fail).

---

## 🧠 Intuition

```
Dog dog = new Dog();
Animal a = dog;              // UPCASTING — implicit, safe (Dog IS-A Animal)

Animal a2 = new Dog();
Dog d = (Dog)a2;             // DOWNCASTING — explicit, only safe if a2 ACTUALLY holds a Dog
```

---

## 💻 Code Examples

**Basic — Upcasting (implicit, always safe):**

```csharp
public class Animal { public virtual void Speak() => Console.WriteLine("Animal sound"); }
public class Dog : Animal { public override void Speak() => Console.WriteLine("Bark"); }

Dog dog = new Dog();
Animal animal = dog;   // implicit upcast — no cast syntax needed
animal.Speak();        // "Bark" — polymorphism still applies
```

**Intermediate — Downcasting (explicit, risky):**

```csharp
Animal animal = new Dog();
Dog dog = (Dog)animal;   // explicit downcast — works because animal IS actually a Dog

Animal animal2 = new Animal();
Dog dog2 = (Dog)animal2;  // ❌ throws InvalidCastException — animal2 is NOT a Dog
```

**Practical — safe downcasting with `as` and pattern matching:**

```csharp
Animal animal = GetSomeAnimal();   // could be Dog, Cat, etc.

// Option 1: 'as' operator — returns null instead of throwing
Dog dog = animal as Dog;
if (dog != null)
    dog.Speak();

// Option 2: pattern matching (modern, preferred, C# 7+)
if (animal is Dog d)
{
    d.Speak();
}

// Option 3: switch pattern matching for multiple types
string description = animal switch
{
    Dog => "It's a dog",
    Cat => "It's a cat",
    _ => "Unknown animal"
};
```

---

## 📊 Casting Methods Comparison

| Method                  | Syntax                   | Behavior on failure                                            |
| ----------------------- | ------------------------ | -------------------------------------------------------------- |
| Direct cast             | `(Dog)animal`          | Throws`InvalidCastException`                                 |
| `as` operator         | `animal as Dog`        | Returns`null` (only works for reference types)               |
| `is` pattern matching | `if (animal is Dog d)` | Returns`false`, no exception, safely extracts typed variable |

---

## 🚨 Common Mistakes

- ❌ Using direct cast `(Dog)animal` without checking type first — throws `InvalidCastException` if wrong
- ❌ Forgetting `as` only works with reference types/nullable types, not plain value types
- ❌ Overusing downcasting in general — frequent downcasting often signals a design smell; consider polymorphism (virtual methods) instead of type-checking + casting

---

## 🎤 Interview Questions

| Question                                                 | Key Point                                                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| What's the difference between upcasting and downcasting? | Upcasting = derived→base, implicit, always safe; Downcasting = base→derived, explicit, can fail |
| What's the safest way to downcast?                       | `is` pattern matching (`if (obj is Dog d)`) — avoids exceptions entirely                     |
| Why is upcasting always safe?                            | Because a derived object always fully satisfies the base type's contract (IS-A relationship)      |

---

## 📝 30-Second Revision Cheat Sheet

- Upcasting = derived → base, implicit, always safe
- Downcasting = base → derived, explicit, CAN throw `InvalidCastException`
- Safe downcasting: `as` (returns null) or `is` pattern matching (preferred, modern)
- Frequent downcasting = potential design smell — consider polymorphism instea

# Upcasting and Downcasting

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
