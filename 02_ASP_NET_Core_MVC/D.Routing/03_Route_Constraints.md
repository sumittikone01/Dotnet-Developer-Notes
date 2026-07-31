# 03 — Route Constraints

## 📌 What is it?

**Route constraints** restrict what values a route parameter will accept — enforcing type, format, range, or custom rules directly at the routing level, before the request even reaches your Controller action.

## 🤔 Why do we need it?

Without constraints, `/Product/Details/{id}` would match `/Product/Details/abc` just as readily as `/Product/Details/42` — the string `"abc"` simply fails to bind to an `int` parameter, often producing a confusing error instead of a clean `404`. Constraints let routing itself reject invalid URLs early.

## 💻 Code examples

### Inline constraints (attribute routing)

```csharp
[HttpGet("{id:int}")]                         // must be an integer
public IActionResult GetById(int id) { ... }

[HttpGet("{id:int:min(1)}")]                   // integer AND >= 1
public IActionResult GetById(int id) { ... }

[HttpGet("{slug:alpha}")]                      // letters only
public IActionResult GetBySlug(string slug) { ... }

[HttpGet("{date:datetime}")]                   // must parse as DateTime
public IActionResult GetByDate(DateTime date) { ... }

[HttpGet("{code:length(6)}")]                  // exactly 6 characters
public IActionResult GetByCode(string code) { ... }
```

### Conventional routing with constraints

```csharp
app.MapControllerRoute(
    name: "productById",
    pattern: "products/{id:int:min(1)}",
    defaults: new { controller = "Product", action = "Details" });
```

## 📊 Common built-in constraints

| Constraint              | Meaning                        | Example                               |
| ----------------------- | ------------------------------ | ------------------------------------- |
| `int`                 | Must be a valid integer        | `{id:int}`                          |
| `bool`                | Must be`true`/`false`      | `{flag:bool}`                       |
| `datetime`            | Must parse as a valid DateTime | `{date:datetime}`                   |
| `decimal`             | Must be a valid decimal        | `{price:decimal}`                   |
| `guid`                | Must be a valid GUID           | `{id:guid}`                         |
| `alpha`               | Letters only (a-z, A-Z)        | `{slug:alpha}`                      |
| `length(n)`           | Exact string length            | `{code:length(6)}`                  |
| `length(min,max)`     | String length range            | `{code:length(3,10)}`               |
| `min(n)` / `max(n)` | Numeric range bound            | `{id:int:min(1)}`                   |
| `range(min,max)`      | Numeric range                  | `{age:range(18,65)}`                |
| `regex(pattern)`      | Custom regex match             | `{code:regex(^[A-Z]{{3}}\d{{3}}$)}` |

## 🖼 What happens without vs with a constraint

```
Route: {id}            (no constraint)
GET /Product/Details/abc  →  Matches route → binding fails → id defaults to 0
                             (silent bug: wrong product loaded, or unexpected behavior)

Route: {id:int}        (with constraint)
GET /Product/Details/abc  →  Route does NOT match at all → falls through to next route
                             or results in a clean 404 if no other route matches
```

This is the key benefit: constraints reject bad input **before** it reaches your action, rather than letting it silently bind to a default/incorrect value.

## ⚙️ Custom route constraints (advanced)

For rules not covered by built-ins, you can create a class implementing `IRouteConstraint`:

```csharp
public class EvenNumberConstraint : IRouteConstraint
{
    public bool Match(HttpContext httpContext, IRouter route, string routeKey,
        RouteValueDictionary values, RouteDirection routeDirection)
    {
        if (values.TryGetValue(routeKey, out var value) && int.TryParse(value?.ToString(), out int number))
        {
            return number % 2 == 0;
        }
        return false;
    }
}

// Register in Program.cs
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("even", typeof(EvenNumberConstraint));
});

// Usage: [HttpGet("{id:even}")]
```

## 🚨 Common mistakes

- Relying only on `int id` as the C# parameter type without an explicit `{id:int}` constraint — model binding will fail gracefully for the action, but routing may still "match" first, leading to less clean error responses in some scenarios.
- Overusing `regex()` constraints for things better validated in the Controller/Model (constraints are best for **shape/type** checks, not full business validation).
- Forgetting that constraints affect **route matching**, not full validation — a value can pass a constraint but still be semantically invalid (e.g., `{id:int}` accepts `999999` even if no product with that ID exists).

## 💡 Best practices

- Always constrain numeric/typed route parameters (`{id:int}`, `{id:guid}`) — it's a cheap, effective first line of defense and produces cleaner 404s for malformed URLs.
- Use constraints for **shape validation** (is this the right type/format?), and leave **business validation** (does this ID actually exist?) to the Controller/Service layer.
- Combine constraints when useful: `{id:int:min(1)}` is more precise than `{id:int}` alone.

## 🎤 Interview questions

1. What problem do route constraints solve that plain route parameters don't?
2. What's the difference in behavior between `/Product/Details/{id}` and `/Product/Details/{id:int}` when given a non-numeric value?
3. How would you create a custom route constraint, and when might you need one?
4. Do route constraints replace the need for Model validation? Why or why not?

## 📝 30-second revision cheat sheet

- Route constraints restrict what values a route segment accepts (`{id:int}`, `{id:guid}`, `{slug:alpha}`, etc.).
- They reject malformed URLs at the **routing level**, before reaching the Controller.
- Custom constraints implement `IRouteConstraint` and get registered in `RouteOptions`.
- Constraints check **shape/type**, not business rules — still validate business logic separately.
