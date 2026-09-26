# 02_HTTP_Methods_and_Status_Codes

> **HTTP Methods** tell the server **what action** to perform on a resource; **HTTP Status Codes** tell the client **what happened** as a result. Together, they're the "verb + outcome" vocabulary that makes REST (`01_REST_API_Principles.md`) actually work.

## 📌 What is it?

`01_REST_API_Principles.md` said "let the HTTP verb express the action, don't bake it into the URL." This chapter defines exactly **which verb means what**, and exactly **which status code to return** for every outcome — the two halves of a REST request/response.

## 🤔 Why do we need it?

Without a shared, standard vocabulary, every API would invent its own conventions — "does POST mean create or update here? Does this API return 200 or 201 for a successful creation? Does an error come back as a 400 or a 200 with an error flag in the body?" HTTP Methods + Status Codes remove that ambiguity **industry-wide**.

## 📊 HTTP Methods — Full Reference

| Method           | Purpose                     | Idempotent? | Safe? (no side effects) | Typical use                                          |
| ---------------- | --------------------------- | ----------- | ----------------------- | ---------------------------------------------------- |
| **GET**    | Retrieve a resource         | ✅ Yes      | ✅ Yes                  | `GET /products/5` — read data                     |
| **POST**   | Create a new resource       | ❌ No       | ❌ No                   | `POST /products` — create a new product           |
| **PUT**    | Replace a resource entirely | ✅ Yes      | ❌ No                   | `PUT /products/5` — overwrite the whole product   |
| **PATCH**  | Partially update a resource | ❌ No*      | ❌ No                   | `PATCH /products/5` — update just the price field |
| **DELETE** | Remove a resource           | ✅ Yes      | ❌ No                   | `DELETE /products/5` — delete the product         |

> *PATCH's idempotency depends on the specific operation (e.g., "set price to 50" is idempotent; "increment price by 5" is NOT).

## 🧠 Idempotent vs Safe — the distinction that trips people up

| Term                 | Meaning                                                            | Example                                                                                                                                                                                                             |
| -------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Safe**       | Doesn't change server state at all                                 | `GET` — reading data never modifies anything                                                                                                                                                                     |
| **Idempotent** | Calling it once or 100 times produces the**same end result** | `DELETE /products/5` called 5 times still ends with the product gone (not an error each time) — same end state; `PUT /products/5` with the same body 5 times leaves the resource in the exact same final state |

```
POST /products (creates a new product) called 3 times:
   → Creates THREE separate products (NOT idempotent — each call has a new effect)

PUT /products/5 { name: "Widget", price: 50 } called 3 times:
   → Product #5 ends up as { name: "Widget", price: 50 } — same result every time (IDEMPOTENT)

DELETE /products/5 called 3 times:
   → First call deletes it. 2nd and 3rd calls: product's already gone — end STATE is the
     same (gone) even if the 2nd/3rd calls technically return 404 instead of 204
```

## 📊 PUT vs PATCH — the other commonly confused pair

