# 03 — Controller Responsibilities

## 📌 What is it?

The **Controller** handles incoming HTTP requests, coordinates between the Model and View, and returns a response. It's the "traffic cop" — it doesn't do the heavy lifting itself, it delegates and orchestrates.

## 🤔 Why do we need it?

Without a clear Controller layer, request-handling logic (parsing input, calling business logic, choosing a response) would be scattered or mixed with either the UI or the data layer — breaking separation of concerns.

## 🧠 Intuition

The Controller is like a **receptionist/dispatcher**: it receives the request, figures out what's being asked, calls the right department (Service/Model) to do the actual work, and sends back the appropriate response (View or data) — without doing the department's job itself.

## 💻 Code example — a well-structured "thin" Controller

```csharp
public class ProductController : Controller
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;

    public ProductController(IProductService productService, ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Details(int id)
    {
        var product = _productService.GetById(id);   // delegate to Service
        if (product == null)
        {
            _logger.LogWarning("Product {Id} not found", id);
            return NotFound();
        }

        var viewModel = MapToViewModel(product);       // shape data for the View
        return View(viewModel);                          // hand off to View
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(ProductCreateViewModel model)
    {
        if (!ModelState.IsValid)                          // validate input
            return View(model);

        _productService.Create(model);                    // delegate business logic
        return RedirectToAction(nameof(Details), new { id = model.Id });
    }

    private ProductDetailsViewModel MapToViewModel(Product product) =>
        new()
        {
            ProductName = product.Name,
            FormattedPrice = product.Price.ToString("C"),
            CanAddToCart = product.IsInStock()
        };
}
```

Notice everything the Controller does here: **receive request → validate input → delegate to service → shape response → return result**. It never computes business rules itself (e.g., stock calculations) or touches the database directly.

## ⚙️ Core Controller responsibilities

| Responsibility                              | Example                                                      |
| ------------------------------------------- | ------------------------------------------------------------ |
| Receive and parse HTTP request data         | Route values, query strings, form data, JSON body            |
| Validate input                              | `ModelState.IsValid`, Data Annotations                     |
| Delegate to business/service layer          | Call`_productService.GetById(id)`                          |
| Choose the appropriate response             | `View()`, `Ok()`, `NotFound()`, `RedirectToAction()` |
| Handle cross-cutting request-level concerns | Authorization checks (often via attributes), logging         |

## 🚨 What does NOT belong in a Controller

- Direct database queries (`_dbContext.Products.Where(...)`) — belongs in a repository/service
- Complex business rule calculations — belongs on the Model or in a Service
- HTML generation — that's the View's job entirely

## 📊 "Fat Controller" vs "Thin Controller"

| Fat Controller (anti-pattern)                          | Thin Controller (best practice)                 |
| ------------------------------------------------------ | ----------------------------------------------- |
| Contains DB queries directly                           | Delegates to a Service/Repository               |
| Contains complex business rule branching               | Business rules live in Model/Service layer      |
| Hard to unit test (needs a real DB)                    | Easy to unit test (mock the Service)            |
| Logic duplicated if reused elsewhere (e.g., in an API) | Logic reusable across MVC, API, background jobs |

## 🚨 Common mistakes

- Writing LINQ-to-Entities queries directly inside a Controller action — tightly couples the Controller to the database and makes testing painful.
- Returning inconsistent result types for similar actions (sometimes `View()`, sometimes raw strings, sometimes throwing exceptions) — keep response conventions consistent.
- Forgetting `[ValidateAntiForgeryToken]` on POST actions that modify data — leaves the app open to CSRF attacks (covered in depth in `N.Security`).

## 💡 Best practices

- Keep Controllers thin: **receive → validate → delegate → respond**.
- Inject Services (never `DbContext` directly, ideally) via constructor injection.
- Use `IActionResult` return type when an action can return multiple different result types (`View`, `NotFound`, `Redirect`, etc.); use `ActionResult<T>` for typed API responses (compared in detail in `E.Controllers/03`).

## 🎤 Interview questions

1. What are the core responsibilities of a Controller, and what should explicitly NOT live there?
2. What's a "fat controller" and why is it considered an anti-pattern?
3. Why does keeping Controllers thin improve testability?
4. Walk through what happens step-by-step inside a well-structured Controller action, from request to response.

## 📝 30-second revision cheat sheet

- Controller = receive request → validate → delegate to Service → choose response.
- Should NOT contain DB queries or complex business logic — that's a "fat controller" anti-pattern.
- Thin Controllers are easier to test, reuse, and maintain.
- Constructor-inject Services, not `DbContext`, for clean separation.
