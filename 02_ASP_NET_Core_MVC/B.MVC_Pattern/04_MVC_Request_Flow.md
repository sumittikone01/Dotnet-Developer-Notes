# 04 — MVC Request Flow

## 📌 What is it?

This topic ties together everything from `A.Fundamentals` and the first three `B.MVC_Pattern` notes into **one complete, end-to-end walkthrough** of what happens from the moment a browser sends a request to the moment it receives rendered HTML back.

## 🖼 The complete flow (ASCII diagram)

```
1. Browser sends request
   GET /Product/Details/42
         │
         ▼
2. Kestrel receives the request
         │
         ▼
3. Middleware Pipeline (see A.04)
   Exception Handling → Static Files → Routing → AuthN → AuthZ
         │
         ▼
4. Routing matches the URL to a Controller + Action
   "Product" → ProductController
   "Details" → Details(int id) action
   "42"      → id parameter
         │
         ▼
5. Model Binding
   Route value "42" → bound to the 'id' parameter (int)
         │
         ▼
6. Controller Action executes
   - Calls _productService.GetById(id)   ← delegates to Model/Service layer
   - Service queries DB, returns a Product entity
   - Controller maps Product → ProductDetailsViewModel
         │
         ▼
7. Controller returns View(viewModel)
         │
         ▼
8. Razor View Engine locates and renders the View
   Views/Product/Details.cshtml
   - Combines HTML markup + ViewModel data
   - Applies _Layout.cshtml (master page)
         │
         ▼
9. Rendered HTML is returned as the HTTP Response
         │
         ▼
10. Response flows back UP through the middleware pipeline
         │
         ▼
11. Browser receives and displays the HTML
```

## 🧠 Intuition

This is the "full circuit" — like tracing a **single water droplet** from the moment it enters a plumbing system (the request), through each valve and filter (middleware, routing, model binding), reaching the reservoir (Controller/Model), and coming back out through the tap (View → Response) to the glass (browser).

## 💻 Putting it all together — one working example

```csharp
// Program.cs (relevant excerpt)
app.UseRouting();
app.UseAuthorization();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

```csharp
// Controllers/ProductController.cs
public class ProductController : Controller
{
    private readonly IProductService _productService;
    public ProductController(IProductService productService) => _productService = productService;

    public IActionResult Details(int id)
    {
        var product = _productService.GetById(id);
        if (product == null) return NotFound();

        var vm = new ProductDetailsViewModel
        {
            ProductName = product.Name,
            FormattedPrice = product.Price.ToString("C"),
            CanAddToCart = product.IsInStock()
        };
        return View(vm); // → looks for Views/Product/Details.cshtml
    }
}
```

```html
<!-- Views/Product/Details.cshtml -->
@model ProductDetailsViewModel
<h2>@Model.ProductName</h2>
<p>@Model.FormattedPrice</p>
```

Request: `GET /Product/Details/42`

1. Routing extracts `controller = Product`, `action = Details`, `id = 42`
2. Model binder converts `"42"` (string from URL) → `42` (int)
3. `ProductController.Details(42)` executes
4. View `Details.cshtml` renders using the returned ViewModel
5. Final HTML sent back to browser

## 📊 Key checkpoints and what can go wrong at each

| Step              | What can go wrong                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| Routing           | URL doesn't match any route pattern → 404                                                              |
| Model Binding     | Route/query value type mismatch (e.g., "abc" for an`int id`) → binding fails, `ModelState` invalid |
| Controller Action | Service throws exception → bubbles up to exception-handling middleware                                 |
| View Resolution   | View file name/location doesn't match convention →`InvalidOperationException: view not found`        |
| Layout            | `_ViewStart.cshtml` missing or misconfigured → View renders without expected layout                  |

## 🚨 Common mistakes

- Assuming routing and model binding are "automatic magic" without understanding that they follow **explicit conventions** (route patterns, parameter name matching) — debugging becomes much easier once you know the exact matching rules.
- Not realizing the View resolution follows a **search path** — first the Controller-specific folder (`Views/Product/`), then falls back to `Views/Shared/` if not found there.
- Forgetting that everything after `return View(vm)` still has real execution cost — Razor compilation/rendering can throw exceptions too, which then get caught by the *outer* exception-handling middleware, not something you can `try/catch` cleanly inside the Controller.

## 💡 Best practices

- When debugging "my page isn't working," mentally walk this exact pipeline top-to-bottom — 90% of MVC bugs live at one of these checkpoints (bad route, bad binding, wrong View path, missing layout).
- Use logging (`ILogger`) at the Controller level to make each step's success/failure visible during development.

## 🎤 Interview questions

1. Walk through, step-by-step, what happens between a browser request and the rendered HTML response in ASP.NET Core MVC.
2. Where does Model Binding happen in this flow, and what does it actually do?
3. If a View can't be found, at which stage does that failure occur, and what exception is typically thrown?
4. How does the middleware pipeline relate to the MVC-specific routing/binding/action flow — are they the same thing?

## 📝 30-second revision cheat sheet

- Flow: Request → Middleware → Routing → Model Binding → Controller Action → Service/Model → ViewModel → View → Response → back through Middleware → Browser.
- Routing decides *which* Controller/Action; Model Binding decides *what data* gets passed in.
- View resolution follows a convention-based search path (`Views/{Controller}/` then `Views/Shared/`).
- Most MVC bugs map cleanly to one specific checkpoint in this pipeline — use it as a debugging checklist.
