
# 09_API_Versioning

> The ASP.NET Core-specific IMPLEMENTATION of the versioning strategy already covered conceptually in `12_System_Design/I.API_Design/03_API_Versioning.md` — using the `Asp.Versioning` package to actually wire up multiple live API versions in a real Web API project.

> Continues **K.Web_API**. That System Design chapter explained WHY and WHICH STRATEGY to pick (URL path versioning, most commonly); this chapter is HOW to actually build it with `[ApiVersion]` attributes and versioned routes.

## 📌 What is it?

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok(new { id, name = "Widget", price = 49.99m }); // OLD shape
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id) =>
        Ok(new { id, name = "Widget", pricing = new { amount = 49.99m, currency = "USD" } }); // NEW shape
}
```

Both controllers live side by side, each serving its own version, so existing clients on v1 never break when v2 ships.

## 🤔 Why do we need a library for this, rather than just rolling our own?

| DIY approach                                              | `Asp.Versioning` package                                                                             |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Manually parse a version from the URL/header yourself     | Built-in readers for URL segment, query string, or header-based versioning                             |
| No standard way to report available versions to clients   | `ReportApiVersions = true` adds an `api-supported-versions` response header automatically          |
| Hard to mark a version "deprecated" in a discoverable way | `[ApiVersion("1.0", Deprecated = true)]` flags it in generated docs/headers                          |
| Manually route to the right controller based on version   | The library integrates directly with ASP.NET Core's routing to select the correct versioned controller |

## ⚙️ Internal working — setup and registration

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true; // requests with NO version → assume v1
    options.ReportApiVersions = true;                    // tells clients which versions exist, via headers
    options.ApiVersionReader = new UrlSegmentApiVersionReader(); // read the version from the URL path
}).AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV"; // formats version groups for API documentation tools
});
```

```
Request: GET /api/v1/products/5    → routed to ProductsV1Controller
Request: GET /api/v2/products/5    → routed to ProductsV2Controller
Request: GET /api/products/5       → (if version unspecified in URL) assumes DEFAULT version (v1)
```

## 📊 Version Reader Options

| Reader                          | How the client specifies a version                   | Setup                                                       |
| ------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------- |
| `UrlSegmentApiVersionReader`  | In the URL path:`/api/v2/products`                 | Most common — matches`12_System_Design`'s recommendation |
| `QueryStringApiVersionReader` | Query string:`/api/products?api-version=2.0`       | `new QueryStringApiVersionReader("api-version")`          |
| `HeaderApiVersionReader`      | Custom header:`X-Api-Version: 2.0`                 | `new HeaderApiVersionReader("X-Api-Version")`             |
| Combining readers               | Accept version from EITHER a URL segment OR a header | `ApiVersionReader.Combine(reader1, reader2)`              |

## 🖼 Deprecating an old version gracefully

```csharp
[ApiVersion("1.0", Deprecated = true)] // still fully functional, but flagged
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase { ... }
```

```
Response header automatically includes:
   api-deprecated-versions: 1.0
   api-supported-versions: 1.0, 2.0

Clients/tooling that inspect these headers can proactively warn developers
their integration is using a version scheduled for eventual removal.
```

## 💻 Code examples

### Basic — a shared BAL/DAL underneath two versioned controllers (avoiding logic duplication)

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    private readonly ProductService _service; // SAME service used by both versions
    public ProductsV1Controller(ProductService service) => _service = service;

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id); // shared BAL/DAL call
        return Ok(new { product.Id, product.Name, product.Price }); // OLD response shape
    }
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    private readonly ProductService _service; // SAME service — no duplicated business logic
    public ProductsV2Controller(ProductService service) => _service = service;

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id); // shared BAL/DAL call
        return Ok(new { product.Id, product.Name, Pricing = new { Amount = product.Price, Currency = "USD" } }); // NEW shape
    }
}
```

### Intermediate — supporting MULTIPLE versions in ONE controller (an alternative to separate controller classes)

```csharp
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    [MapToApiVersion("1.0")] // this action handles ONLY v1 requests
    public IActionResult GetByIdV1(int id) => Ok(new { id, name = "Widget", price = 49.99m });

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")] // this action handles ONLY v2 requests
    public IActionResult GetByIdV2(int id) =>
        Ok(new { id, name = "Widget", pricing = new { amount = 49.99m, currency = "USD" } });
}
```

### Practical — combining URL and header-based version reading

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-Api-Version"));
    // A client can specify the version via EITHER the URL OR the header — whichever is present wins
});
```

## ⚡ Performance considerations

- Version-based routing adds negligible overhead — it's resolved as part of the normal routing process (`04_Routing_in_Web_API.md`), not a separate, costly lookup.
- Sharing the SAME underlying BAL/DAL service across multiple versioned controllers (as shown above) avoids duplicating business logic — keeps maintenance cost down even as the number of live API versions grows.

## 🚨 Common mistakes

- ❌ Duplicating business logic across versioned controllers instead of sharing the same BAL/DAL — multiplies maintenance burden every time a bug fix or business rule change is needed.
- ❌ Never deprecating/retiring old versions — results in an ever-growing set of versions to maintain indefinitely, each needing its own testing and support.
- ❌ Forgetting `AssumeDefaultVersionWhenUnspecified` — a request with no version specified fails outright instead of falling back sensibly to a default.
- ❌ Introducing a new version for a purely additive, backward-compatible change — unnecessary version churn (recap from `12_System_Design/I.API_Design/03_API_Versioning.md`).

## 💡 Best practices

- ✅ Share the same BAL/DAL logic across all versioned controllers; let only the Controller/response-shaping layer differ between versions.
- ✅ Use `UrlSegmentApiVersionReader` as the default choice — matches the most common, most discoverable real-world convention.
- ✅ Mark deprecated versions explicitly (`Deprecated = true`) and communicate a retirement timeline, rather than supporting old versions indefinitely.
- ✅ Use `[MapToApiVersion]` within a single controller class for closely-related versions that share most routing/structure, reserving fully separate controller classes for versions with substantially different behavior.

## 🎤 Interview Quick-Fire Q&A

| Question                                                            | Answer                                                                                                                            |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| What package provides API versioning support in ASP.NET Core?       | `Asp.Versioning` (formerly part of Microsoft.AspNetCore.Mvc.Versioning)                                                         |
| How do you specify which API version a controller/action serves?    | The`[ApiVersion("x.y")]` attribute on the controller, optionally combined with `[MapToApiVersion("x.y")]` on specific actions |
| What does`AssumeDefaultVersionWhenUnspecified` do?                | Lets requests with no explicit version fall back to a configured default version, instead of failing                              |
| How do you mark an API version as deprecated without removing it?   | `[ApiVersion("1.0", Deprecated = true)]` — still functional, but flagged in response headers and generated docs                |
| Why should versioned controllers share the same underlying BAL/DAL? | To avoid duplicating business logic across versions — only the response shape/Controller layer should typically differ           |

## 📝 30-second Revision Cheat Sheet

- `Asp.Versioning` package implements URL/query/header-based versioning in ASP.NET Core.
- `[ApiVersion("x.y")]` on a controller; `[MapToApiVersion("x.y")]` to route specific actions within a shared controller.
- `UrlSegmentApiVersionReader` is the most common choice, matching `/api/v1/...`, `/api/v2/...`.
- Share the same BAL/DAL across versions — only the Controller/response shape should differ.
- Mark old versions `Deprecated = true` and communicate a retirement plan, rather than maintaining them forever.
