# 04_Routing_in_Web_API

> **Attribute Routing** = defining routes directly on Controllers/Actions via `[Route]`/`[HttpGet]` attributes — the ONLY routing style supported by `[ApiController]`-decorated controllers (recall `02_ApiController_Attribute.md`'s "attribute routing requirement").

> Continues **K.Web_API**. Builds on `03_HTTP_Verbs.md`'s verb attributes — this chapter is specifically about the ROUTE TEMPLATES those attributes carry, and how route parameters, constraints, and tokens work.

## 📌 What is it?

```csharp
[ApiController]
[Route("api/products")]             // CONTROLLER-level base route
public class ProductsApiController : ControllerBase
{
    [HttpGet]                        // → GET api/products
    public IActionResult GetAll() => ...

    [HttpGet("{id:int}")]            // → GET api/products/5 — route PARAMETER with a TYPE CONSTRAINT
    public IActionResult GetById(int id) => ...

    [HttpGet("category/{name}")]     // → GET api/products/category/electronics
    public IActionResult GetByCategory(string name) => ...
}
```

Unlike conventional MVC routing (one global route pattern registered in `Program.cs` for traditional Controllers), attribute routing puts the route DEFINITION right next to the action it applies to.

## 🤔 Why do we need to understand route templates in depth?

| Need                                                              | How route templates help                                                                  |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Different actions on the SAME controller need different sub-paths | Each action's own`[HttpGet("...")]` appends to the controller's base route              |
| A route parameter should only accept numbers                      | Route constraints (`{id:int}`) reject non-matching requests before the action even runs |
| Multiple route "shapes" should reach the SAME action              | Multiple`[HttpGet("...")]` attributes can be stacked on one action                      |
| Nested resources need clear, hierarchical URLs                    | Route templates naturally express this (`products/{productId}/reviews/{reviewId}`)      |

## 🌍 Real-world analogy

Attribute routing is like **labeling each office door with its own specific room number sign**, rather than having one single master directory at the building entrance listing every room (conventional routing). The sign is right there, on the door — you don't need to cross-reference a separate central list to know what's behind it.

## 📊 Route Template Tokens

| Token            | Meaning                                                                           |
| ---------------- | --------------------------------------------------------------------------------- |
| `{id}`         | A route parameter — captures a URL segment, binds to a matching action parameter |
| `{id:int}`     | Route CONSTRAINT — only matches if the segment is a valid integer                |
| `{id:int?}`    | Optional parameter — the`?` makes it not required                              |
| `{id:int=1}`   | Default value — used if the segment is omitted                                   |
| `[controller]` | Replaced with the Controller's name (minus "Controller" suffix) automatically     |
| `[action]`     | Replaced with the Action method's name automatically                              |

```csharp
[Route("api/[controller]")]  // for "ProductsApiController" → becomes "api/products" automatically
public class ProductsApiController : ControllerBase { ... }
```

## 📊 Common Route Constraints

| Constraint                  | Matches                             |
| --------------------------- | ----------------------------------- |
| `{id:int}`                | Integer only                        |
| `{id:guid}`               | A valid GUID                        |
| `{slug:alpha}`            | Alphabetic characters only          |
| `{price:decimal}`         | A decimal number                    |
| `{date:datetime}`         | A valid date/time                   |
| `{id:min(1)}`             | Integer, minimum value 1            |
| `{category:length(3,20)}` | String with length between 3 and 20 |

```csharp
[HttpGet("{id:int:min(1)}")] // combines MULTIPLE constraints — must be an int AND at least 1
public IActionResult GetById(int id) => ...

// GET api/products/5   → matches
// GET api/products/abc → does NOT match this route (falls through, likely 404)
// GET api/products/0   → does NOT match (fails the min(1) constraint)
```

## ⚙️ Internal working — route template COMBINATION (controller + action)

```
[Route("api/products")]          ← Controller-level base
public class ProductsApiController : ControllerBase
{
    [HttpGet("{id}/reviews")]     ← Action-level ADDITION
    public IActionResult GetReviews(int id) => ...
}

FINAL route = "api/products" + "/" + "{id}/reviews"
            = "api/products/{id}/reviews"

Request: GET /api/products/5/reviews → MATCHES, id = 5
```

## 🖼 Nested resources — a common real-world pattern

```csharp
[Route("api/products/{productId}/reviews")]
public class ProductReviewsApiController : ControllerBase
{
    [HttpGet]                    // GET api/products/5/reviews
    public IActionResult GetAll(int productId) => Ok(_service.GetReviewsForProduct(productId));

    [HttpGet("{reviewId}")]      // GET api/products/5/reviews/42
    public IActionResult GetOne(int productId, int reviewId) =>
        Ok(_service.GetReview(productId, reviewId));
}
```

## 💻 Code examples

### Basic — a complete, well-structured attribute-routed controller

```csharp
[ApiController]
[Route("api/[controller]")] // resolves to "api/products" automatically
public class ProductsController : ControllerBase
{
    [HttpGet]                                  // GET api/products
    public IActionResult GetAll() => Ok(_service.GetAllProducts());

    [HttpGet("{id:int}")]                       // GET api/products/5
    public IActionResult GetById(int id) => Ok(_service.GetProductById(id));

    [HttpGet("search")]                         // GET api/products/search?term=widget
    public IActionResult Search([FromQuery] string term) => Ok(_service.SearchProducts(term));

    [HttpPost]                                  // POST api/products
    public IActionResult Create(ProductViewModel model) => Ok(_service.CreateProduct(model));
}
```

### Intermediate — using route constraints to disambiguate similar routes

```csharp
[HttpGet("{id:int}")]           // matches GET api/products/5
public IActionResult GetById(int id) => ...

[HttpGet("{slug:alpha}")]       // matches GET api/products/widget-pro (alphabetic, not numeric)
public IActionResult GetBySlug(string slug) => ...

// Without the :int and :alpha constraints, these two routes would be AMBIGUOUS —
// the framework wouldn't know which one "api/products/5" or "api/products/widget" should match.
```

### Practical — multiple route attributes on ONE action

```csharp
[HttpGet("")]          // GET api/products
[HttpGet("all")]       // ALSO matches GET api/products/all
public IActionResult GetAll() => Ok(_service.GetAllProducts());
```

### Practical — a route with an optional parameter and default value

```csharp
[HttpGet("page/{pageNumber:int=1}")] // GET api/products/page or api/products/page/3
public IActionResult GetPage(int pageNumber) => Ok(_service.GetPage(pageNumber));
```

## ⚡ Performance considerations

- Route matching happens once per request, early in the pipeline (`UseRouting()`, per `I.Middleware_and_Filters/02_Built_in_Middleware.md`) — the overhead is negligible regardless of how many routes exist, since ASP.NET Core builds an efficient internal structure for matching.
- Route constraints (`:int`, `:alpha`, etc.) actually IMPROVE routing precision and can prevent ambiguous-route exceptions that would otherwise only surface at runtime.

## 🚨 Common mistakes

- ❌ Defining two routes that can match the SAME URL ambiguously (e.g., two `{id}` parameters with no constraints, where one was meant for integers and another for strings) — causes an `AmbiguousMatchException` at runtime.
- ❌ Forgetting that `[ApiController]` requires attribute routing — attempting to rely on conventional routing (`MapControllerRoute`) for an `[ApiController]`-decorated controller doesn't work as expected.
- ❌ Overcomplicating route templates with too many constraints/segments when a simpler structure (or query parameters instead of route segments) would be clearer.
- ❌ Using `[controller]`/`[action]` tokens inconsistently across a codebase — mixing literal route strings and token-based routes makes the overall API surface harder to predict by convention.

## 💡 Best practices

- ✅ Use `[Route("api/[controller]")]` at the controller level for a consistent naming convention tied directly to the Controller's class name.
- ✅ Add route constraints (`:int`, `:guid`, etc.) whenever a parameter's type should be enforced at the ROUTING level, not just via the action parameter's C# type.
- ✅ Keep route templates shallow and readable — 1-2 levels of nesting (`products/{id}/reviews`) is usually the practical limit before considering query parameters instead.
- ✅ Be consistent: pick either token-based (`[controller]`) or literal route strings as your team's convention, and apply it uniformly.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                         | Answer                                                                                                                                           |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| What is attribute routing?                                                                       | Defining routes directly on Controllers/Actions via`[Route]`/`[HttpGet]` attributes, rather than a central, conventional route configuration |
| What does`{id:int}` mean in a route template?                                                  | A route parameter named`id`, constrained to only match if the segment is a valid integer                                                       |
| How does the final route get built when both the Controller and an Action have route attributes? | The Controller-level route is the base, and the Action-level route template is appended to it                                                    |
| What happens if two attribute routes could match the same incoming URL ambiguously?              | The framework throws an`AmbiguousMatchException` at request time                                                                               |
| What do the`[controller]` and `[action]` tokens do in a route template?                      | They're automatically replaced with the Controller's name (minus "Controller") and the Action method's name, respectively                        |

## 📝 30-second Revision Cheat Sheet

- Attribute routing = `[Route]`/`[HttpGet("...")]` directly on Controllers/Actions — required for `[ApiController]`.
- Route tokens: `{id}` (parameter), `{id:int}` (constraint), `{id:int?}` (optional), `{id:int=1}` (default).
- Final route = Controller-level base route + Action-level route template, combined.
- Use constraints (`:int`, `:guid`, `:alpha`) to disambiguate similar routes and catch invalid values at routing time.
- Ambiguous overlapping routes throw `AmbiguousMatchException` — constraints usually prevent this.
