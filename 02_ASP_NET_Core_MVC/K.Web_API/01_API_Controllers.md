# 01_API_Controllers

> **API Controllers** = Controllers designed specifically to return DATA (JSON/XML) to callers — mobile apps, JavaScript frontends (like your team's Kendo UI), or other services — rather than rendering HTML Views like a traditional MVC Controller.

> New chapter: **K.Web_API**. Everything in earlier chapters (Controllers, Model Handling, Filters) applies to API Controllers too — this chapter is specifically about what makes an API Controller distinct from a View-rendering MVC Controller.

## 📌 What is it?

```csharp
// Traditional MVC Controller — returns a VIEW (HTML)
public class ProductsController : Controller
{
    public IActionResult Index()
    {
        var products = _service.GetAllProducts();
        return View(products); // renders a .cshtml Razor view
    }
}

// API Controller — returns DATA (JSON)
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = _service.GetAllProducts();
        return Ok(products); // serializes to JSON automatically
    }
}
```

## 🤔 Why do we need a distinct API Controller style?

| Traditional MVC need                                | API need                                                                         |
| --------------------------------------------------- | -------------------------------------------------------------------------------- |
| Render HTML for a browser to display                | Return structured data (JSON) for a client to consume/process                    |
| Redirect to another page after a POST               | Return a status code + data (no redirects — the CLIENT decides what to do next) |
| Use`ViewData`/`ViewBag`/Model binding for Razor | Use strongly-typed request/response DTOs                                         |
| Session-based, cookie-authenticated typically       | Often token-based (JWT) authentication, since callers aren't always browsers     |

In your team's stack, this distinction matters because your ASP.NET Core MVC app likely serves BOTH: Razor Views for full pages, AND API endpoints that your Kendo UI Grid/widgets call via AJAX to fetch/save data.

## 🌍 Real-world analogy

A traditional MVC Controller is like a **waiter who brings you a fully plated, ready-to-eat meal** (a rendered HTML page). An API Controller is like a **butler who hands you the raw ingredients in labeled containers** (structured JSON data) — YOU (the client-side JavaScript/Kendo widget) decide how to "cook" and present it on the page.

## ⚙️ Internal working — `Controller` vs `ControllerBase`

```
Controller (MVC base class)
    │
    ├── Inherits from ControllerBase
    │
    └── ADDS View-rendering capabilities: View(), PartialView(), ViewBag, ViewData, TempData

ControllerBase (API base class)
    │
    └── Has EVERYTHING needed for APIs: Ok(), NotFound(), BadRequest(), CreatedAtAction(),
        ModelState, HttpContext, User, Request/Response — but NO View() method
```

```csharp
public class ProductsController : Controller { }        // for VIEWS — has access to View()
public class ProductsApiController : ControllerBase { } // for APIs — leaner, no View-related baggage
```

> **Rule of thumb:** if a Controller will NEVER return a View, inherit from `ControllerBase` — it's the more accurate, leaner choice and signals clear intent to anyone reading the code.

## 📊 Common Action Result Types for API Controllers

| Method                              | Returns                                     | HTTP Status          |
| ----------------------------------- | ------------------------------------------- | -------------------- |
| `Ok(data)`                        | The data, serialized to JSON                | 200                  |
| `NotFound()` / `NotFound(data)` | Optional error data                         | 404                  |
| `BadRequest(ModelState)`          | Validation errors                           | 400                  |
| `CreatedAtAction(...)`            | The created resource + a`Location` header | 201                  |
| `NoContent()`                     | Nothing                                     | 204                  |
| `Unauthorized()`                  | Nothing                                     | 401                  |
| `Forbid()`                        | Nothing                                     | 403                  |
| `StatusCode(code, data)`          | Custom status code + optional data          | Whatever you specify |

> Full status-code semantics were covered in `12_System_Design/I.API_Design/02_HTTP_Methods_and_Status_Codes.md` — this table is the ASP.NET Core-specific "how to actually return each one."

## 💻 Code examples

### Basic — a typical API Controller following your team's Controller → BAL → DAL flow

```csharp
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    private readonly ProductService _service; // BAL

    public ProductsApiController(ProductService service) => _service = service;

    [HttpGet]
    public IActionResult GetAll() => Ok(_service.GetAllProducts());

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id); // BAL → DAL → Stored Procedure
        return product == null ? NotFound() : Ok(product);
    }

    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);

        var created = _service.CreateProduct(model);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

### Intermediate — an API Controller backing a Kendo Grid (matches your stack exactly)

```csharp
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpGet]
    public IActionResult GetGridData([FromQuery] DataSourceRequest request)
    {
        var products = _service.GetAllProducts();
        var result = products.ToDataSourceResult(request); // Kendo-specific paging/sorting/filtering result
        return Ok(result);
    }

    [HttpPost("update")]
    public IActionResult UpdateGridRow([DataSourceRequest] DataSourceRequest request, ProductViewModel model)
    {
        if (ModelState.IsValid)
        {
            _service.UpdateProduct(model);
        }
        return Ok(new[] { model }.ToDataSourceResult(request, ModelState));
    }
}
```

### Practical — a Controller serving BOTH Views and API endpoints (common in a real MVC app)

```csharp
// A Controller CAN mix View-returning and data-returning actions, but it's cleaner to separate concerns:
public class ProductsController : Controller  // inherits Controller — needs View()
{
    [HttpGet]
    public IActionResult Index() => View(); // renders the page shell (Kendo Grid lives here)

