# 👪 Base and Derived Class

## 📌 What is it?

> **Base class** (parent/superclass) — the class being inherited from.
> **Derived class** (child/subclass) — the class that inherits from the base class.

---

## 💻 Code Examples

**Basic — identifying base vs derived:**

```csharp
public class Animal          // BASE class
{
    public string Name;
    public void Eat() => Console.WriteLine($"{Name} is eating");
}

public class Dog : Animal    // DERIVED class
{
    public void Bark() => Console.WriteLine($"{Name} is barking");
}

var dog = new Dog { Name = "Rex" };
dog.Eat();    // inherited from Animal
dog.Bark();   // defined in Dog itself
```

**Intermediate — a derived class can itself be a base for another (multi-level):**

```csharp
public class Animal { public string Name; }
public class Dog : Animal { public void Bark() { } }
public class Puppy : Dog { public void Play() { } }   // Puppy → Dog → Animal chain

var puppy = new Puppy { Name = "Buddy" };
puppy.Bark();  // from Dog
puppy.Play();  // from Puppy itself
```

**Practical — polymorphic reference (base type referencing derived object):**

```csharp
Animal myPet = new Dog { Name = "Rex" };   // base class reference, derived object
myPet.Eat();     // ✅ works — Eat() is defined in Animal
// myPet.Bark(); ❌ Compile error — Bark() isn't visible via Animal reference, even though the object IS a Dog
```

> This "upcasting" behavior is explored fully in `05_Polymorphism/04_Upcasting_and_Downcasting.md`.

---

## 📊 Base vs Derived — Quick Reference

| Aspect                      | Base Class               | Derived Class                                         |
| --------------------------- | ------------------------ | ----------------------------------------------------- |
| Role                        | Provides common members  | Extends/specializes the base                          |
| Can be instantiated alone?  | Yes (unless`abstract`) | Yes                                                   |
| Access to base members      | N/A                      | Can access`public`/`protected` members            |
| Can override base behavior? | N/A                      | Yes, using`virtual`/`override` (see Polymorphism) |

---

## 🚨 Common Mistakes

- ❌ Referencing a derived object through a base type variable and expecting derived-only members to be accessible without casting
- ❌ Assuming inheritance chains can go infinitely deep without design cost — deep hierarchies (Animal → Dog → Puppy → BabyPuppy...) become hard to maintain; prefer composition when hierarchies get too deep

---

## 🎤 Interview Questions

| Question                                                                               | Key Point                                                                 |
| -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| What's a base class vs derived class?                                                  | Base = the class being inherited from; Derived = the class inheriting     |
| Can a derived class also be a base class for another?                                  | Yes — multi-level inheritance is allowed                                 |
| If you assign a`Dog` object to an `Animal` reference, what members are accessible? | Only members defined in`Animal` (base), unless you cast back to `Dog` |

---

## 📝 30-Second Revision Cheat Sheet

- Base class = provides shared members | Derived class = extends/specializes
- Multi-level inheritance is allowed (chains of base → derived → derived)
- Base-typed reference to a derived object only exposes base members (without casting

# Base and Derived Class

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
