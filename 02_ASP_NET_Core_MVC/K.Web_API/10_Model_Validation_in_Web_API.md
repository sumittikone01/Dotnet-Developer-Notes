
# 10_Model_Validation_in_Web_API

> A full synthesis chapter, pulling together `H.Model_Handling`'s Data Annotations and custom validation attributes with `02_ApiController_Attribute.md`'s automatic `ModelState` checking — the complete, end-to-end picture of how a Web API validates incoming data, from attribute to response.

> Closes out **K.Web_API** (01–10). This is the "how everything you've learned fits together" chapter for validation specifically in a Web API context (as opposed to a traditional Razor View-based form flow).

## 📌 What is it?

```csharp
public class ProductViewModel
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = "";

    [Range(0.01, 100000)]
    public decimal Price { get; set; }

    [SkuFormat] // custom attribute, from H.Model_Handling/05_Custom_Validation_Attributes.md
    public string Sku { get; set; } = "";
}

[ApiController] // enables AUTOMATIC validation — no manual ModelState.IsValid check needed!
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        // By the time we're HERE, 'model' is GUARANTEED valid — [ApiController] already checked
        var created = _service.CreateProduct(model);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

## 🤔 Why revisit validation again here, after `H.Model_Handling` and `02_ApiController_Attribute.md`?

| Earlier coverage                                                 | What this chapter adds                                                                                                                                                              |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `H.Model_Handling/05_Custom_Validation_Attributes.md`          | How to WRITE validation rules                                                                                                                                                       |
| `H.Model_Handling/06_Client_Side_vs_Server_Side_Validation.md` | Why BOTH client and server validation matter — in a RAZOR VIEW context                                                                                                             |
| `02_ApiController_Attribute.md`                                | That`[ApiController]` automates the CHECK                                                                                                                                         |
| **This chapter**                                           | The FULL picture specific to Web APIs: there's no client-side JS validation layer the way a Razor View has — the API's validation IS the only validation most API callers ever see |

## 🌍 Real-world analogy

A Razor View's validation is like a **store clerk checking your order before you even walk to the register** (client-side) and the **register itself double-checking everything** (server-side) — two chances to catch a problem. A Web API has NO clerk — it's like a **vending machine**: the ONLY check that happens is right at the point of the actual transaction (server-side). This is exactly why Web API validation needs to be especially thorough and give CLEAR, ACTIONABLE error messages — there's no earlier UI layer to have already caught obvious mistakes.

## ⚙️ Internal working — the full validation flow for a Web API request

```
1. Request arrives: POST /api/products  { "name": "", "price": -5, "sku": "bad-format" }
        │
        ▼
2. Model binding populates a ProductViewModel from the JSON body
        │
        ▼
3. Data Annotations run automatically during binding:
     [Required] on Name      → FAILS (empty string)
     [Range] on Price         → FAILS (-5 is below 0.01)
     [SkuFormat] on Sku       → FAILS (doesn't match pattern)
        │
        ▼
4. [ApiController] checks ModelState.IsValid → FALSE
        │
        ▼
5. Framework AUTOMATICALLY returns 400 Bad Request with a
   ValidationProblemDetails body — the ACTION METHOD NEVER RUNS AT ALL
```

```json
// The AUTOMATIC response (no code needed to produce this):
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The field Price must be between 0.01 and 100000."],
    "Sku": ["SKU must be in the format ABC-12345."]
  }
}
```

## 📊 Validation Layers in a Web API, End to End

| Layer                                                | What it catches                                                           | Covered in                                              |
| ---------------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------- |
| Data Annotations (`[Required]`, `[Range]`, etc.) | Basic field-level rules                                                   | `H.Model_Handling`                                    |
| Custom`ValidationAttribute`                        | Business-specific, single-property rules                                  | `H.Model_Handling/05_Custom_Validation_Attributes.md` |
| `IValidatableObject`                               | Cross-property rules (EndDate > StartDate)                                | `H.Model_Handling/05_Custom_Validation_Attributes.md` |
| `[ApiController]`'s automatic check                | Enforces ALL of the above automatically, before the action runs           | `02_ApiController_Attribute.md`                       |
| BAL-level business validation                        | Rules requiring a DB lookup or complex logic (can't live in an attribute) | Your Controller → BAL → DAL flow                      |

> **Critical distinction:** Data Annotations/custom attributes validate the SHAPE and basic rules of the data. Deeper business validation that needs a DATABASE CHECK (e.g., "is this SKU already taken?") belongs in the BAL — NOT in a `ValidationAttribute` (recall `05_Custom_Validation_Attributes.md`'s warning against DB calls inside `IsValid`).

## 💻 Code examples

### Basic — relying entirely on automatic validation (the common case)

```csharp
public class CreateOrderRequest
{
    [Required]
    public int ProductId { get; set; }

    [Range(1, 1000)]
    public int Quantity { get; set; }
}

