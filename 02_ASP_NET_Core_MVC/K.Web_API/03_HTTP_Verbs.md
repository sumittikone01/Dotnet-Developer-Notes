# 03_HTTP_Verbs

> How ASP.NET Core MAPS HTTP verbs to action methods via attributes (`[HttpGet]`, `[HttpPost]`, etc.) — the practical, framework-specific application of the HTTP method semantics already covered conceptually in `12_System_Design/I.API_Design/02_HTTP_Methods_and_Status_Codes.md`.

> Continues **K.Web_API**. Where that System Design chapter explained WHAT each verb means, this chapter is specifically HOW to wire that up in an ASP.NET Core Web API Controller.

## 📌 What is it?

```csharp
[ApiController]
[Route("api/products")]
public class ProductsApiController : ControllerBase
{
    [HttpGet]           public IActionResult GetAll() => ...
    [HttpGet("{id}")]   public IActionResult GetById(int id) => ...
    [HttpPost]          public IActionResult Create(ProductViewModel model) => ...
    [HttpPut("{id}")]   public IActionResult Update(int id, ProductViewModel model) => ...
    [HttpPatch("{id}")] public IActionResult PartialUpdate(int id, JsonPatchDocument<Product> patch) => ...
    [HttpDelete("{id}")] public IActionResult Delete(int id) => ...
}
```

Each `[Http*]` attribute tells the framework: "only route requests using THIS HTTP method to this action."

## 🤔 Why do we need explicit verb attributes?

| Without explicit verb attributes                                                                      | With them                                                                                            |
| ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| ASP.NET Core wouldn't know which action handles GET vs POST to the same route                         | Each action is matched to the EXACT verb it should respond to                                        |
| Multiple actions on the same route path would conflict ambiguously                                    | `[HttpGet]`/`[HttpPost]`/etc. on the SAME route path disambiguate by verb                        |
| REST semantics (`02_HTTP_Methods_and_Status_Codes.md`) wouldn't be enforceable at the routing level | The verb attribute IS the enforcement — a GET request literally cannot reach a`[HttpPost]` action |

## 🌍 Real-world analogy

Think of verb attributes like **different doors into the same building, each requiring a specific type of entry pass** — the "front door" (`[HttpGet]`) only lets in "viewing" passes, the "loading dock" (`[HttpPost]`) only accepts "delivery" passes. Both lead into the same building (the same URL path), but which door you can use depends entirely on what KIND of visit you're making.

## 📊 Verb Attributes Reference (recap from System Design, now as actual C# attributes)

| Attribute         | HTTP Verb | Typical use                                                        |
| ----------------- | --------- | ------------------------------------------------------------------ |
| `[HttpGet]`     | GET       | Retrieve data — safe, idempotent, cacheable                       |
| `[HttpPost]`    | POST      | Create a new resource                                              |
| `[HttpPut]`     | PUT       | Replace an existing resource entirely                              |
| `[HttpPatch]`   | PATCH     | Partially update a resource                                        |
| `[HttpDelete]`  | DELETE    | Remove a resource                                                  |
| `[HttpHead]`    | HEAD      | Like GET, but returns only headers, no body (rarely used directly) |
| `[HttpOptions]` | OPTIONS   | Used for CORS preflight requests (see`L.Security/10_CORS.md`)    |

## ⚙️ Internal working — multiple actions, same route, different verbs

```csharp
[Route("api/products/{id}")]
public class ProductsApiController : ControllerBase
{
    [HttpGet]
    public IActionResult Get(int id) => Ok(_service.GetProductById(id));

    [HttpPut]
    public IActionResult Update(int id, ProductViewModel model) => Ok(_service.UpdateProduct(id, model));

    [HttpDelete]
    public IActionResult Delete(int id)
    {
        _service.DeleteProduct(id);
        return NoContent();
    }
}
```

```
Request: GET /api/products/5     → routed to Get(int id)
Request: PUT /api/products/5     → routed to Update(int id, ProductViewModel model)
Request: DELETE /api/products/5  → routed to Delete(int id)

ALL THREE share the exact same URL path — the HTTP VERB is what
disambiguates which action method actually handles the request.
```

## 🖼 `[HttpPatch]` and `JsonPatchDocument` — the trickiest verb in practice

```csharp
[HttpPatch("{id}")]
public IActionResult PartialUpdate(int id, [FromBody] JsonPatchDocument<Product> patchDoc)
{
    var product = _service.GetProductById(id);
    if (product == null) return NotFound();

    patchDoc.ApplyTo(product, ModelState); // applies ONLY the specified field changes

    if (!ModelState.IsValid) return BadRequest(ModelState);

    _service.UpdateProduct(product);
    return NoContent();
}
```

```json
// A PATCH request body — "JSON Patch" format, describing ONLY the changes
[
  { "op": "replace", "path": "/price", "value": 45.00 }
]
```

> ⚠️ `PATCH` is genuinely more complex to implement correctly than `PUT` (it needs a patch document format and partial-update logic) — many real-world APIs simplify by just using `PUT` for all updates (sending the FULL object) and skip `PATCH` entirely, unless partial updates are a strong, specific requirement.

