# 03_API_Versioning

> **API Versioning** = a strategy for evolving an API over time (adding/changing fields, behavior) **without breaking existing clients** who are still using an older version.

> Directly addresses a gap left open by `01_REST_API_Principles.md`'s "Client-Server" constraint — client and server evolve independently, but that only works safely if the server has a plan for changing its contract without breaking clients it doesn't control (a mobile app already installed on users' phones, a third-party integration, etc.).

## 📌 What is it?

Once an API is public (or used by other teams/services — as in `H.../02_Microservices.md`), you can't just change its behavior overnight. Old clients are still calling it exactly as it worked before. Versioning lets you introduce **breaking changes** in a new version while the old version keeps working for whoever still needs it.

```
Client (old mobile app, v1)  ──► /api/v1/products  → old response shape
Client (new web app, v2)     ──► /api/v2/products  → new response shape (e.g., added a field, renamed one)

BOTH work simultaneously — nobody's app breaks just because you shipped a new version.
```

## 🤔 Why do we need it?

| Problem without versioning                                                                       | How versioning helps                                                                                   |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| Changing a field name breaks every client still expecting the old name                           | Old clients keep hitting the old version; new clients opt into the new one                             |
| You can't ever safely remove/rename a field once it's public                                     | You CAN remove/rename it — in the next version, while the old version stays frozen                    |
| Different consumers (mobile app, web app, third-party) upgrade at different speeds               | Each can move to a new version on its own timeline                                                     |
| A microservice's contract change breaks OTHER services calling it (`H.../02_Microservices.md`) | Internal services can also version their APIs to avoid coordinated "everyone deploys at once" releases |

## 🌍 Real-world analogy

**Software installers with version numbers** (like `MyApp_v1.exe` and `MyApp_v2.exe` sitting side by side). Someone still running the old installer isn't affected when a new one is released — both can coexist until everyone has migrated off the old one, at which point you can finally retire it.

## 📊 Versioning Strategies — Comparison

| Strategy                                                  | How it looks                                              | Pros                                                                   | Cons                                                                                 |
| --------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **URL Path Versioning**                             | `/api/v1/products`, `/api/v2/products`                | Simple, visible, easy to test/browse directly, most common in practice | URL isn't purely about the resource anymore (arguably less "pure" REST)              |
| **Query Parameter Versioning**                      | `/api/products?version=2`                               | Keeps the URL path clean                                               | Easy to forget/omit; less visible than path versioning                               |
| **Header Versioning**                               | `Accept: application/vnd.myapp.v2+json` (custom header) | Keeps URLs completely clean, "purest" REST approach                    | Less discoverable — you can't just see the version by looking at a URL in a browser |
| **No versioning (not recommended for public APIs)** | Just change the API in place                              | Simplest — nothing to maintain                                        | Breaks every existing client immediately on any breaking change                      |

> **Most real-world APIs (Stripe, GitHub, Twitter/X) use URL Path Versioning** — it's the easiest to understand, document, test, and debug, even if header versioning is sometimes argued to be more "RESTful in spirit."

## ⚙️ Internal working — running two versions side by side

```
Client Request:  GET /api/v1/products/5             GET /api/v2/products/5
                        │                                    │
                        ▼                                    ▼
              ProductsV1Controller                  ProductsV2Controller
                        │                                    │
                        ▼                                    ▼
              (returns OLD response shape)         (returns NEW response shape,
               { id, name, price }                   e.g. { id, name, pricing: {...} })
                        │                                    │
                        └──────────┬─────────────────────────┘
                                   ▼
                        Both likely call the SAME BAL/DAL underneath —
                        only the Controller/response-shaping layer differs
```

## 🖼 What counts as a Breaking vs Non-Breaking change

| Change type                                           | Breaking? | Needs a new version?                                             |
| ----------------------------------------------------- | --------- | ---------------------------------------------------------------- |
| Adding a new, optional field to a response            | ❌ No     | No — existing clients simply ignore fields they don't recognize |
| Adding a new endpoint                                 | ❌ No     | No                                                               |
| Removing a field                                      | ✅ Yes    | Yes                                                              |
| Renaming a field                                      | ✅ Yes    | Yes                                                              |
| Changing a field's data type (e.g., string → number) | ✅ Yes    | Yes                                                              |
| Changing required request parameters                  | ✅ Yes    | Yes                                                              |
| Changing the meaning/behavior of an existing endpoint | ✅ Yes    | Yes                                                              |

> Rule of thumb: **if an existing, well-behaved client's code would break or misbehave**, it's a breaking change — version it. Purely additive changes usually don't need a new version at all.

## 💻 Code examples

### Basic — URL path versioning in ASP.NET Core (using the Asp.Versioning package)

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // tells clients which versions exist via response headers
});
```

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id);
        // OLD shape — kept exactly as-is for existing clients
        return Ok(new { product.Id, product.Name, product.Price });
    }
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id); // SAME underlying BAL/DAL call
        // NEW shape — e.g., pricing info restructured into a nested object
        return Ok(new
        {
            product.Id,
            product.Name,
            Pricing = new { product.Price, Currency = "USD" }
        });
    }
}
```

