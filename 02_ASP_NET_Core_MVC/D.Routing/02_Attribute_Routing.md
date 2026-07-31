# 02 — Attribute Routing

## 📌 What is it?

**Attribute routing** defines routes **directly on Controllers and Action methods** using attributes like `[Route]`, `[HttpGet]`, `[HttpPost]`, etc. — instead of one central template, each endpoint declares its own URL pattern explicitly.

## 🤔 Why do we need it?

Web APIs typically need **fine-grained, resource-oriented URLs** that don't fit a single uniform `{controller}/{action}/{id}` pattern (e.g., `/api/products/{id}/reviews`, `/api/orders/{orderId}/items/{itemId}`). Attribute routing gives you full, explicit control per endpoint.

## 💻 Code example

```csharp
[ApiController]
[Route("api/[controller]")]           // becomes "api/Products"
public class ProductsController : ControllerBase
{
    [HttpGet]                          // GET api/products
    public IActionResult GetAll() => Ok(_service.GetAll());

    [HttpGet("{id}")]                  // GET api/products/5
    public IActionResult GetById(int id) => Ok(_service.GetById(id));

    [HttpGet("{id}/reviews")]          // GET api/products/5/reviews
    public IActionResult GetReviews(int id) => Ok(_service.GetReviews(id));

    [HttpPost]                         // POST api/products
    public IActionResult Create(Product product) => Ok(_service.Create(product));

    [HttpPut("{id}")]                  // PUT api/products/5
    public IActionResult Update(int id, Product product) => Ok(_service.Update(id, product));

    [HttpDelete("{id}")]               // DELETE api/products/5
    public IActionResult Delete(int id) => Ok(_service.Delete(id));
}
```

## 🖼 How `[controller]` token works

`[Route("api/[controller]")]` on `ProductsController` automatically becomes `api/Products` — the `[controller]` token is replaced with the Controller's name (minus "Controller" suffix) at compile time. This keeps routes in sync automatically if you rename the class.

## 📊 Conventional Routing vs Attribute Routing

| Aspect                   | Conventional Routing                | Attribute Routing                      |
| ------------------------ | ----------------------------------- | -------------------------------------- |
| Definition location      | Central, in`Program.cs`           | On each Controller/Action              |
| Best suited for          | Traditional MVC apps (Views)        | Web APIs, REST-style resources         |
| Flexibility per-endpoint | Low (follows the shared template)   | High (each route is explicit)          |
| Consistency              | High (uniform URL shape everywhere) | Requires discipline to stay consistent |
| Typical usage today      | MVC Controllers returning Views     | `[ApiController]` classes            |

## ⚙️ Combining route prefixes and HTTP verbs

```csharp
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpGet("{orderId}/items/{itemId}")]   // GET api/orders/10/items/3
    public IActionResult GetItem(int orderId, int itemId) { ... }
}
```

Route templates can be nested/combined across the class-level `[Route]` and the method-level `[HttpGet]`/`[HttpPost]` attributes — the final URL is the concatenation of both.

## 🚨 Common mistakes

- Mixing conventional and attribute routing on the **same Controller** inconsistently — pick one style per Controller for clarity (mixing across the whole app is fine; mixing within one Controller gets confusing).
- Forgetting `[ApiController]` on API controllers — you lose automatic model validation (`400 Bad Request` on invalid `ModelState`) and other API-specific conveniences.
- Creating ambiguous routes — two actions that could match the same URL pattern, causing an `AmbiguousMatchException` at runtime.

## 💡 Best practices

- Use attribute routing for **all Web API controllers** — it's the de facto standard and pairs naturally with `[ApiController]`.
- Keep route templates RESTful and resource-oriented: `/api/products/{id}/reviews`, not `/api/getProductReviews?id=5`.
- Use the `[controller]` token in class-level routes so renaming a Controller doesn't silently break its URLs.

## 🎤 Interview questions

1. What's the practical difference between conventional and attribute routing, and when would you choose each?
2. What does the `[controller]` token do inside a `[Route]` attribute?
3. What happens if two actions in the same Controller define routes that could match the same URL and HTTP verb?
4. What extra behavior does `[ApiController]` enable in relation to routing/model validation?

## 📝 30-second revision cheat sheet

- Attribute routing = routes declared directly via `[Route]`, `[HttpGet]`, etc. on Controllers/Actions.
- Best for Web APIs — precise, resource-oriented URLs.
- `[controller]` token auto-fills the Controller's name in route templates.
- Pairs with `[ApiController]` for automatic model validation and API conventions.
