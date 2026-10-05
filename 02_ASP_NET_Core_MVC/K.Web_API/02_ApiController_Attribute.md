# 02_ApiController_Attribute

> **`[ApiController]`** = an attribute that turns on several automatic, API-friendly behaviors for a Controller — most notably, automatic model validation, so you can delete the `if (!ModelState.IsValid) return BadRequest(...)` boilerplate you've been writing in every action.

> Continues **K.Web_API**, directly following `01_API_Controllers.md`. This is the single attribute that makes an "API Controller" behave meaningfully differently from a plain `ControllerBase` Controller.

## 📌 What is it?

```csharp
[ApiController]              // ← this one attribute unlocks everything below
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        // NOTICE: no "if (!ModelState.IsValid) return BadRequest(...)" here!
        // [ApiController] does this AUTOMATICALLY, before this method even runs.
        var created = _service.CreateProduct(model);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

## 🤔 Why do we need it?

| Without`[ApiController]`                                                          | With`[ApiController]`                                                                       |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Must manually check`ModelState.IsValid` in EVERY action                           | Automatic — invalid models short-circuit BEFORE the action runs, returning 400 automatically |
| Action parameters need explicit`[FromBody]`/`[FromQuery]` attributes more often | Smarter automatic inference of where a parameter comes from                                   |
| Routing errors (e.g., missing route attribute) fail less predictably                | Requires attribute routing — enforces a clearer, more consistent API structure               |
| Raw validation error responses can be inconsistent                                  | Standardized`ValidationProblemDetails` response format for validation errors                |

## 📊 The Four Automatic Behaviors `[ApiController]` Enables

| Behavior                                      | What it does                                                                                                                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Automatic Model Validation**          | If`ModelState.IsValid` is false, the framework AUTOMATICALLY returns a 400 response BEFORE your action method even runs — you never need to check it manually               |
| **Automatic Binding Source Inference**  | Smarter default rules for`[FromBody]`/`[FromRoute]`/`[FromQuery]` — often you don't need to specify them explicitly at all (see `05_FromBody_FromQuery_FromRoute.md`) |
| **Attribute Routing Requirement**       | Enforces that Controllers use ATTRIBUTE routing (`[Route]`, `[HttpGet]`, etc.) — conventional routing isn't supported for `[ApiController]`-decorated controllers       |
| **Problem Details for error responses** | Validation errors and certain error responses automatically use the standardized`application/problem+json` format (RFC 7807)                                                 |

## ⚙️ Internal working — automatic model validation, in detail

```
Request arrives with an invalid body (e.g., missing a [Required] field)
        │
        ▼
Model binding happens — populates the action's parameter object
        │
        ▼
[ApiController] CHECKS ModelState.IsValid AUTOMATICALLY, as a built-in filter-like behavior
        │
   ┌────┴────┐
   ▼         ▼
 VALID     INVALID
   │         │
   │         ▼
   │   Framework returns 400 Bad Request AUTOMATICALLY,
   │   with a ValidationProblemDetails body —
   │   YOUR ACTION METHOD CODE NEVER EVEN RUNS
   ▼
Action method runs normally
```

```json
// Automatic error response shape (ValidationProblemDetails) when ModelState is invalid:
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The field Price must be between 0.01 and 100000."]
  }
}
```

## 📊 Binding Source Inference — smarter defaults

| Parameter situation                                                 | Inferred binding source (WITH`[ApiController]`)            |
| ------------------------------------------------------------------- | ------------------------------------------------------------ |
| Complex type (a class like`ProductViewModel`)                     | `[FromBody]` — assumed to come from the JSON request body |
| Simple type (`int`, `string`) matching a route template segment | `[FromRoute]`                                              |
| Simple type NOT matching a route segment                            | `[FromQuery]`                                              |
| A parameter typed as a registered service                           | `[FromServices]`                                           |

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id) => ...  // 'id' automatically inferred as [FromRoute] — matches "{id}" in the route

[HttpGet]
public IActionResult Search(string category) => ...  // automatically inferred as [FromQuery] — no matching route segment

[HttpPost]
public IActionResult Create(ProductViewModel model) => ...  // automatically inferred as [FromBody] — it's a complex type
```

> Full depth on this in `05_FromBody_FromQuery_FromRoute.md` — the point here is that `[ApiController]` is WHY these inferences happen automatically at all; without it, you'd need to specify the binding source attribute explicitly far more often.

## 💻 Code examples

### Basic — comparing WITH and WITHOUT `[ApiController]`

```csharp
// WITHOUT [ApiController] — manual validation check required EVERY time
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        if (!ModelState.IsValid) // must be written explicitly, every single action
        {
            return BadRequest(ModelState);
        }
        return Ok(_service.CreateProduct(model));
    }
}

// WITH [ApiController] — this check happens automatically; the code above is DEAD CODE if kept
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        // ModelState is ALREADY guaranteed valid by the time this line runs
        return Ok(_service.CreateProduct(model));
    }
}
```