### Intermediate — header-based versioning (alternative approach)

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("X-Api-Version");
});
```

```
Client request:
   GET /api/products/5
   X-Api-Version: 2.0        ← version specified via header, URL stays clean
```

### Practical — deprecating an old version gracefully

```csharp
[ApiVersion("1.0", Deprecated = true)] // marks v1 as deprecated in generated docs/headers
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    // Still fully functional — just flagged so clients know to migrate
}
```

## ⚡ Performance / design considerations

- Maintaining multiple live versions means maintaining multiple controllers/response-shaping code — but ideally the **same underlying BAL/DAL** logic, so the maintenance burden stays mostly at the presentation layer, not duplicated business logic.
- Don't keep old versions alive forever — establish a **deprecation policy** (e.g., "v1 will be removed 12 months after v2 ships") and communicate it clearly via headers/docs.

## 🚨 Common mistakes

- ❌ Making a breaking change to an existing, live version instead of introducing a new version — this is the exact failure versioning exists to prevent.
- ❌ Never deprecating/retiring old versions — leads to an ever-growing pile of versions to maintain indefinitely.
- ❌ Duplicating business logic across version-specific controllers instead of sharing the same BAL/DAL and only varying the response shape at the controller/DTO level.
- ❌ Treating purely additive changes (new optional field) as breaking and unnecessarily bumping the version — creates version churn for no reason.

## 💡 Best practices

- ✅ Default to URL path versioning (`/api/v1/...`) unless you have a specific reason to prefer header-based versioning — it's the most widely understood and easiest to test/debug.
- ✅ Keep versioning at the Controller/DTO layer; share the same BAL/DAL logic across versions wherever the underlying behavior hasn't actually changed.
- ✅ Only introduce a new version for genuinely breaking changes — purely additive changes usually don't need one.
- ✅ Set and communicate a clear deprecation timeline for old versions, rather than supporting them forever by default.

## 🎤 Interview Quick-Fire Q&A

| Question                                                        | Answer                                                                                                                     |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Why is API versioning necessary?                                | To let an API evolve (including breaking changes) without breaking existing clients still using an older contract          |
| Name the three common versioning strategies.                    | URL Path versioning, Query Parameter versioning, Header versioning                                                         |
| Which strategy is most commonly used in real-world public APIs? | URL Path versioning (e.g.,`/api/v1/...`)                                                                                 |
| What kind of change does NOT require a new API version?         | A purely additive, backward-compatible change — like adding a new optional field to a response                            |
| What's a common mistake when maintaining multiple API versions? | Duplicating business logic per version instead of sharing the same underlying BAL/DAL and only changing the response shape |

## 📝 30-second Revision Cheat Sheet

- API Versioning = let the API evolve with breaking changes without breaking existing clients.
- Three strategies: URL Path (most common), Query Parameter, Header-based.
- Only breaking changes need a new version — purely additive changes don't.
- Share the same underlying BAL/DAL across versions; only vary the Controller/response shape.
- Always set a deprecation policy for old versions — don't support them forever by default.
