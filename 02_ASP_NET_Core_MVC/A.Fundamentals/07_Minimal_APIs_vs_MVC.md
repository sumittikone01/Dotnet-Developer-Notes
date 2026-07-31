
# 07 — Minimal APIs vs MVC

## 📌 What is it?

**Minimal APIs** (introduced in .NET 6) are a lightweight way to build HTTP APIs with minimal boilerplate — no Controllers, no attributes, endpoints defined directly in `Program.cs` (or extracted to small extension methods).

## 🤔 Why do we need it?

Traditional MVC/Web API Controllers require a fair amount of ceremony (class, base class, attributes, action methods) even for a simple endpoint. Minimal APIs exist for cases where that ceremony is unnecessary overhead — microservices, small APIs, prototypes, serverless functions.

## 📊 Side-by-side comparison

| Aspect                            | MVC / Web API Controllers                               | Minimal APIs                                   |
| --------------------------------- | ------------------------------------------------------- | ---------------------------------------------- |
| Boilerplate                       | More (class,`[ApiController]`, action methods)        | Very little (lambda per endpoint)              |
| Best for                          | Full apps, complex APIs, large teams, Views             | Small APIs, microservices, quick endpoints     |
| Filters (Action Filters etc.)     | Fully supported                                         | Supported via Endpoint Filters (different API) |
| Model binding                     | Automatic, rich (`[FromBody]`, `[FromQuery]`, etc.) | Also supported, slightly more explicit         |
| Razor View support                | Yes                                                     | No (API-only, no HTML views)                   |
| Testability                       | Well-established patterns (mock Controllers)            | Newer patterns, less tooling maturity          |
| Discoverability in large codebase | High (organized by Controller class per resource)       | Can get messy if not organized carefully       |
| Startup performance               | Slightly heavier                                        | Slightly faster/lighter                        |

## 💻 Code example — same endpoint, both styles

### MVC / Web API Controller style

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _productService.GetById(id);
        if (product == null) return NotFound();
        return Ok(product);
    }
}
```

### Minimal API style

```csharp
var app = builder.Build();

app.MapGet("/api/products/{id}", (int id, IProductService productService) =>
{
    var product = productService.GetById(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});

app.Run();
```

Notice: no class, no attributes, no base class — just a route + a lambda. Dependency injection still works (`IProductService` is auto-resolved as a parameter).

## 🧠 Intuition

MVC Controllers are like **furnished apartments** — more setup, but you get rooms (methods) organized by purpose, ready for many tenants (endpoints) over time.

Minimal APIs are like **a single-purpose kiosk** — quick to set up, perfect for one specific job, but doesn't scale well if you keep bolting more and more onto it.

## 🖼 When to choose which

```
Need Razor Views / server-rendered HTML?  ──▶ Use MVC
Building a large, structured REST API
  with many resources & complex logic?     ──▶ Use MVC/Web API Controllers
Building a small microservice,
  a handful of endpoints, or prototyping?  ──▶ Use Minimal APIs
Need heavy use of Filters/Conventions?     ──▶ Use MVC Controllers (more mature)
```

## 🚨 Common mistakes

- Assuming Minimal APIs replace MVC entirely — they don't; MVC is still the standard for apps with Views or complex API surfaces.
- Cramming a large API (30+ endpoints) into a single `Program.cs` file using Minimal APIs without organizing into extension methods/route groups — becomes unmaintainable.
- Forgetting that Minimal APIs use **Endpoint Filters** (`app.MapGet(...).AddEndpointFilter(...)`), not the same `IActionFilter` used in MVC.

## 💡 Best practices

- For small, focused microservices → Minimal APIs, organized using **Route Groups** (`app.MapGroup("/api/products")`) to keep `Program.cs` clean.
- For anything with server-rendered Views, or a large complex API → stick with MVC Controllers; the ecosystem (filters, model binding conventions, tooling) is more mature.
- Given your ASP.NET/Kendo UI/AJAX background (server-rendered views + AJAX calls back to Controllers), **MVC Controllers will likely remain your primary tool** — but knowing Minimal APIs helps you read newer .NET codebases and choose the right tool for small API-only projects.

## 🎤 Interview questions

1. What's the core architectural difference between Minimal APIs and MVC Controllers?
2. In what scenario would Minimal APIs clearly be the better choice over full MVC?
3. How does dependency injection work in a Minimal API endpoint compared to a Controller?
4. What's the Minimal API equivalent of an MVC Action Filter?

## 📝 30-second revision cheat sheet

- Minimal APIs = lightweight, no Controller class, endpoints as lambdas in `Program.cs`.
- MVC Controllers = more structure, needed for Views, large APIs, mature filter/tooling support.
- Both support DI, routing, model binding — Minimal APIs use **Endpoint Filters** instead of Action Filters.
- Choose based on scale/complexity: small API → Minimal; full app with Views or complex logic → MVC.
