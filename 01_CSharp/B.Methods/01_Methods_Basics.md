# 🔧 Methods Basics

## 📌 What is it?

> A **method** is a named, reusable block of code that performs a specific task — optionally taking input (parameters) and optionally returning output.

---

## 🧠 Intuition

```
        ┌───────────────┐
Input ──►│    Method      │──► Output
(params) │  (logic runs)  │  (return value)
        └───────────────┘
```

---

## ⚙️ Anatomy of a Method

```csharp
public       int      CalculateTotal   (int quantity, decimal price)
   │          │              │                    │
Access     Return         Method              Parameters
Modifier    Type           Name
{
    return (int)(quantity * price);   // method body
}
```

| Part            | Purpose                                                                |
| --------------- | ---------------------------------------------------------------------- |
| Access modifier | Who can call it (`public`, `private`, `protected`, `internal`) |
| Return type     | What type of value it gives back (`void` if nothing)                 |
| Method name     | PascalCase, verb-based (`GetOrder`, `CalculateTotal`)              |
| Parameters      | Inputs the method needs                                                |
| Body            | The actual logic                                                       |

---

## 💻 Code Examples

**Basic — void vs return type:**

```csharp
// void — performs an action, returns nothing
void LogMessage(string message)
{
    Console.WriteLine(message);
}

// returns a value
int Add(int a, int b)
{
    return a + b;
}
```

**Intermediate — Expression-bodied method (concise syntax):**

```csharp
// Traditional
int Square(int x)
{
    return x * x;
}

// Expression-bodied (C# 6+) — same thing, shorter
int Square(int x) => x * x;
```

**Practical — Typical BAL method in your stack:**

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;

    public OrderService(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public OrderDto GetOrderById(int orderId)
    {
        var order = _orderRepository.GetById(orderId);   // DAL call

        if (order == null)
            return null;

        return new OrderDto
        {
            Id = order.Id,
            TotalAmount = order.TotalAmount
        };
    }
}
```

---

## 📊 Method Call Flow

```
Controller  →  BAL (Business Logic)  →  DAL (Data Access)  →  Stored Procedure
    │                  │                       │
 calls              calls                  executes
GetOrderById()   GetOrderById()          ADO.NET SqlCommand
```

> 💡 This is exactly your company's architecture — Controller calls BAL method, BAL calls DAL method, DAL executes the stored procedure via ADO.NET.

---

## 🚨 Common Mistakes

- ❌ Giving methods vague names (`DoStuff()`, `Process()`) — always name based on **what it does**, verb + noun (`CalculateDiscount`, `ValidateOrder`)
- ❌ Making one method do too many things — violates Single Responsibility Principle (see `C.OOP/09_SOLID_Principles`)
- ❌ Forgetting `return` in a non-void method — compile error, "not all code paths return a value"

---

## 🎤 Interview Questions

| Question                                 | Key Point                                                                  |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| What does`void` mean as a return type? | The method performs an action but returns no value                         |
| What is an expression-bodied method?     | Shorthand syntax (`=>`) for methods with a single-line body              |
| Why should methods be small and focused? | Easier to test, read, reuse — aligns with Single Responsibility Principle |

---

## 📝 30-Second Revision Cheat Sheet

- Method = reusable named block of logic, optional input/output
- `void` = no return value | otherwise, return type must match what's returned
- Expression-bodied (`=>`) = shorthand for single-line methods
- Real-world flow: Controller → BAL → DAL → Stored Procedu

# Methods Basics

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
