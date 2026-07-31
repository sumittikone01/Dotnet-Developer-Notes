# 🎯 Explicit Interface Implementation

## 📌 What is it?

> A way to implement an interface member such that it can **only be accessed through an interface reference**, not directly through the class — used to resolve naming conflicts or hide implementation details.

---

## 🤔 Why do we need it?

Two scenarios:

1. **Name conflicts** — a class implements two interfaces that both have a method with the same name
2. **Hiding interface details** — you want the interface method to not clutter the class's public API

---

## 💻 Code Examples

**Basic — resolving a naming conflict:**

```csharp
public interface IEnglishGreeting { void Greet(); }
public interface IHindiGreeting { void Greet(); }

public class Person : IEnglishGreeting, IHindiGreeting
{
    void IEnglishGreeting.Greet() => Console.WriteLine("Hello!");    // explicit
    void IHindiGreeting.Greet() => Console.WriteLine("Namaste!");    // explicit
}

var person = new Person();
// person.Greet();  ❌ Compile error — not accessible directly

IEnglishGreeting eng = person;
eng.Greet();    // "Hello!"

IHindiGreeting hindi = person;
hindi.Greet();  // "Namaste!"
```

**Intermediate — hiding implementation detail from public API:**

```csharp
public interface IValidator
{
    bool Validate(object data);
}

public class OrderProcessor : IValidator
{
    bool IValidator.Validate(object data)   // explicit — hidden from OrderProcessor's normal usage
    {
        return data != null;
    }

    public void Process(Order order)
    {
        IValidator validator = this;
        if (!validator.Validate(order))
            throw new ArgumentException("Invalid order");

        Console.WriteLine("Processing order...");
    }
}
```

---

## 📊 Implicit vs Explicit Interface Implementation

| Aspect                          | Implicit                     | Explicit                                                    |
| ------------------------------- | ---------------------------- | ----------------------------------------------------------- |
| Syntax                          | `public void Greet()`      | `void IInterfaceName.Greet()`                             |
| Accessible via class reference? | ✅ Yes                       | ❌ No — only via interface reference                       |
| Access modifier                 | Must be`public`            | Cannot specify one (implicitly private-ish)                 |
| Use case                        | Normal, most common scenario | Resolving conflicts, hiding interface details from main API |

---

## 🚨 Common Mistakes

- ❌ Trying to call an explicitly implemented method directly on the class instance — always requires casting to the interface type first
- ❌ Adding an access modifier (`public`) to an explicit implementation — not allowed, causes compile error
- ❌ Overusing explicit implementation when there's no actual naming conflict — implicit implementation is simpler and more discoverable

---

## 🎤 Interview Questions

| Question                                                                           | Key Point                                                                                                              |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| When would you use explicit interface implementation?                              | To resolve naming conflicts between multiple interfaces, or to hide interface members from the class's main public API |
| Can you call an explicitly implemented method through the class instance directly? | No — you must cast/reference it through the interface type                                                            |
| Can explicit implementations have access modifiers?                                | No — they cannot have`public`/`private` etc.                                                                      |

---

## 📝 30-Second Revision Cheat Sheet

- Explicit implementation: `void IInterface.Method()` — only accessible via interface reference
- Solves: naming conflicts across multiple interfaces, hiding implementation from main API
- Cannot have access modifiers, cannot be called directly on the class instanc

# Explicit Interface Implementation

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
