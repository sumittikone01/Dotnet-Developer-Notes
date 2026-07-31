# 🙈 Data Hiding

## 📌 What is it?

> **Data Hiding** is the practice of restricting direct access to an object's internal fields, exposing only what's necessary through a controlled interface (properties/methods).

It's a **specific technique** used to achieve encapsulation — not a separate pillar, but often tested as a distinct interview concept.

---

## 📊 Data Hiding vs Encapsulation (common confusion point)

| Aspect       | Encapsulation                                                     | Data Hiding                                           |
| ------------ | ----------------------------------------------------------------- | ----------------------------------------------------- |
| Scope        | Broader concept — bundling data + behavior                       | Narrower — specifically about restricting visibility |
| Goal         | Organize related data/behavior into one unit                      | Protect internal state from outside interference      |
| How achieved | Classes, properties, methods                                      | Access modifiers (`private`, `protected`)         |
| Relationship | Data hiding is a**technique used to achieve** encapsulation | A subset/mechanism of encapsulation                   |

> 💡 Interview-safe answer: "Data hiding is how we *achieve* encapsulation — encapsulation is the broader principle, data hiding is the specific mechanism of restricting field access."

---

## 💻 Code Example

```csharp
public class BankAccount
{
    private decimal balance;   // HIDDEN — no direct outside access

    public decimal GetBalance() => balance;   // controlled, read-only exposure

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("Invalid deposit");
        balance += amount;
    }
}

var account = new BankAccount();
// account.balance = 999999;  ❌ Not accessible — compile error
account.Deposit(500);          // ✅ only way to modify balance
Console.WriteLine(account.GetBalance());
```

---

## 🚨 Common Mistakes

- ❌ Treating "data hiding" and "encapsulation" as completely unrelated concepts in interviews — they're closely tied, data hiding is part of how encapsulation is implemented
- ❌ Hiding data but then exposing an unrestricted public setter anyway — defeats the purpose (e.g., `public decimal Balance { get; set; }` with no validation is NOT real data hiding)

---

## 🎤 Interview Questions

| Question                                         | Key Point                                                                                                |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| What is data hiding?                             | Restricting direct access to internal fields, exposing controlled access instead                         |
| How is data hiding different from encapsulation? | Data hiding is the specific technique (via access modifiers); encapsulation is the broader OOP principle |
| How do you implement data hiding in C#?          | `private`/`protected` fields + public properties/methods with validation                             |

---

## 📝 30-Second Revision Cheat Sheet

- Data hiding = restrict direct field access, expose controlled interface
- It's the mechanism; encapsulation is the broader principle
- Achieved via `private` fields + validated public properties/metho

# Data Hiding

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
