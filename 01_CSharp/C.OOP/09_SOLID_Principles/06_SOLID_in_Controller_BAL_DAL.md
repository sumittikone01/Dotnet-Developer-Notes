# 🏢 SOLID in Controller / BAL / DAL (Real Project Example)

## 📌 What is it?

> A consolidated, real-world walkthrough showing all 5 SOLID principles applied together in the exact architecture you use daily: **Controller → BAL → DAL**, with ADO.NET and stored procedures.

---

## 🖼 The Architecture

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────────┐
│  Controller   │ ───► │     BAL       │ ───► │     DAL       │ ───► │  Stored Procedure  │
│  (HTTP layer) │      │ (business     │      │ (data access) │      │  (SQL Server)      │
│               │      │  logic)       │      │               │      │                    │
└──────────────┘      └──────────────┘      └──────────────┘      └──────────────────┘
```

---

## 💻 Full Example — All 5 Principles Applied

```csharp
// ── DAL Layer ──
public interface IOrderRepository                          // ISP + DIP: focused interface, abstraction
{
    Order GetById(int id);
    void Save(Order order);
}

public class OrderRepository : IOrderRepository             // SRP: only handles data access
{
    public Order GetById(int id)
    {
        // ADO.NET + stored procedure call
        using var conn = new SqlConnection("connection_string");
        using var cmd = new SqlCommand("sp_GetOrderById", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@OrderId", id);
        // ... execute and map to Order object
        return null;
    }

    public void Save(Order order)
    {
        // ADO.NET + stored procedure insert/update
    }
}

// ── BAL Layer ──
public interface IDiscountStrategy                          // OCP: extend via new strategies, no modification
{
    decimal Calculate(decimal amount);
}

public class RegularDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal amount) => amount * 0.05m;
}

public class OrderValidator                                  // SRP: only handles validation
{
    public bool Validate(Order order) => order.Quantity > 0;
}

public class OrderService                                    // SRP: only orchestrates business workflow
{
    private readonly IOrderRepository _repository;            // DIP: depends on abstraction
    private readonly OrderValidator _validator;
    private readonly IDiscountStrategy _discountStrategy;      // DIP + OCP: swappable strategy

    public OrderService(
        IOrderRepository repository,
        OrderValidator validator,
        IDiscountStrategy discountStrategy)                    // constructor injection
    {
        _repository = repository;
        _validator = validator;
        _discountStrategy = discountStrategy;
    }

    public void PlaceOrder(Order order)
    {
        if (!_validator.Validate(order))
            throw new ArgumentException("Invalid order");

        order.Total -= _discountStrategy.Calculate(order.Total);
        _repository.Save(order);
    }
}

// ── Controller Layer ──
public class OrderController : Controller                     // SRP: only handles HTTP concerns
{
    private readonly OrderService _orderService;

    public OrderController(OrderService orderService)          // DI: dependency injected by ASP.NET Core
    {
        _orderService = orderService;
    }

    [HttpPost]
    public IActionResult PlaceOrder(OrderViewModel model)
    {
        var order = new Order { Quantity = model.Quantity, Total = model.Total };
        _orderService.PlaceOrder(order);
        return Ok();
    }
}

// ── Program.cs — DI Container Wiring ──
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IDiscountStrategy, RegularDiscount>();
builder.Services.AddScoped<OrderValidator>();
builder.Services.AddScoped<OrderService>();
```

---

## 📊 Mapping SOLID to Each Layer

| Principle     | How it shows up in this architecture                                                                        |
| ------------- | ----------------------------------------------------------------------------------------------------------- |
| **S**RP | Each class (Controller, Service, Validator, Repository) has ONE job                                         |
| **O**CP | `IDiscountStrategy` lets you add new discount types without touching `OrderService`                     |
| **L**SP | Any`IDiscountStrategy` implementation can substitute another without breaking `OrderService`            |
| **I**SP | `IOrderRepository` only exposes what's needed — no bloated "do everything" interface                     |
| **D**IP | `OrderService` depends on `IOrderRepository`/`IDiscountStrategy` (abstractions), not concrete classes |

---

## 🚨 Common Mistakes

- ❌ Putting validation or business logic directly in the Controller — Controllers should stay thin, only handling HTTP concerns
- ❌ Instantiating `OrderRepository` directly inside `OrderService` (`new OrderRepository()`) instead of injecting `IOrderRepository`
- ❌ Skipping interfaces for DAL/BAL classes "because it's simpler" — loses testability and violates DIP

---

## 🎤 Interview Questions

| Question                                                                                | Key Point                                                                                                        |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| How does layered architecture (Controller/BAL/DAL) relate to SOLID?                     | It's a practical application of SRP (each layer has one job) and DIP (layers depend on interfaces, wired via DI) |
| Why does`OrderService` depend on `IOrderRepository` instead of `OrderRepository`? | DIP — enables testability (mock the interface) and flexibility (swap implementations)                           |
| Where should validation logic live in this architecture?                                | In the BAL (business logic layer), not the Controller or DAL                                                     |

---

## 📝 30-Second Revision Cheat Sheet

- Controller (HTTP) → BAL (business logic) → DAL (data access) → Stored Procedure
- Each layer = SRP in action | Interfaces between layers = DIP + ISP in action
- Strategy-pattern-style extensibility (like discount calculation) = OCP in action
- ASP.NET Core's built-in DI container wires it all together via constructor injectio

# SOLID in Controller BAL DAL

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
