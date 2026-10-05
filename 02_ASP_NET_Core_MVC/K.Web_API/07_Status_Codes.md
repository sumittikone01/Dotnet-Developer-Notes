# 07_Status_Codes

> The practical, ASP.NET Core-specific reference for returning the RIGHT status code from a Web API action — mapping the conceptual rules from `12_System_Design/I.API_Design/02_HTTP_Methods_and_Status_Codes.md` onto the actual C# helper methods and `ProblemDetails` conventions this framework uses.

> Continues **K.Web_API**. `01_API_Controllers.md` listed the common result helpers briefly; this chapter is the deep, deliberate reference for choosing the correct one every time.

## 📌 What is it?

```csharp
return Ok(data);                              // 200
return CreatedAtAction(nameof(GetById), new { id }, data); // 201
return NoContent();                           // 204
return BadRequest(ModelState);                // 400
return Unauthorized();                        // 401
return Forbid();                              // 403
return NotFound();                            // 404
return Conflict("Resource already exists.");  // 409
return StatusCode(422, new { error = "..." }); // 422 — no dedicated helper exists for this one
return StatusCode(500, "Unexpected error.");  // 500
```

## 🤔 Why do we need a dedicated deep-dive on this, beyond the basics?

| Gap from earlier coverage                                    | What this chapter adds                                                            |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| `01_API_Controllers.md` listed helpers briefly             | Full reference: EVERY common helper, when to use EACH one, and what it returns    |
| Easy to default to`Ok()`/`BadRequest()` for everything   | Precise guidance on 401 vs 403, 404 vs 409, and when NO dedicated helper exists   |
| Teams often build inconsistent, ad-hoc error response shapes | `ProblemDetails` — the STANDARD .NET convention for structured error responses |

## 📊 Full Result Helper Reference

| Helper                                         | Status Code | Use when                                                                                                                     |
| ---------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `Ok(data)`                                   | 200         | Successful GET, or a successful PUT/PATCH that returns data                                                                  |
| `CreatedAtAction(action, routeValues, data)` | 201         | Successful POST — includes a`Location` header pointing to the new resource                                                |
| `Accepted()`                                 | 202         | The request was accepted for ASYNC/background processing, not yet complete                                                   |
| `NoContent()`                                | 204         | Successful DELETE, or a PUT/PATCH with nothing to return                                                                     |
| `BadRequest(errors)`                         | 400         | Invalid input — validation failures, malformed request                                                                      |
| `Unauthorized()`                             | 401         | Caller is NOT authenticated at all                                                                                           |
| `Forbid()`                                   | 403         | Caller IS authenticated, but lacks permission for this action                                                                |
| `NotFound()`                                 | 404         | The requested resource doesn't exist                                                                                         |
| `Conflict(data)`                             | 409         | Request conflicts with current state (duplicate, version mismatch)                                                           |
| `UnprocessableEntity(errors)`                | 422         | Well-formed request, but semantically invalid (business rule violation) — available directly in newer ASP.NET Core versions |
| `StatusCode(code, data)`                     | Anything    | The escape hatch for any status code without a dedicated helper                                                              |

## ⚙️ Internal working — `ProblemDetails`, the standardized error shape

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _service.GetProductById(id);
    if (product == null)
    {
        return NotFound(new ProblemDetails
        {
            Title = "Product not found",
            Detail = $"No product exists with Id {id}.",
            Status = StatusCodes.Status404NotFound
        });
    }
    return Ok(product);
}
```

```json
// The RFC 7807-standardized shape this produces:
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Product not found",
  "status": 404,
  "detail": "No product exists with Id 5."
}
```

> Recall from `02_ApiController_Attribute.md`: `[ApiController]` ALREADY produces `ValidationProblemDetails` automatically for model validation failures — using `ProblemDetails` deliberately for YOUR OWN error responses keeps the whole API's error shape CONSISTENT with that automatic behavior.

## 🖼 401 vs 403 vs 404 vs 409 — the precise decision tree

```
Is the caller even authenticated (do we know who they are)?
   NO  → 401 Unauthorized

Is the caller authenticated, but not allowed to do THIS specific action?
   YES → 403 Forbidden

Does the requested resource simply not exist?
   YES → 404 Not Found

Does the request conflict with the CURRENT STATE of the resource
(e.g., trying to create a duplicate, or a version mismatch)?
   YES → 409 Conflict

Otherwise, if the data itself was semantically invalid (passed validation,
but violates a business rule)?
   → 422 Unprocessable Entity (or 400, if your team prefers to keep it simpler)
