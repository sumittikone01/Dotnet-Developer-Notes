    

# 📁 Namespaces & Using Directives

## 📌 What is it?

> **Namespace** — a logical container that organizes related classes/types and prevents naming collisions.
> **`using` directive** — imports a namespace so you can reference its types without fully qualifying them.

---

## 🌍 Real-world analogy

Think of namespaces like **folders on a computer**. You can have two files named `report.docx` as long as they're in different folders (`/2024/report.docx` vs `/2025/report.docx`). Similarly, two classes can both be named `Order` if they live in different namespaces (`Sales.Order` vs `Inventory.Order`).

---

## 🧠 Intuition

```csharp
namespace MyApp.Sales
{
    public class Order { }
}

namespace MyApp.Inventory
{
    public class Order { }   // ✅ No conflict — different namespace
}

// To use them, fully qualify OR import via 'using':
MyApp.Sales.Order salesOrder = new MyApp.Sales.Order();

using MyApp.Sales;
Order salesOrder2 = new Order();   // now shorthand works
```

---

## ⚙️ Types of `using`

| Syntax                                 | Purpose                                                                           |
| -------------------------------------- | --------------------------------------------------------------------------------- |
| `using System;`                      | Standard namespace import                                                         |
| `using System.Collections.Generic;`  | Import for generic collections (`List<T>`, `Dictionary<T>`)                   |
| `using Alias = Full.Namespace.Path;` | Create a shorter alias for a long/conflicting namespace                           |
| `global using System;` (C# 10+)      | Applies the`using` to **every file** in the project — reduces repetition |
| `using static System.Math;`          | Import static members directly — call`Sqrt(x)` instead of `Math.Sqrt(x)`     |

---

## 💻 Code Examples

**Basic:**

```csharp
using System;
using System.Collections.Generic;

List<string> names = new List<string>();  // works because of the using directive above
Console.WriteLine("Hello");
```

**Intermediate — Alias for conflict resolution:**

```csharp
using SalesOrder = MyApp.Sales.Order;
using InventoryOrder = MyApp.Inventory.Order;

SalesOrder so = new SalesOrder();
InventoryOrder io = new InventoryOrder();
```

**Practical — Typical top of an ASP.NET Core MVC Controller (your daily code):**

```csharp
using Microsoft.AspNetCore.Mvc;
using MyApp.BAL.Interfaces;
using MyApp.Models.ViewModels;
using System.Data;
using System.Data.SqlClient;   // ADO.NET connection handling

namespace MyApp.Controllers
{
    public class OrderController : Controller
    {
        private readonly IOrderService _orderService;

        public OrderController(IOrderService orderService)
        {
            _orderService = orderService;
        }
    }
}
```

**global usings (C# 10+, modern .NET projects):**

```csharp
// In a file like GlobalUsings.cs, applies project-wide:
global using System;
global using System.Collections.Generic;
global using System.Linq;
```

> 💡 Many modern ASP.NET Core project templates auto-generate an `<ImplicitUsings>enable</ImplicitUsings>` setting in the `.csproj`, which auto-includes common namespaces without you writing `using` everywhere.

---

## 🚨 Common Mistakes

- ❌ Overusing `using static` — can make code confusing since it's unclear where a method came from
- ❌ Forgetting to import a namespace and getting confused by "type or namespace not found" errors — IDE usually offers a quick-fix
- ❌ Deeply nested namespace hierarchies that don't match folder structure — makes navigation harder (convention: namespace should mirror folder path)

---

## 🎤 Interview Questions

| Question                               | Key Point                                                                               |
| -------------------------------------- | --------------------------------------------------------------------------------------- |
| What is the purpose of a namespace?    | Organizes code and prevents naming collisions between types with the same name          |
| What does`global using` do (C# 10+)? | Applies the`using` directive to all files in the project, reducing repetitive imports |
| Can two classes have the same name?    | Yes, if they're in different namespaces                                                 |

---

## 📝 30-Second Revision Cheat Sheet

- Namespace = logical folder for types, avoids naming collisions
- `using` = import shorthand so you don't fully qualify every type
- `global using` (C# 10+) = project-wide import, less repetition
- Convention: namespace structure should mirror your folder structure
