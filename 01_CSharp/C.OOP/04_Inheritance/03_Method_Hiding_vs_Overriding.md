# 🎭 Method Hiding vs Overriding

## 📌 What is it?

> **Overriding** (`virtual`/`override`) — derived class REPLACES the base class's implementation; the derived version runs even when called through a base reference (runtime polymorphism).
> **Hiding** (`new`) — derived class DEFINES a separate method with the same name; which version runs depends on the **reference type**, not the actual object (no polymorphism).

This is one of the most commonly confused — and most interview-tested — C# topics.

---

## 🧠 Intuition

```
OVERRIDING (virtual/override):
Animal a = new Dog();
a.Speak();   → calls Dog's Speak()  (runtime decides, based on ACTUAL object)

HIDING (new):
Animal a = new Dog();
a.Speak();   → calls Animal's Speak()  (compile-time decides, based on REFERENCE TYPE)
```

---

## 💻 Code Examples

**Overriding — polymorphic, correct behavior:**

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal makes a sound");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Dog barks");
}

Animal a = new Dog();
a.Speak();   // "Dog barks" — correct, runtime polymorphism works
```

**Hiding — the "gotcha" behavior:**

```csharp
public class Animal
{
    public void Speak() => Console.WriteLine("Animal makes a sound");   // NOT virtual
}

public class Dog : Animal
{
    public new void Speak() => Console.WriteLine("Dog barks");   // HIDES base method
}

Animal a = new Dog();
a.Speak();   // "Animal makes a sound" ❌ — reference type (Animal) decides, NOT actual object!

Dog d = new Dog();
d.Speak();   // "Dog barks" — when accessed via Dog reference, hiding version runs
```

---

## 📊 Overriding vs Hiding

| Aspect                                   | Overriding (`virtual`/`override`) | Hiding (`new`)                                        |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------------------- |
| Keyword in base                          | `virtual`                           | (none required)                                         |
| Keyword in derived                       | `override`                          | `new`                                                 |
| Which version runs (via base reference)? | Derived version (polymorphic)         | Base version (based on reference type)                  |
| Runtime vs Compile-time                  | Runtime polymorphism                  | Compile-time resolution                                 |
| Relationship to base                     | TRUE specialization/replacement       | Separate, unrelated method that happens to share a name |

---

## 🚨 Common Mistakes

- ❌ Forgetting `virtual` in the base class — without it, `override` in the derived class is a compile error, and you're forced into hiding (`new`) instead
- ❌ Using `new` when `override` was intended — silently breaks polymorphism, one of the most dangerous subtle bugs in OOP code (works fine until called via a base reference)
- ❌ Not marking the intent explicitly — always using `new` deliberately (with awareness), never by accident/omission

---

## 🎤 Interview Questions

| Question                                                             | Key Point                                                                                                                                                                             |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Difference between method overriding and method hiding?              | Overriding = true polymorphic replacement (`virtual`/`override`), runtime-resolved; Hiding = separate method with same name (`new`), resolved by reference type at compile-time |
| What happens if you call a hidden method via a base class reference? | The BASE class's version runs, not the derived one — this breaks polymorphism                                                                                                        |
| Can you override a non-virtual method?                               | No — the base method must be marked`virtual` (or `abstract`) to be overridden                                                                                                    |

---

## 📝 30-Second Revision Cheat Sheet

- Overriding (`virtual` + `override`) → runtime polymorphism, derived version wins even via base reference
- Hiding (`new`) → compile-time resolution, reference type decides which version runs
- Missing `virtual` in base + trying `override` in derived = compile error
- Hiding is a common accidental bug source — always be intentional about i

# Method Hiding vs Overriding

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
