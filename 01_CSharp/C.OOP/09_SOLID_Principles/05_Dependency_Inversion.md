# 5️⃣ Dependency Inversion Principle (DIP)

## 📌 What is it?

> **"High-level modules should not depend on low-level modules; both should depend on abstractions."** Depend on interfaces/abstractions, not concrete implementations.

The **D** in **SOLID** — arguably the most important one for real-world architecture (this is the foundation of Dependency Injection).

---

## 🧠 Intuition

```
❌ VIOLATES DIP — high-level class directly depends on low-level concrete class:
class OrderService
{
    private SqlOrderRepository repository = new SqlOrderRepository();   // tightly coupled!
    // If you switch databases, or want to unit test with a fake repository, you're stuck
}

✅ FOLLOWS DIP — both depend on an ABSTRACTION:
interface IOrderRepository { void Save(Order order); }

class SqlOrderRepository : IOrderRepository { public void Save(Order order) { /* SQL logic */ } }

class OrderService
{
    private readonly IOrderRepository repository;   // depends on the ABSTRACTION

    public OrderService(IOrderRepository repository)   // injected — Dependency Injection!
    {
        this.repository = repository;
    }
}
```

---

## 💻 Code Example — Full Real-World Pattern

```csharp
public interface IOrderRepository
{
    Order GetById(int id);
    void Save(Order order);
}

public class SqlOrderRepository : IOrderRepository   // low-level module — concrete DAL implementation
{
    public Order GetById(int id) { /* ADO.NET stored procedure call */ return null; }
    public void Save(Order order) { /* ADO.NET save logic */ }
}

public class OrderService   // high-level module — BAL, depends on ABSTRACTION not concrete class
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public void PlaceOrder(Order order)
    {
        _repository.Save(order);
    }
}

// Program.cs — dependency injection wiring (ASP.NET Core built-in DI container)
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddScoped<OrderService>();

// Controller receives OrderService, which receives IOrderRepository — all wired automatically
public class OrderController : Controller
{
    private readonly OrderService _orderService;
    public OrderController(OrderService orderService) => _orderService = orderService;
}
```

> 💡 This IS your ASP.NET Core MVC architecture — Controllers depend on BAL interfaces, BAL depends on DAL interfaces, and ASP.NET Core's built-in DI container wires up the concrete implementations at runtime.

---

## 🖼 High-Level vs Low-Level — Visual

```
WITHOUT DIP:
OrderService (high-level)  ──depends on──►  SqlOrderRepository (low-level, concrete)
                                             tightly coupled, hard to test/swap

WITH DIP:
OrderService (high-level)  ──depends on──►  IOrderRepository (ABSTRACTION)
                                                      ▲
                                                      │ implements
                                             SqlOrderRepository (low-level, concrete)

Both high-level AND low-level now depend on the SAME abstraction — decoupled!
```

---

## 📊 Why DIP Matters (Real Benefits)

| Benefit               | Explanation                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **Testability** | Can inject a fake/mock`IOrderRepository` in unit tests, no real database needed         |
| **Flexibility** | Swap`SqlOrderRepository` for `MongoOrderRepository` without touching `OrderService` |
| **Decoupling**  | `OrderService` doesn't know or care HOW data is persisted                               |

---

## 🚨 Common Mistakes

- ❌ Confusing "Dependency Inversion" with "Dependency Injection" — DIP is the PRINCIPLE (depend on abstractions); DI is a TECHNIQUE/pattern often used to implement it (injecting dependencies via constructor)
- ❌ Instantiating concrete dependencies directly inside a class (`new SqlOrderRepository()`) instead of injecting an interface
- ❌ Depending on abstractions that are just as volatile/unstable as the concrete implementation — the abstraction should be genuinely stable

---

## 🎤 Interview Questions

| Question                                                          | Key Point                                                                                                         |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| What is the Dependency Inversion Principle?                       | High-level and low-level modules should both depend on abstractions, not on each other directly                   |
| Difference between Dependency Inversion and Dependency Injection? | DIP = the design principle; DI = a technique (constructor/property injection) used to achieve it                  |
| Why is DIP important for unit testing?                            | Allows swapping real dependencies for mocks/fakes via the shared interface, without touching the class under test |

---

## 📝 30-Second Revision Cheat Sheet

- DIP = depend on abstractions (interfaces), not concrete implementations
- Foundation of Dependency Injection — your Controller → BAL → DAL architecture IS DIP in action
- DIP (principle) ≠ DI (technique) — related but distinct terms
- Key benefit: testability + flexibility to swap implementation

# Dependency Inversion Principle

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