    [HttpGet]
    [Route("api/products/grid-data")] // a "hybrid" action returning JSON from a View-capable controller
    public IActionResult GetGridData([FromQuery] DataSourceRequest request)
    {
        var result = _service.GetAllProducts().ToDataSourceResult(request);
        return Ok(result); // still works fine even though this Controller inherits Controller, not ControllerBase
    }
}
```

## ⚡ Performance considerations

- `ControllerBase` vs `Controller` has no meaningful performance difference at runtime — the choice is about CODE CLARITY (signaling "this never returns a View"), not speed.
- JSON serialization (via `System.Text.Json` by default in modern ASP.NET Core) is efficient out of the box — avoid manually serializing to strings and returning `ContentResult` unless you have a specific reason to.

## 🚨 Common mistakes

- ❌ Inheriting from `Controller` for a pure API Controller that never returns a View — adds unnecessary View-related surface area and can be confusing about the Controller's actual purpose.
- ❌ Returning raw domain/database entities directly instead of DTOs/ViewModels — leaks internal structure (and potentially sensitive fields) directly into the API's public contract.
- ❌ Manually setting status codes with magic numbers (`StatusCode(200, ...)`) instead of the more readable, dedicated helper methods (`Ok(...)`).
- ❌ Forgetting `[ApiController]` on a pure API controller — loses several useful automatic behaviors covered in `02_ApiController_Attribute.md`.

## 💡 Best practices

- ✅ Inherit from `ControllerBase` (not `Controller`) for any Controller that only returns data, never Views.
- ✅ Use the dedicated result helper methods (`Ok`, `NotFound`, `CreatedAtAction`, etc.) rather than manually constructing status codes.
- ✅ Keep API Controllers thin — delegate all real logic to the BAL, exactly as your team's architecture already does; the Controller's job is just translating HTTP concerns (routes, status codes) to/from your business logic.
- ✅ Return DTOs/ViewModels, not raw database entities, from API actions.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                           | Answer                                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between`Controller` and `ControllerBase`?                                | `Controller` inherits from `ControllerBase` and adds View-rendering capabilities (`View()`, `ViewBag`, etc.); `ControllerBase` has everything needed for APIs, without the View-related additions |
| Which base class should a pure API Controller use?                                                 | `ControllerBase` — it's the leaner, more accurate choice when the Controller never returns a View                                                                                                        |
| What does`CreatedAtAction(...)` do, and why use it over `Ok(...)` for a POST?                  | Returns a 201 status code with a`Location` header pointing to the newly created resource — the correct REST semantics for successful creation                                                            |
| Why avoid returning raw database entities directly from an API action?                             | It leaks internal structure/fields into the public API contract; DTOs/ViewModels give you control over exactly what's exposed                                                                               |
| Can a Controller that inherits from`Controller` (not `ControllerBase`) still return JSON data? | Yes —`Controller` includes everything `ControllerBase` has, plus View support; it can still return `Ok(data)` etc. just fine                                                                         |

## 📝 30-second Revision Cheat Sheet

- API Controllers return DATA (JSON); traditional MVC Controllers return VIEWS (HTML).
- `ControllerBase` = API-focused (no View support); `Controller` = `ControllerBase` + View-rendering capabilities.
- Use dedicated result helpers: `Ok`, `NotFound`, `BadRequest`, `CreatedAtAction`, `NoContent`.
- Return DTOs/ViewModels, never raw database entities, from API actions.
- Your team's stack often mixes both — a Kendo-Grid-backed page uses a View-returning action for the shell, plus JSON-returning actions for grid data.
