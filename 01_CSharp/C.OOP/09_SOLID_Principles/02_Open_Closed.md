# 2️⃣ Open/Closed Principle (OCP)

## 📌 What is it?

> **"Classes should be OPEN for extension, but CLOSED for modification."** You should be able to add new functionality WITHOUT changing existing, tested code.

The **O** in **SOLID**.

---

## 🧠 Intuition

```
❌ VIOLATES OCP — must MODIFY existing method for every new type:
decimal CalculateDiscount(string customerType, decimal amount)
{
    if (customerType == "Regular") return amount * 0.05m;
    if (customerType == "Premium") return amount * 0.10m;
    if (customerType == "VIP") return amount * 0.20m;
    // adding a new type means editing THIS method again — risky, retest everything
}

✅ FOLLOWS OCP — EXTEND via new classes, don't touch existing code:
interface IDiscountStrategy { decimal Calculate(decimal amount); }
class RegularDiscount : IDiscountStrategy { public decimal Calculate(decimal amount) => amount * 0.05m; }
class PremiumDiscount : IDiscountStrategy { public decimal Calculate(decimal amount) => amount * 0.10m; }
// Adding VIP? Just add a NEW class — zero changes to existing, tested code!
class VipDiscount : IDiscountStrategy { public decimal Calculate(decimal amount) => amount * 0.20m; }
```

---

## 💻 Code Example — Full Pattern (Strategy Pattern)

```csharp
public interface IDiscountStrategy
{
    decimal Calculate(decimal amount);
}

public class RegularDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal amount) => amount * 0.05m;
}

public class PremiumDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal amount) => amount * 0.10m;
}

public class OrderService
{
    private readonly IDiscountStrategy _discountStrategy;

    public OrderService(IDiscountStrategy discountStrategy)   // injected — depends on ABSTRACTION
    {
        _discountStrategy = discountStrategy;
    }

    public decimal GetFinalPrice(decimal amount)
    {
        return amount - _discountStrategy.Calculate(amount);
    }
}

// Usage — swap strategy without touching OrderService at all:
var service = new OrderService(new PremiumDiscount());
Console.WriteLine(service.GetFinalPrice(1000));
```

---

## 🌍 Real-world analogy

Think of a **power strip/extension cord**. You can plug in NEW devices (extend functionality) without rewiring the wall socket (modifying the core system). The socket is "closed" for modification but "open" for extension via new plugs.

---

## 🚨 Common Mistakes

- ❌ Using long `if/else` or `switch` chains that need editing every time a new case is added — a strong signal OCP is being violated
- ❌ Over-engineering with strategy patterns for things that will realistically never change — OCP is valuable when extension is a REAL, anticipated need, not a hypothetical one
- ❌ Forgetting OCP typically requires abstraction (interfaces/abstract classes) to work — you can't "extend without modifying" concrete, non-abstracted code easily

---

## 🎤 Interview Questions

| Question                                                 | Key Point                                                                                     |
| -------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| What does Open/Closed Principle mean?                    | Classes should allow extension (new behavior) without requiring modification of existing code |
| How is OCP typically achieved in C#?                     | Through abstraction — interfaces or abstract classes, combined with polymorphism             |
| What's a code smell that suggests OCP is being violated? | Long if/else or switch statements that grow every time a new type/case is added               |

---

## 📝 30-Second Revision Cheat Sheet

- OCP = open for extension, closed for modification
- Achieved via interfaces/abstract classes + polymorphism (Strategy Pattern is the classic example)
- Warning sign of violation: growing if/else or switch chains for every new cas

# Open Closed Principle

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
