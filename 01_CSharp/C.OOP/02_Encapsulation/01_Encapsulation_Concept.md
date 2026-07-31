# 🔒 Encapsulation Concept

## 📌 What is it?

> **Encapsulation** = bundling data (fields) and behavior (methods) together inside a class, while **restricting direct access** to internal details from outside.

One of the 4 pillars of OOP (Encapsulation, Abstraction, Inheritance, Polymorphism).

---

## 🌍 Real-world analogy

Think of a **capsule medicine** — the active ingredients are sealed inside; you don't interact with the raw chemicals directly, you just take the capsule as designed. Similarly, a class hides its internal data and exposes only safe, controlled ways to interact with it (via public methods/properties).

---

## 🧠 Intuition

```
❌ Without Encapsulation:
public class BankAccount
{
    public decimal Balance;   // anyone can do: account.Balance = -99999;  💥
}

✅ With Encapsulation:
public class BankAccount
{
    private decimal balance;

    public decimal Balance => balance;   // read-only from outside

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("Invalid amount");
        balance += amount;
    }

    public void Withdraw(decimal amount)
    {
        if (amount > balance) throw new InvalidOperationException("Insufficient funds");
        balance -= amount;
    }
}
```

---

## 💻 Code Example — Practical

```csharp
public class Employee
{
    private decimal salary;   // hidden from outside

    public string Name { get; set; }

    public decimal Salary
    {
        get => salary;
        set
        {
            if (value < 0)
                throw new ArgumentException("Salary cannot be negative");
            salary = value;
        }
    }

    public decimal GetAnnualSalary() => salary * 12;   // controlled behavior exposed
}
```

---

## 📊 Benefits of Encapsulation

| Benefit                      | Explanation                                                       |
| ---------------------------- | ----------------------------------------------------------------- |
| **Data protection**    | Prevents invalid states (e.g., negative balance)                  |
| **Flexibility**        | Internal implementation can change without breaking external code |
| **Controlled access**  | You decide what's readable/writable via access modifiers          |
| **Easier maintenance** | Changes are localized inside the class                            |

---

## 🚨 Common Mistakes

- ❌ Making all fields `public` — defeats the entire purpose of encapsulation
- ❌ Exposing a mutable collection field directly (`public List<Order> Orders;`) — callers can bypass validation and clear/modify the list directly. Prefer exposing `IReadOnlyList<T>` or controlled add/remove methods
- ❌ Confusing encapsulation with just "using properties" — it's about hiding *implementation details* and protecting invariants, not just wrapping fields

---

## 🎤 Interview Questions

| Question                             | Key Point                                                                              |
| ------------------------------------ | -------------------------------------------------------------------------------------- |
| What is encapsulation?               | Bundling data + behavior, restricting direct access to protect internal state          |
| How is encapsulation achieved in C#? | Access modifiers (`private`, `public`) + properties with validation logic          |
| Why not just make everything public? | Loses control over valid state — any code could corrupt data (e.g., negative balance) |

---

## 📝 30-Second Revision Cheat Sheet

- Encapsulation = hide internal data, expose controlled access
- Achieved via: `private` fields + `public` properties/methods with validation
- Protects object's internal state ("invariants") from invalid external changes
- One of the 4 OOP pilla

# Encapsulation Concept

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
