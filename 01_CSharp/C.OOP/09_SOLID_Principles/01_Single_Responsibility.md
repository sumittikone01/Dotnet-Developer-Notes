# 1️⃣ Single Responsibility Principle (SRP)

## 📌 What is it?

> **"A class should have only ONE reason to change."** Each class should be responsible for a single, well-defined piece of functionality.

The **S** in **SOLID**.

---

## 🧠 Intuition

```
❌ VIOLATES SRP:
class OrderManager
{
    void CreateOrder() { }
    void SaveToDatabase() { }        ← data access responsibility
    void SendConfirmationEmail() { } ← notification responsibility
    void GenerateInvoicePdf() { }    ← reporting responsibility
}
// 4 DIFFERENT reasons this class could need to change!

✅ FOLLOWS SRP:
class OrderService { void CreateOrder() { } }
class OrderRepository { void SaveToDatabase() { } }
class EmailService { void SendConfirmationEmail() { } }
class InvoiceGenerator { void GenerateInvoicePdf() { } }
```

---

## 💻 Code Example — Real Project Refactor

**Before (violates SRP):**

```csharp
public class OrderManager
{
    public void ProcessOrder(Order order)
    {
        // validation
        if (order.Quantity <= 0) throw new ArgumentException("Invalid quantity");

        // database logic
        using (var conn = new SqlConnection("connection_string"))
        {
            // save order via ADO.NET...
        }

        // email logic
        var smtp = new SmtpClient();
        smtp.Send("order confirmation email...");
    }
}
```

**After (follows SRP — matches your BAL/DAL architecture!):**

```csharp
public class OrderValidator
{
    public bool Validate(Order order) => order.Quantity > 0;
}

public class OrderRepository   // DAL — single responsibility: data access
{
    public void Save(Order order) { /* ADO.NET save logic */ }
}

public class EmailNotifier    // single responsibility: notifications
{
    public void SendConfirmation(Order order) { /* SMTP logic */ }
}

public class OrderService     // BAL — orchestrates, single responsibility: business workflow
{
    private readonly OrderValidator _validator;
    private readonly OrderRepository _repository;
    private readonly EmailNotifier _notifier;

    public OrderService(OrderValidator validator, OrderRepository repository, EmailNotifier notifier)
    {
        _validator = validator;
        _repository = repository;
        _notifier = notifier;
    }

    public void ProcessOrder(Order order)
    {
        if (!_validator.Validate(order)) throw new ArgumentException("Invalid order");
        _repository.Save(order);
        _notifier.SendConfirmation(order);
    }
}
```

> 💡 This is EXACTLY why your company uses BAL/DAL layering — it's SRP applied at an architectural level. Controllers handle HTTP, BAL handles business logic, DAL handles data access.

---

## 🚨 Common Mistakes

- ❌ Creating "God classes" that do everything (validation + data access + notifications + reporting)
- ❌ Over-splitting into too many tiny classes for trivial responsibilities — SRP is about cohesive responsibility, not maximum fragmentation
- ❌ Confusing SRP with "one method per class" — SRP is about one **reason to change**, a class can have multiple related methods

---

## 🎤 Interview Questions

| Question                                          | Key Point                                                                                              |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| What does SRP mean?                               | A class should have only one reason to change — one well-defined responsibility                       |
| How does SRP relate to your BAL/DAL architecture? | Controller (HTTP), BAL (business logic), DAL (data access) each have a single, separate responsibility |
| What's a "God class" and why is it bad?           | A class doing too many unrelated things — hard to maintain, test, and reason about; violates SRP      |

---

## 📝 30-Second Revision Cheat Sheet

- SRP = one class, one reason to change, one responsibility
- Real-world application: Controller/BAL/DAL layered architecture
- Avoid "God classes" that mix validation, data access, and notification logi

# Single Responsibility Principle

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