[ApiController]
[Route("api/orders")]
public class OrdersApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateOrderRequest request)
    {
        // No manual validation check needed — [ApiController] already guaranteed it's valid
        var order = _service.CreateOrder(request.ProductId, request.Quantity);
        return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
    }
}
```

### Intermediate — layering BAL-level business validation on top of automatic attribute validation

```csharp
[HttpPost]
public IActionResult Create(CreateOrderRequest request)
{
    // Attribute-level validation ALREADY passed by this point (guaranteed by [ApiController])

    try
    {
        // BAL-level validation — requires a DB lookup, can't live in a ValidationAttribute
        var order = _service.CreateOrder(request.ProductId, request.Quantity);
        return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
    }
    catch (InsufficientStockException ex)
    {
        // A BUSINESS rule violation, distinct from basic shape/format validation
        return Conflict(new ProblemDetails { Title = "Insufficient stock", Detail = ex.Message, Status = 409 });
    }
    catch (ProductNotFoundException ex)
    {
        return NotFound(new ProblemDetails { Title = "Product not found", Detail = ex.Message, Status = 404 });
    }
}
```

### Practical — customizing the automatic validation error SHAPE for API consistency

```csharp
// Program.cs — tailor the automatic 400 response to match your team's preferred error contract
builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.InvalidModelStateResponseFactory = context =>
    {
        var errors = context.ModelState
            .Where(kvp => kvp.Value?.Errors.Count > 0)
            .SelectMany(kvp => kvp.Value!.Errors.Select(e => new { field = kvp.Key, message = e.ErrorMessage }));

        return new BadRequestObjectResult(new
        {
            success = false,
            errors
        });
    };
});
```

### Practical — a cross-property rule relevant to Web API request DTOs

```csharp
public class CreatePromotionRequest : IValidatableObject
{
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (EndDate <= StartDate)
        {
            yield return new ValidationResult(
                "End date must be after the start date.",
                new[] { nameof(EndDate) });
        }
    }
}
// [ApiController] automatically invokes THIS too — no extra wiring needed beyond implementing the interface
```

## ⚡ Performance considerations

- Automatic model validation runs the same lightweight checks you'd write manually — no added cost versus doing it yourself, just moved earlier and made automatic and consistent.
- BAL-level business validation (requiring DB lookups) should still be the LAST line of defense, checked only AFTER cheap attribute-level validation has already passed — avoids wasting a database round-trip validating a request that was already obviously malformed.

## 🚨 Common mistakes

- ❌ Writing manual `if (!ModelState.IsValid)` checks that are now dead code, after `[ApiController]` already handles it automatically — harmless but redundant clutter.
- ❌ Putting business rules that require a database lookup inside a `ValidationAttribute` — violates the "keep validation attributes fast and side-effect-free" principle from `H.Model_Handling/05_Custom_Validation_Attributes.md`.
- ❌ Returning inconsistent error shapes for attribute-level validation (automatic `ValidationProblemDetails`) versus BAL-level business errors (a custom shape) — confusing for API consumers; standardize both on the same `ProblemDetails`-based format.
- ❌ Assuming Web API clients will always send well-formed requests "because our own frontend does" — any API endpoint could be called directly (Postman, another service, a malicious actor), so validation must never be skipped.

## 💡 Best practices

- ✅ Rely on `[ApiController]`'s automatic validation for all attribute-level/shape validation — don't write manual checks for what it already handles.
- ✅ Reserve BAL-level validation specifically for rules that genuinely need a database lookup or complex cross-entity logic.
- ✅ Standardize BOTH automatic attribute-validation errors AND manually-thrown business-rule errors on the same `ProblemDetails` response shape, for a consistent API contract.
- ✅ Treat the API's own validation as the ONLY safety net — there's no client-side JS layer to rely on the way a Razor View has, so make error messages clear and complete.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                              | Answer                                                                                                                                                               |
| ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why is server-side validation especially critical for a Web API, compared to a Razor View-based form? | A Web API has no client-side JS validation layer in front of it by default — it's often the ONLY validation any caller (including direct API clients) will ever hit |
| What triggers the automatic 400 response for an API Controller?                                       | `[ApiController]` checking `ModelState.IsValid` automatically, based on Data Annotations and custom validation attributes applied to the model                   |
| Where should business rules requiring a database lookup be validated?                                 | In the BAL (business logic layer) — NOT inside a`ValidationAttribute`, which should stay fast and side-effect-free                                                |
| How can you customize the shape of the automatic validation error response?                           | Configure`ApiBehaviorOptions.InvalidModelStateResponseFactory` in `Program.cs`                                                                                   |
| Does`[ApiController]` automatically invoke `IValidatableObject.Validate()` too?                   | Yes — cross-property validation via`IValidatableObject` is included in the same automatic `ModelState` check                                                    |

## 📝 30-second Revision Cheat Sheet

- A Web API has NO client-side validation layer by default — server-side validation is the ONLY safety net for most callers.
- `[ApiController]` automatically runs Data Annotations, custom `ValidationAttribute`s, AND `IValidatableObject` checks — all before the action method executes.
- Reserve BAL-level validation for rules needing a database lookup; keep attribute-based validation fast and side-effect-free.
- Standardize both automatic validation errors and manually-thrown business errors on the same `ProblemDetails` shape.
- Never write a manual `ModelState.IsValid` check in an `[ApiController]`-decorated controller — it's redundant.

---

✅ **K.Web_API chapter complete** (01–10). Next up: **L.Security**.
