# ✍️ Comments & Naming Conventions

## 📌 What is it?

> **Comments** — non-executable text explaining code intent.
> **Naming conventions** — consistent rules for naming variables, methods, classes, etc., so code is predictable and readable across a team.

---

## 💻 Comment Types

```csharp
// Single-line comment

/* Multi-line
   comment block */

/// <summary>
/// XML documentation comment — shows up in IntelliSense tooltips
/// </summary>
/// <param name="id">The customer ID</param>
/// <returns>Customer object or null</returns>
public Customer GetCustomer(int id) { ... }
```

| Type              | Use case                                                             |
| ----------------- | -------------------------------------------------------------------- |
| `//`            | Quick inline explanation                                             |
| `/* */`         | Temporarily disabling a code block, or longer explanations           |
| `/// <summary>` | Public API documentation — IntelliSense picks this up automatically |

---

## 📊 C# Naming Conventions (Microsoft Standard)

| Element                              | Convention                           | Example                                              |
| ------------------------------------ | ------------------------------------ | ---------------------------------------------------- |
| **Class / Interface / Method** | PascalCase                           | `class CustomerService`, `void CalculateTotal()` |
| **Interface prefix**           | `I` + PascalCase                   | `ICustomerRepository`                              |
| **Local variable / parameter** | camelCase                            | `int totalAmount`, `string customerName`         |
| **Private field**              | `_camelCase` (underscore prefix)   | `private int _orderCount;`                         |
| **Constant**                   | PascalCase                           | `const int MaxRetries = 3;`                        |
| **Property**                   | PascalCase                           | `public string FirstName { get; set; }`            |
| **Namespace**                  | PascalCase, matches folder structure | `MyApp.Services.Orders`                            |

---

## 💻 Practical Example — Naming in a real BAL/DAL layer (your stack)

```csharp
namespace MyApp.BAL
{
    public interface IOrderService              // Interface: I + PascalCase
    {
        OrderDto GetOrderById(int orderId);      // Method: PascalCase, param: camelCase
    }

    public class OrderService : IOrderService
    {
        private readonly IOrderRepository _orderRepository;  // private field: _camelCase
        private const int MaxPageSize = 50;                  // constant: PascalCase

        public OrderService(IOrderRepository orderRepository)
        {
            _orderRepository = orderRepository;
        }

        public OrderDto GetOrderById(int orderId)
        {
            // fetch order and map to DTO
            return _orderRepository.GetById(orderId);
        }
    }
}
```

---

## 🚨 Common Mistakes

- ❌ Inconsistent casing across a team (`orderID` vs `OrderId` vs `order_id`) — pick PascalCase/camelCase per Microsoft convention and stick to it
- ❌ Over-commenting obvious code (`// increment i` above `i++`) — comments should explain **why**, not restate **what**
- ❌ Under-commenting complex business logic — always explain non-obvious "why" decisions (e.g., "// skipping tax for international orders per policy X")
- ❌ Leaving commented-out dead code in the codebase — delete it; source control preserves history

---

## 🎤 Interview Questions

| Question                                           | Key Point                                                                                  |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| What's the difference between`//` and `///`?   | `///` generates XML documentation used by IntelliSense/API docs                          |
| Why prefix private fields with`_`?               | Distinguishes fields from local variables/parameters at a glance, avoids naming collisions |
| What naming convention do interfaces follow in C#? | `I` prefix + PascalCase, e.g., `IDisposable`, `IOrderService`                        |

---

## 📝 30-Second Revision Cheat Sheet

- Classes/Methods/Properties → **PascalCase** | Locals/Parameters → **camelCase** | Private fields → **_camelCase**
- Interfaces → `I` + PascalCase
- `///` → XML doc comments (IntelliSense) | `//` → inline notes
- Comments explain **why**, not **what** — code should already show "wha
