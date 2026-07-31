# 01 — Dependency Injection Overview

## 📌 What is it?

**Dependency Injection (DI)** is a design pattern where a class **receives its dependencies from the outside** (via constructor, method, or property) instead of creating them itself. ASP.NET Core has DI **built into the framework core** — it's not an optional add-on like it was in ASP.NET Framework (where you needed Ninject, Unity, Autofac, etc.).

## 🤔 Why do we need it?

Without DI, classes create their own dependencies directly, which causes:

- **Tight coupling** — hard to swap implementations (e.g., switch from SQL to a mock for testing)
- **Poor testability** — can't easily substitute a fake/mock dependency
- **Duplicated object creation logic** scattered everywhere

## 🧠 Intuition

Think of a **restaurant kitchen**: instead of every chef growing their own vegetables (creating their own dependencies), a supplier delivers exactly what's needed to each station (constructor injection). Chefs focus on cooking (business logic), not farming (object construction).

## 💻 Code example — without vs with DI

### ❌ Without DI (tight coupling)

```csharp
public class ProductController : Controller
{
    private readonly ProductService _productService;

    public ProductController()
    {
        _productService = new ProductService(new SqlProductRepository()); // hardcoded!
    }
}
```

Problem: `ProductController` is now permanently tied to `SqlProductRepository`. Testing requires a real database connection. Swapping implementations means editing this class directly.

### ✅ With DI (loose coupling)

```csharp
public class ProductController : Controller
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService) // injected!
    {
        _productService = productService;
    }
}
```

```csharp
// Program.cs — registration
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, SqlProductRepository>();
```

Now `ProductController` depends only on the **abstraction** (`IProductService`), not a specific implementation. Swapping to a different implementation (or a mock for unit tests) requires **zero changes** to the Controller.

## 🖼 How the DI container works (conceptually)

```
┌─────────────────────────────┐
│   DI Container (built-in)     │
│                              │
│  IProductService  → ProductService     │
│  IProductRepository → SqlProductRepository │
│  ILogger<T>       → (framework-provided)  │
└──────────────┬──────────────┘
               │ "I need an IProductService"
               ▼
      ┌──────────────────┐
      │ ProductController │  ← DI container automatically
      │  (constructor)     │     resolves & injects dependencies
      └──────────────────┘
```

When ASP.NET Core needs to create a `ProductController` to handle a request, it inspects the constructor, sees it needs an `IProductService`, looks up the registered implementation, creates (or reuses) an instance, and passes it in — all automatically.

## ⚙️ The three parts of DI in ASP.NET Core

1. **Service** — the class providing functionality (`ProductService`)
2. **Registration** — telling the container which implementation to use for an interface (`builder.Services.AddScoped<IProductService, ProductService>()`)
3. **Injection** — the container automatically supplying the dependency wherever it's needed (usually via constructor parameters)

## 📊 DI in ASP.NET Framework vs ASP.NET Core

| Aspect                           | ASP.NET Framework                                     | ASP.NET Core                                 |
| -------------------------------- | ----------------------------------------------------- | -------------------------------------------- |
| Built-in DI container            | ❌ No — required 3rd party (Ninject, Unity, Autofac) | ✅ Yes, built into the framework             |
| Where you register services      | Custom bootstrapper code, varies by library           | `Program.cs` (`builder.Services.Add...`) |
| Consistency across the framework | Varies by 3rd-party library conventions               | Standardized`IServiceCollection` API       |

## 🚨 Common mistakes

- Still manually `new`-ing up dependencies inside Controllers/Services out of old habits — defeats the purpose of having DI at all.
- Forgetting to **register** a service before trying to inject it — results in a runtime `InvalidOperationException: Unable to resolve service for type...`.
- Depending directly on **concrete classes** instead of interfaces in constructors — loses the flexibility DI is meant to provide.

## 💡 Best practices

- Always depend on **interfaces/abstractions** in constructors, not concrete implementations.
- Register every service that a Controller/class needs — DI won't "guess"; every dependency must be explicitly wired up.
- Use DI consistently across the whole app — mixing manual instantiation and DI leads to confusing, inconsistent object lifetimes.

## 🎤 Interview questions

1. What problem does Dependency Injection solve, and how does it improve testability?
2. Why does ASP.NET Core come with DI built-in, unlike the older ASP.NET Framework?
3. What are the three core parts of using DI (service, registration, injection)?
4. What happens if you try to inject a service that was never registered in `Program.cs`?

## 📝 30-second revision cheat sheet

- DI = classes receive dependencies from outside instead of creating them internally.
- Built into ASP.NET Core (`IServiceCollection`) — no 3rd party library needed.
- Always depend on interfaces, not concrete classes, in constructors.
- Three steps: define service → register it (`builder.Services.Add...`) → inject it (constructor parameter).