```

## 💻 Code examples

### Basic — choosing the right code across a full CRUD controller

```csharp
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id);
        return product == null ? NotFound() : Ok(product); // 404 or 200
    }

    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        var created = _service.CreateProduct(model);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created); // 201
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, ProductViewModel model)
    {
        if (!_service.Exists(id)) return NotFound(); // 404
        _service.UpdateProduct(id, model);
        return NoContent(); // 204
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        if (!_service.Exists(id)) return NotFound(); // 404
        _service.DeleteProduct(id);
        return NoContent(); // 204
    }
}
```

### Intermediate — 401 vs 403 vs 409 in a realistic scenario

```csharp
[HttpDelete("{id}")]
[Authorize] // the framework handles 401 automatically if there's no valid token at all
public IActionResult Delete(int id)
{
    if (!User.IsInRole("Admin"))
        return Forbid(); // 403 — authenticated, but not allowed

    var product = _service.GetProductById(id);
    if (product == null) return NotFound(); // 404

    if (product.IsLockedForAudit)
        return Conflict(new ProblemDetails
        {
            Title = "Cannot delete",
            Detail = "This product is currently locked for audit.",
            Status = StatusCodes.Status409Conflict
        }); // 409

    _service.DeleteProduct(id);
    return NoContent(); // 204
}
```

### Practical — using `StatusCode()` for the 422 case (business rule violation)

```csharp
[HttpPost("{id}/apply-discount")]
public IActionResult ApplyDiscount(int id, decimal discountPercent)
{
    var product = _service.GetProductById(id);
    if (product == null) return NotFound();

    if (discountPercent > product.MaxAllowedDiscount)
    {
        // Well-formed request, valid data types — but violates a BUSINESS rule, not basic validation
        return StatusCode(422, new ProblemDetails
        {
            Title = "Discount exceeds allowed maximum",
            Detail = $"Maximum allowed discount for this product is {product.MaxAllowedDiscount}%.",
            Status = 422
        });
    }

    _service.ApplyDiscount(id, discountPercent);
    return NoContent();
}
```

## ⚡ Performance considerations

- Choosing the correct status code has no direct performance cost — but it HEAVILY affects client behavior: a correctly-returned 429 (Too Many Requests, `J.Reliability_Patterns`-adjacent) or 503 tells a well-behaved client to retry; a generic 500 for everything gives the client no useful signal at all.
- `ProblemDetails` responses are lightweight — no meaningful overhead versus a plain anonymous object, while providing a standardized, tooling-friendly shape.

## 🚨 Common mistakes

- ❌ Returning 200 OK for everything, including failures, with success/failure indicated only in a custom JSON field — breaks client tooling, caching assumptions, and monitoring/alerting that relies on real HTTP status codes.
- ❌ Confusing 401 and 403 (see `12_System_Design/I.API_Design/02_HTTP_Methods_and_Status_Codes.md` for the full breakdown) — a very common, recurring mistake worth re-flagging here specifically for Web API contexts.
- ❌ Using 400 for EVERY kind of input problem, never distinguishing basic validation failures (400) from deeper business-rule violations (409/422) — makes client-side error handling harder to get right.
- ❌ Building ad-hoc, inconsistent error response shapes across different actions instead of standardizing on `ProblemDetails`.

## 💡 Best practices

- ✅ Use the dedicated result helper for every common case; reserve `StatusCode(code, data)` for the less common codes (422, 429, 503) that lack a dedicated helper.
- ✅ Standardize error responses on `ProblemDetails`/`ValidationProblemDetails` across the whole API — consistent with what `[ApiController]` already does automatically for validation errors.
- ✅ Distinguish 400 (malformed/invalid input), 409 (conflicts with current state), and 422 (semantically invalid per business rules) deliberately, rather than defaulting everything to 400.
- ✅ Always return `CreatedAtAction` (not just `Ok`) for successful resource creation, so clients get the `Location` header pointing to the new resource.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                            | Answer                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What status code should a successful DELETE typically return?                                       | 204 No Content                                                                                                                                            |
| What's the difference between 409 Conflict and 422 Unprocessable Entity?                            | 409 means the request conflicts with the CURRENT STATE of a resource (e.g., duplicate); 422 means the request is well-formed but violates a business rule |
| What does`ProblemDetails` provide?                                                                | A standardized (RFC 7807), consistent JSON shape for error responses, with fields like`title`, `status`, and `detail`                               |
| What helper should you use when no dedicated result method exists for a status code?                | `StatusCode(code, data)` — the general-purpose escape hatch                                                                                            |
| Why is returning 200 OK for every response, with errors indicated only in the body, a bad practice? | It breaks standard HTTP tooling assumptions (caching, retries, monitoring) that rely on the actual status code to understand the outcome                  |

## 📝 30-second Revision Cheat Sheet

- Use the dedicated helper for each case: `Ok`, `CreatedAtAction`, `NoContent`, `BadRequest`, `Unauthorized`, `Forbid`, `NotFound`, `Conflict`; fall back to `StatusCode(code, data)` for anything else (422, 429, 503).
- 401 = not authenticated; 403 = authenticated but not allowed; 404 = doesn't exist; 409 = conflicts with current state; 422 = valid format, violates a business rule.
- Standardize error responses on `ProblemDetails`/`ValidationProblemDetails` for consistency with `[ApiController]`'s automatic validation errors.
- Always use `CreatedAtAction` for successful POST responses, to include the `Location` header.
- Never return 200 for failures with errors hidden in the body — it breaks real HTTP tooling and monitoring.
