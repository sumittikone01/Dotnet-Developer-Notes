# 03 — MVC Architecture

## 📌 What is it?

**MVC (Model-View-Controller)** is a software design pattern that splits an application into three interconnected components, each with a single responsibility.

| Component            | Responsibility                                                              |
| -------------------- | --------------------------------------------------------------------------- |
| **Model**      | Data + business logic (what the app "knows")                                |
| **View**       | Presentation/UI (what the user "sees")                                      |
| **Controller** | Handles input, coordinates Model and View (what "responds" to user actions) |

## 🤔 Why do we need it?

Without separation of concerns, web apps become a tangled mess — HTML mixed with SQL mixed with business rules in one file (classic "spaghetti code," common in old-style ASP pages).

MVC solves this by enforcing **separation of concerns**:

- Easier to **test** (Controllers/Models can be unit tested without a browser)
- Easier to **maintain** (a UI designer can edit Views without touching business logic)
- Easier to **scale a team** (different devs can work on Model, View, Controller independently)

## 🧠 Intuition

Think of MVC as **division of labor** in a restaurant:

- **Model** = the kitchen's ingredients & recipes (the data and rules)
- **View** = the plate presentation (how the food looks to the customer)
- **Controller** = the waiter (takes the order/request, tells the kitchen what to prepare, brings back the result)

The waiter never cooks. The kitchen never talks to the customer directly. Each role stays in its lane.

## 🖼 MVC Request Flow (ASCII diagram)

```
        ┌─────────────┐
 Request│             │
───────▶│  Controller  │
        │             │
        └──────┬──────┘
               │ 1. Calls Model for data
               ▼
        ┌─────────────┐
        │    Model     │  (business logic + data access)
        └──────┬──────┘
               │ 2. Returns data
               ▼
        ┌─────────────┐
        │  Controller  │
        └──────┬──────┘
               │ 3. Passes data to View
               ▼
        ┌─────────────┐
        │     View     │  (Razor .cshtml — renders HTML)
        └──────┬──────┘
               │ 4. Rendered HTML
               ▼
        ┌─────────────┐
 Response│             │
◀────────│   Browser    │
        └─────────────┘
```

## 💻 Code example (Basic)

```csharp
// Model
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Controller
public class ProductController : Controller
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService)
    {
        _productService = productService; // injected via DI
    }

    public IActionResult Details(int id)
    {
        Product product = _productService.GetById(id); // talk to Model
        return View(product);                           // pass to View
    }
}
```

```html
<!-- View: Details.cshtml -->
@model Product

<h2>@Model.Name</h2>
<p>Price: @Model.Price.ToString("C")</p>
```

Notice: the Controller doesn't know *how* HTML is rendered, and the View doesn't know *where* the data came from (database, API, cache). That's the separation in action.

## 📊 Comparison: MVC vs Web Forms (since you have that background)

| Aspect                 | Web Forms                                   | MVC                                              |
| ---------------------- | ------------------------------------------- | ------------------------------------------------ |
| Control over HTML      | Low (server controls generate HTML for you) | Full (you write the HTML/Razor yourself)         |
| State management       | ViewState (heavy, page-level)               | Stateless, explicit (TempData/Session as needed) |
| Testability            | Hard (tightly coupled to page lifecycle)    | Easy (Controllers are plain classes)             |
| Separation of concerns | Weak (code-behind mixes UI + logic)         | Strong (enforced by pattern)                     |

## 🚨 Common mistakes

- Putting business logic directly inside Controllers (Controllers should **delegate**, not **implement** business rules — that belongs in a Service/Model layer).
- Putting data-access code inside Views (Views should only handle presentation).
- Treating the "Model" as only referring to database entities — it's a broader term for the entire data + business logic layer.

## 💡 Best practices

- Keep Controllers **thin** — just orchestrate calls to services/repositories.
- Keep business logic in a separate **Service layer**, not in Controllers or Views.
- Use **ViewModels** (not raw database entities) to shape exactly what a View needs — covered later in `G.Data_Passing`.

## 🎤 Interview questions

1. What problem does MVC solve compared to a monolithic page-based approach (like classic ASP or Web Forms)?
2. In MVC, which component should contain business logic — and why is it a common mistake to put it in the Controller?
3. Walk through the full request lifecycle in MVC from browser request to rendered response.
4. Why is MVC considered more testable than Web Forms?

## 📝 30-second revision cheat sheet

- **Model** = data + business logic, **View** = UI, **Controller** = traffic cop between them.
- Flow: Request → Controller → Model → Controller → View → Response.
- Enforces separation of concerns → better testability, maintainability, team scalability.
- Controllers should stay **thin**; logic belongs in a Service layer.
