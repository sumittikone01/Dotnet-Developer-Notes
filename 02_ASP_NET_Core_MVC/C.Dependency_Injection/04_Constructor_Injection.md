# 04 — Constructor Injection

## 📌 What is it?

**Constructor injection** is the primary (and recommended) way to receive dependencies in ASP.NET Core — a class declares what it needs as **constructor parameters**, and the DI container automatically supplies them when creating an instance.

## 🤔 Why do we need it?

Constructor injection makes dependencies **explicit and mandatory** — you can't create the class without providing what it needs, which prevents "half-initialized" objects and makes the class's requirements obvious just by reading its constructor signature.

## 💻 Code example

```csharp
public class OrderController : Controller
{
    private readonly IOrderService _orderService;
    private readonly ILogger<OrderController> _logger;
    private readonly IEmailSender _emailSender;

    // Constructor injection — the DI container supplies all three automatically
    public OrderController(
        IOrderService orderService,
        ILogger<OrderController> logger,
        IEmailSender emailSender)
    {
        _orderService = orderService;
        _logger = logger;
        _emailSender = emailSender;
    }

    public IActionResult Checkout(int orderId)
    {
        var order = _orderService.GetById(orderId);
        _logger.LogInformation("Processing checkout for order {OrderId}", orderId);
        _emailSender.SendConfirmation(order);
        return View(order);
    }
}
```

You never manually call `new OrderController(...)` — ASP.NET Core's DI container does this automatically whenever a request needs to be routed to this Controller.

## 🖼 What happens behind the scenes

```
1. Request comes in, routing determines: OrderController.Checkout() should handle it
                     │
                     ▼
2. DI container inspects OrderController's constructor
                     │
                     ▼
3. For each parameter, container looks up its registration:
   IOrderService        → registered as OrderService (Scoped)
   ILogger<OrderController> → framework-provided logger
   IEmailSender          → registered as SmtpEmailSender (Transient)
                     │
                     ▼
4. Container creates (or reuses, per lifetime rules) each dependency
                     │
                     ▼
5. Container calls: new OrderController(orderService, logger, emailSender)
                     │
                     ▼
6. Checkout() executes with everything it needs, fully ready
```

## ⚙️ Constructor injection vs other injection styles

| Style                                           | How it works                                            | When used                                                                    |
| ----------------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Constructor injection**                 | Dependencies passed via constructor parameters          | Default, recommended for almost everything                                   |
| **Method injection** (`[FromServices]`) | A specific action method parameter is resolved from DI  | Rare — for a dependency only needed in ONE action, not the whole Controller |
| **Property injection**                    | Dependency set via a public property after construction | Rare in ASP.NET Core; more common in some other DI frameworks                |

```csharp
// Method injection example — dependency only needed for this one action
public IActionResult Export([FromServices] IReportGenerator reportGenerator)
{
    return File(reportGenerator.Generate(), "application/pdf");
}
```

## 📊 Why constructor injection is preferred

| Benefit                         | Explanation                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Explicit dependencies** | Reading the constructor tells you exactly what the class needs — no hidden surprises                          |
| **Immutability**          | Dependencies assigned to`readonly` fields — can't be swapped out accidentally later                         |
| **Fail-fast**             | If a required dependency can't be resolved, the app fails immediately at startup/first use, not silently later |
| **Easy testing**          | Unit tests just pass mock implementations directly into the constructor — no DI container needed for tests    |

## 💻 Testability example

```csharp
[Fact]
public void Checkout_LogsInformation()
{
    var mockOrderService = new Mock<IOrderService>();
    var mockLogger = new Mock<ILogger<OrderController>>();
    var mockEmailSender = new Mock<IEmailSender>();

    var controller = new OrderController(
        mockOrderService.Object,
        mockLogger.Object,
        mockEmailSender.Object);

    var result = controller.Checkout(1);

    // Assertions on mock behavior...
}
```

Because `OrderController` receives everything via its constructor, testing it requires **no DI container at all** — just pass mocks directly.

## 🚨 Common mistakes

- Adding too many constructor parameters (5+) — a sign the class is doing too much (violates Single Responsibility Principle); consider splitting the class or introducing a facade service.
- Using `new SomeService()` inside a Controller instead of injecting `ISomeService` — reintroduces tight coupling and breaks testability, defeating the purpose of DI entirely.
- Forgetting that constructor injection requires **all** listed dependencies to be resolvable — if even one can't be resolved, the *entire* Controller fails to instantiate (fail-fast, but can be confusing if the error message points to an unrelated dependency).

## 💡 Best practices

- Prefer constructor injection for nearly everything — reserve `[FromServices]` method injection only for dependencies used in a single action, not throughout the Controller.
- Keep constructors focused — if a Controller/class needs many dependencies, it's often a sign to extract a coordinating Service that itself takes some of those dependencies.
- Always assign injected dependencies to `readonly` fields — signals intent (never reassigned) and prevents accidental mutation.

## 🎤 Interview questions

1. Why is constructor injection preferred over property injection in ASP.NET Core?
2. How does constructor injection improve unit testability compared to manually instantiating dependencies inside a class?
3. What would you do if a Controller's constructor started needing 8+ dependencies?
4. What's the difference between constructor injection and using `[FromServices]` on an action parameter?

## 📝 30-second revision cheat sheet

- Constructor injection = dependencies declared as constructor parameters, auto-supplied by the DI container.
- Preferred over property/method injection for almost all cases — explicit, immutable, fail-fast, easy to test.
- Too many constructor parameters = code smell → consider splitting responsibilities.
- `[FromServices]` method injection is a rare exception, for single-action-only dependencies.
