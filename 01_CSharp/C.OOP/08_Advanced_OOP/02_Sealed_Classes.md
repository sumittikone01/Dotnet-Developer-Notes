# 🔒 Sealed Classes

## 📌 What is it?

> `sealed` prevents a class from being **inherited further**. A sealed class can still be instantiated and used normally — it just can't be a base class for another class.

---

## 🤔 Why do we need it?

- **Security/design intent** — prevent unintended extension that could break assumptions
- **Performance** — the JIT compiler can apply certain optimizations (like devirtualization) more aggressively on sealed classes/methods since there's no possibility of further overriding

---

## 💻 Code Examples

**Basic:**

```csharp
public sealed class PaymentProcessor
{
    public void ProcessPayment(decimal amount) => Console.WriteLine($"Processing {amount}");
}

// public class CustomPaymentProcessor : PaymentProcessor { }  ❌ Compile error — cannot inherit a sealed class

var processor = new PaymentProcessor();   // ✅ still fully usable, just not inheritable
processor.ProcessPayment(500);
```

**Intermediate — `sealed override` on a specific method (not the whole class):**

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal sound");
}

public class Dog : Animal
{
    public sealed override void Speak() => Console.WriteLine("Bark");   // seals just this method
}

public class Puppy : Dog
{
    // public override void Speak() { }  ❌ Compile error — Speak() is sealed in Dog
}
```

> Covered in more depth in `05_Polymorphism/03_virtual_override_new_Keywords.md`.

---

## 📊 When to Use `sealed`

| Scenario                                                     | Use`sealed`?                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------------ |
| A utility/helper class not meant for extension               | ✅ Yes                                                             |
| Security-sensitive classes (e.g., authentication logic)      | ✅ Yes — prevents tampering via inheritance                       |
| A class designed as part of a plugin/extensibility framework | ❌ No — inheritance is the intended use case                      |
| Framework/library public APIs where extension isn't planned  | ✅ Often — reduces support burden and unintended breaking changes |

---

## 🚨 Common Mistakes

- ❌ Sealing every class "just in case" — over-restricts legitimate extension needs; sealing should be intentional, not default
- ❌ Confusing `sealed class` (prevents inheritance) with `readonly` (prevents field reassignment) — completely different concepts
- ❌ Forgetting `string` is itself a `sealed` class in .NET — you can never inherit from `string`

---

## 🎤 Interview Questions

| Question                                 | Key Point                                                                            |
| ---------------------------------------- | ------------------------------------------------------------------------------------ |
| What does`sealed` do?                  | Prevents a class from being further inherited                                        |
| Can a sealed class be instantiated?      | Yes — sealing only blocks inheritance, not instantiation                            |
| Why might`sealed` improve performance? | Enables JIT optimizations like method devirtualization since no override is possible |
| Is`string` sealed in .NET?             | Yes                                                                                  |

---

## 📝 30-Second Revision Cheat Sheet

- `sealed class` = can be instantiated, but NOT inherited further
- `sealed override` = seals just one method against further overriding in deeper subclasses
- Use for: security-sensitive classes, finalized utility classes, performance-critical hot paths
- `string` is a real-world example of a sealed class in .NE

# Sealed Classes

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