| Aspect                        | PUT                                                                  | PATCH                                                |
| ----------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------- |
| Scope of update               | Replaces the ENTIRE resource                                         | Updates only the SPECIFIED fields                    |
| Body content                  | Full resource representation                                         | Partial — only the fields being changed             |
| Missing fields in the request | Typically treated as "clear this field" (since it's a full replace)  | Left untouched — only what's sent is changed        |
| Example                       | `PUT /products/5` with the full `{ name, price, category, ... }` | `PATCH /products/5` with just `{ price: 45.00 }` |

## 📊 HTTP Status Codes — Grouped by Category

```
1xx → Informational (rarely seen directly in typical REST APIs)
2xx → SUCCESS        ✅
3xx → Redirection     ↪
4xx → CLIENT error   ❌ (the caller did something wrong)
5xx → SERVER error   🔥 (something broke on the server side)
```

### 2xx — Success

| Code          | Name       | When to use                                                                                                 |
| ------------- | ---------- | ----------------------------------------------------------------------------------------------------------- |
| **200** | OK         | Standard success —`GET`, successful `PUT`/`PATCH` that returns data                                  |
| **201** | Created    | Successful`POST` that created a new resource — include the new resource's URL in the `Location` header |
| **204** | No Content | Success, but nothing to return — typical for`DELETE`, or `PUT`/`PATCH` that don't return a body      |

### 4xx — Client Errors (the caller needs to fix their request)

| Code          | Name                 | When to use                                                                        |
| ------------- | -------------------- | ---------------------------------------------------------------------------------- |
| **400** | Bad Request          | Malformed request — invalid data, failed validation                               |
| **401** | Unauthorized         | Caller isn't authenticated at all (no/invalid token)                               |
| **403** | Forbidden            | Caller IS authenticated, but doesn't have permission for this action               |
| **404** | Not Found            | The requested resource doesn't exist                                               |
| **409** | Conflict             | Request conflicts with the current state (e.g., duplicate entry, version conflict) |
| **422** | Unprocessable Entity | Request is well-formed, but semantically invalid (e.g., business rule violation)   |
| **429** | Too Many Requests    | Rate limit exceeded (ties into`J.Reliability_Patterns/04_Rate_Limiting.md`)      |

### 5xx — Server Errors (something broke on the server)

| Code          | Name                  | When to use                                                                                            |
| ------------- | --------------------- | ------------------------------------------------------------------------------------------------------ |
| **500** | Internal Server Error | Generic, unhandled server-side failure (e.g., unexpected exception, DB connection failure)             |
| **502** | Bad Gateway           | A gateway/proxy got an invalid response from an upstream service (ties into`H.../03_API_Gateway.md`) |
| **503** | Service Unavailable   | Server is temporarily overloaded or down for maintenance                                               |
| **504** | Gateway Timeout       | A gateway/proxy timed out waiting for an upstream service                                              |

## 🖼 401 vs 403 — precisely, since these are always confused

```
401 Unauthorized:  "I don't know who you are." (missing/invalid token — not authenticated)
403 Forbidden:     "I know exactly who you are, but you're not allowed to do this."
                    (authenticated, but lacking permission)

Example:
  A logged-in regular user tries to access an ADMIN-only endpoint:
  → 403 Forbidden (we know who they are; they just can't do this)

  A request with NO auth token tries to access ANY protected endpoint:
  → 401 Unauthorized (we don't know who's asking at all)
```

## 💻 Code examples

### Basic — correct status codes for each CRUD operation

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id);
        return product == null
            ? NotFound()                 // 404 — resource doesn't exist
            : Ok(product);                // 200 — success, with data
    }

    [HttpPost]
    public IActionResult Create(ProductViewModel model)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState); // 400 — validation failed

        var created = _service.CreateProduct(model);
        // 201 — created, includes Location header pointing to the new resource
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id}")]
    public IActionResult Update(int id, ProductViewModel model)
    {
        if (!_service.Exists(id))
            return NotFound();            // 404

        _service.UpdateProduct(id, model);
        return NoContent();               // 204 — success, nothing to return
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        if (!_service.Exists(id))
            return NotFound();            // 404

        _service.DeleteProduct(id);
        return NoContent();               // 204
    }
}
```

### Intermediate — 401 vs 403 vs 409 in practice

```csharp
[HttpDelete("{id}")]
[Authorize]
public IActionResult Delete(int id)
{
    // Framework already returns 401 automatically if there's no valid token at all

    if (!User.IsInRole("Admin"))
        return Forbid();                          // 403 — authenticated, but not allowed

    var product = _service.GetProductById(id);
    if (product == null)
        return NotFound();                        // 404

    if (product.IsLockedForAudit)
        return Conflict("Product is locked for audit and cannot be deleted."); // 409

    _service.DeleteProduct(id);
    return NoContent();                            // 204
}
```

### Practical — global error handling mapping exceptions to correct status codes

```csharp
// Middleware — centralizes status code selection instead of scattering try/catch everywhere
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var exceptionHandlerFeature = context.Features.Get<IExceptionHandlerFeature>();
        var ex = exceptionHandlerFeature?.Error;

        context.Response.StatusCode = ex switch
        {
            ValidationException => StatusCodes.Status400BadRequest,
            KeyNotFoundException => StatusCodes.Status404NotFound,
            UnauthorizedAccessException => StatusCodes.Status403Forbidden,
            _ => StatusCodes.Status500InternalServerError // fallback for anything unexpected
        };

        await context.Response.WriteAsJsonAsync(new { error = ex?.Message });
    });
});
```

## ⚡ Performance / design considerations

- Correct use of `GET` (safe, cacheable) lets browsers/CDNs/proxies cache responses automatically — misusing `GET` for state-changing operations breaks this entirely.
- Returning proper status codes lets clients (and tools like Postman, monitoring dashboards, API gateways) make automated decisions (retry on 503, don't retry on 400) — this is exactly what powers `03_Fault_Tolerance.md`'s retry logic ("retry transient failures" only makes sense if the status code correctly signals which failures ARE transient).

## 🚨 Common mistakes

- ❌ Returning `200 OK` for every response, with success/failure indicated only inside the JSON body — defeats the entire purpose of status codes and breaks standard HTTP tooling/caching/retry behavior.
- ❌ Using `GET` for an operation that changes data — breaks the "safe" guarantee and can cause caching layers to serve stale or incorrect behavior.
- ❌ Confusing 401 and 403 — returning 401 when the user IS authenticated but just lacks permission (should be 403), or vice versa.
- ❌ Using 500 for everything, including client mistakes (should be 400/404/409, reserving 500 for genuine server-side failures).

## 💡 Best practices

- ✅ Match the HTTP method to the actual semantics of the operation (idempotent replace → PUT, partial update → PATCH, creation → POST).
- ✅ Return specific, accurate status codes — don't default everything to 200 or 500.
- ✅ Use `201 Created` with a `Location` header for successful resource creation, so clients know where to find the new resource.
- ✅ Distinguish 401 (not authenticated) from 403 (authenticated but not authorized) precisely.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                  | Answer                                                                                                                    |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between PUT and PATCH?                              | PUT replaces the entire resource; PATCH updates only the specified fields                                                 |
| What does "idempotent" mean, and give an idempotent HTTP method.          | Calling it once or many times produces the same end result; GET, PUT, and DELETE are idempotent                           |
| What's the difference between 401 and 403?                                | 401 = not authenticated at all (server doesn't know who you are); 403 = authenticated, but not authorized for this action |
| What status code should a successful POST that creates a resource return? | 201 Created (ideally with a Location header pointing to the new resource)                                                 |
| When should you return a 5xx vs a 4xx status code?                        | 4xx when the CLIENT made a mistake (bad input, not found, unauthorized); 5xx when something failed on the SERVER side     |

## 📝 30-second Revision Cheat Sheet

- Methods: GET (read, safe+idempotent), POST (create, neither), PUT (full replace, idempotent), PATCH (partial update), DELETE (remove, idempotent).
- Status codes: 2xx = success, 4xx = client's fault, 5xx = server's fault.
- 401 = "who are you?" (not authenticated); 403 = "I know you, but no" (not authorized).
- 201 Created for successful POST; 204 No Content for successful DELETE/PUT with no body to return.
- Never return 200 for everything with errors hidden in the body — use real status codes so tooling (caching, retries, gateways) works correctly.