## 💻 Code examples

### Basic — combining route-level and verb-level routing

```csharp
[ApiController]
[Route("api/products")] // the "base" route for the whole controller
public class ProductsApiController : ControllerBase
{
    [HttpGet]                  // GET /api/products
    public IActionResult GetAll() => Ok(_service.GetAllProducts());

    [HttpGet("{id}")]          // GET /api/products/5 — "{id}" appends to the base route
    public IActionResult GetById(int id) => Ok(_service.GetProductById(id));

    [HttpGet("category/{categoryName}")] // GET /api/products/category/electronics
    public IActionResult GetByCategory(string categoryName) => Ok(_service.GetByCategory(categoryName));
}
```

### Intermediate — PUT (full replace) vs PATCH (partial update), side by side

```csharp
[HttpPut("{id}")]
public IActionResult Replace(int id, ProductViewModel model)
{
    // Expects the FULL object — any field NOT included is typically treated as cleared/defaulted
    _service.ReplaceProduct(id, model);
    return NoContent();
}

[HttpPatch("{id}")]
public IActionResult PartialUpdate(int id, JsonPatchDocument<Product> patchDoc)
{
    // Only the SPECIFIED fields change — everything else stays exactly as it was
    var product = _service.GetProductById(id);
    patchDoc.ApplyTo(product, ModelState);
    _service.UpdateProduct(product);
    return NoContent();
}
```

### Practical — using `[AcceptVerbs]` for a rare multi-verb action (uncommon, but good to recognize)

```csharp
[AcceptVerbs("GET", "HEAD")] // one action handling BOTH GET and HEAD
public IActionResult GetProductMetadata(int id) => Ok(_service.GetMetadata(id));
```

## ⚡ Performance considerations

- Verb-based routing itself has negligible overhead — the router matches the verb as part of its normal URL-matching process.
- `PATCH` with `JsonPatchDocument` has a bit more processing overhead (parsing the patch operations, applying them one at a time) compared to `PUT`'s simpler "replace everything" approach — rarely significant unless PATCH is used extremely heavily.

## 🚨 Common mistakes

- ❌ Using `[HttpPost]` for operations that are actually retrievals (should be `[HttpGet]`) — breaks caching assumptions and REST semantics (ties back to `12_System_Design/I.API_Design/02_HTTP_Methods_and_Status_Codes.md`).
- ❌ Forgetting that a GET request literally CANNOT reach a `[HttpPost]`-only action — a very common "404/405 Method Not Allowed" debugging trap for beginners who test a POST-only endpoint with a browser's address bar (which always sends GET).
- ❌ Implementing `PATCH` without proper validation of the resulting object after applying the patch — a patch operation could leave the object in an invalid state that wouldn't have been allowed via `POST`/`PUT`.
- ❌ Using `PUT` for a PARTIAL update (sending only some fields) — `PUT` semantically implies a full replace; unintentionally omitted fields may get cleared/defaulted.

## 💡 Best practices

- ✅ Match the HTTP verb to the actual operation semantics, per `02_HTTP_Methods_and_Status_Codes.md` — GET for reads, POST for creation, PUT for full replace, PATCH for partial update, DELETE for removal.
- ✅ Default to `PUT` (full object replace) unless there's a genuine, specific need for partial updates — it's simpler to implement correctly than `PATCH`.
- ✅ Always re-validate `ModelState` AFTER applying a `JsonPatchDocument`, since the patch can produce a state that wasn't directly validated by the original model binding.
- ✅ Test API endpoints with a proper HTTP client (Postman, curl) rather than a browser address bar, which only ever sends GET requests.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                           | Answer                                                                                                                                                 |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| How does ASP.NET Core know which action handles a GET vs a POST to the same route? | Each action is decorated with a verb attribute (`[HttpGet]`, `[HttpPost]`, etc.) that restricts it to that specific HTTP method                    |
| What's the difference between PUT and PATCH in practice?                           | PUT expects the full resource representation (a complete replace); PATCH applies only specific, partial changes via a patch document                   |
| Why would a GET request to a POST-only endpoint fail?                              | The router only matches requests whose HTTP verb matches an action's verb attribute — a GET simply can't reach a`[HttpPost]`-only action            |
| Why is PATCH considered more complex to implement correctly than PUT?              | It requires a patch document format (like JSON Patch) and logic to apply only the specified changes, plus re-validating the resulting object afterward |
| What does`[AcceptVerbs("GET", "HEAD")]` do?                                      | Lets a single action method handle more than one specific HTTP verb                                                                                    |

## 📝 30-second Revision Cheat Sheet

- `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]` map actions to specific HTTP verbs.
- Multiple actions can share the SAME route path, differentiated purely by verb.
- PUT = full replace (expects the whole object); PATCH = partial update (via a patch document, e.g. `JsonPatchDocument`).
- Always re-validate `ModelState` after applying a PATCH, since the resulting object wasn't directly model-bound.
- Many real-world APIs skip PATCH entirely and just use PUT for all updates, for simplicity.