### Intermediate — customizing the automatic validation response (if the default format doesn't fit)

```csharp
// Program.cs
builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.InvalidModelStateResponseFactory = context =>
    {
        var errors = context.ModelState
            .Where(e => e.Value?.Errors.Count > 0)
            .ToDictionary(e => e.Key, e => e.Value!.Errors.Select(x => x.ErrorMessage).ToArray());

        return new BadRequestObjectResult(new { success = false, errors }); // custom shape instead of the default
    };
});
```

### Practical — a full API controller leaning on all four automatic behaviors

```csharp
[ApiController]                          // enables everything below
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    private readonly ProductService _service;
    public ProductsApiController(ProductService service) => _service = service; // [FromServices]-style DI, as always

    [HttpGet("{id}")]
    public IActionResult GetById(int id) =>                     // 'id' auto-inferred as [FromRoute]
        _service.GetProductById(id) is Product p ? Ok(p) : NotFound();

    [HttpGet]
    public IActionResult Search(string? category) =>            // auto-inferred as [FromQuery]
        Ok(_service.SearchProducts(category));

    [HttpPost]
    public IActionResult Create(ProductViewModel model)          // auto-inferred as [FromBody]; auto-validated
    {
        var created = _service.CreateProduct(model);             // ModelState is guaranteed valid here
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

## ⚡ Performance considerations

- Automatic model validation runs the SAME `ModelState.IsValid` check you'd write manually — no additional performance cost, just moved earlier and made automatic.
- No meaningful runtime overhead from `[ApiController]` itself — its behaviors are convenience/consistency features, not expensive additions to the pipeline.

## 🚨 Common mistakes

- ❌ Leaving old, manual `if (!ModelState.IsValid) return BadRequest(...)` checks in place after adding `[ApiController]` — harmless (just unreachable dead code, since it's already been checked), but adds clutter and can confuse readers into thinking it's still necessary.
- ❌ Forgetting that `[ApiController]` REQUIRES attribute routing (`[Route]`/`[HttpGet]` etc.) — mixing it with conventional routing (`app.MapControllerRoute(...)` targeting that controller) doesn't work as expected.
- ❌ Being surprised by the automatic 400 response format when a team expects a different error shape — customize via `ApiBehaviorOptions.InvalidModelStateResponseFactory` if the default doesn't fit your API's conventions.
- ❌ Assuming `[ApiController]` alone provides authentication/authorization — it's purely about model validation, binding inference, and response conventions, NOT security (see `L.Security` for that).

## 💡 Best practices

- ✅ Apply `[ApiController]` to every Controller intended purely as a data API — the automatic validation alone eliminates a large amount of repetitive boilerplate.
- ✅ Remove manual `ModelState.IsValid` checks once `[ApiController]` is applied — they're redundant.
- ✅ Customize the validation error response shape via `ApiBehaviorOptions` if your team has a specific, consistent error contract across the API.
- ✅ Remember `[ApiController]` requires attribute routing — always pair it with `[Route]`/`[HttpGet]`/etc., never rely on conventional routing for these controllers.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                               | Answer                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What is the most significant behavior`[ApiController]` enables?                                      | Automatic model validation — an invalid`ModelState` returns a 400 response automatically, before the action method runs                                                                                      |
| Do you still need`if (!ModelState.IsValid) return BadRequest(...)` with `[ApiController]` applied? | No — it's handled automatically; leaving the manual check in is redundant (though harmless)                                                                                                                    |
| What routing requirement does`[ApiController]` impose?                                               | It requires attribute routing (`[Route]`, `[HttpGet]`, etc.) — conventional routing isn't supported for these controllers                                                                                  |
| What is "binding source inference"?                                                                    | `[ApiController]` automatically infers where a parameter should come from (body, route, or query) based on its type, reducing the need for explicit `[FromBody]`/`[FromQuery]`/`[FromRoute]` attributes |
| How can you customize the automatic validation error response format?                                  | Configure`ApiBehaviorOptions.InvalidModelStateResponseFactory` in `Program.cs`                                                                                                                              |

## 📝 30-second Revision Cheat Sheet

- `[ApiController]` enables: automatic model validation, smarter binding source inference, mandatory attribute routing, standardized Problem Details error responses.
- Automatic validation means `if (!ModelState.IsValid) return BadRequest(...)` is no longer needed — delete that boilerplate.
- Requires attribute routing (`[Route]`/`[HttpGet]`) — doesn't work with conventional routing.
- Customize the default 400 response shape via `ApiBehaviorOptions.InvalidModelStateResponseFactory` if needed.
- `[ApiController]` is about validation/binding conventions — NOT security; auth still needs `[Authorize]` (`L.Security`).
