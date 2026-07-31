# 04 — Route Parameters

## 📌 What is it?

**Route parameters** are the named placeholders (`{id}`, `{category}`, etc.) inside a route template that capture segments of the URL and make them available as values — typically bound directly to Controller action method parameters.

## 🤔 Why do we need it?

URLs need to carry dynamic data (which product, which order, which page). Route parameters are the mechanism that extracts that data from the URL path itself (as opposed to query strings or the request body).

## 💻 Code examples

### Required parameter

```csharp
[HttpGet("{id}")]                       // /api/products/5
public IActionResult GetById(int id) { ... }
```

### Optional parameter

```csharp
[HttpGet("{id?}")]                      // /api/products  OR  /api/products/5
public IActionResult GetById(int? id) { ... }
```

### Parameter with default value

```csharp
[HttpGet("{page=1}")]                   // /api/products/page  →  page defaults to 1
public IActionResult GetPage(int page) { ... }
```

### Catch-all parameter

```csharp
[HttpGet("{*path}")]                    // /files/docs/2024/report.pdf
public IActionResult GetFile(string path) { ... }  // path = "docs/2024/report.pdf"
```

### Multiple parameters

```csharp
[HttpGet("{category}/{id:int}")]        // /products/electronics/42
public IActionResult GetByCategory(string category, int id) { ... }
```

## 🖼 Route parameter vs query string — where data can travel

```
URL: /Product/Details/42?highlight=true&ref=email

┌────────────────┐┌───┐┌──────────────────────────┐
│ /Product/Details││ 42││?highlight=true&ref=email │
└────────────────┘└───┘└──────────────────────────┘
   controller/action  ▲            ▲
                       │            └── Query string (bound via [FromQuery] or automatically)
                       └── Route parameter ({id})
```

Both eventually become available to the Controller action as method parameters — ASP.NET Core's model binding pulls from route values, query string, form data, etc. automatically (order of precedence covered in `H.Model_Handling`).

## 📊 Parameter types comparison

| Type          | Syntax       | Behavior                                                         |
| ------------- | ------------ | ---------------------------------------------------------------- |
| Required      | `{id}`     | URL must include this segment or the route won't match           |
| Optional      | `{id?}`    | URL may omit this segment; parameter becomes`null`/default     |
| Default value | `{id=1}`   | If omitted, uses the specified default instead of`null`        |
| Constrained   | `{id:int}` | Must satisfy the type/format constraint (see topic 03)           |
| Catch-all     | `{*path}`  | Captures the REST of the URL, including slashes, into one string |

## ⚙️ Parameter binding order (how ASP.NET Core decides where a value comes from)

For a Controller action parameter, ASP.NET Core checks, roughly in this order:

1. Route data (`{id}` in the URL path)
2. Query string (`?id=5`)
3. Form data (for POST requests)
4. Request body (for complex types, typically via `[FromBody]`)

You can override this default behavior explicitly with `[FromRoute]`, `[FromQuery]`, `[FromBody]`, `[FromForm]` attributes — covered in depth in `L.Web_API/05_FromBody_FromQuery_FromRoute.md`.

## 🚨 Common mistakes

- Using a catch-all parameter (`{*path}`) in the middle of a route instead of at the end — catch-all parameters must be the **last** segment in the template.
- Naming a route parameter differently than the Controller action's method parameter (`{productId}` in the route vs `int id` in the method signature) — binding silently fails, parameter stays at its default value (e.g., `0` for `int`).
- Forgetting that route parameters are always strings until bound/converted — constraints like `{id:int}` handle conversion validation, but the underlying URL segment is still just text.

## 💡 Best practices

- Keep route parameter names **identical** to the corresponding action method parameter names — avoids silent binding failures.
- Use optional parameters (`{id?}`) sparingly and only when the action logic genuinely supports both "with" and "without" cases — otherwise, prefer separate explicit routes for clarity.
- Reserve catch-all parameters (`{*path}`) for genuinely path-like data (file paths, wildcard proxying) — not general-purpose multi-value passing (use query strings or a request body for that).

## 🎤 Interview questions

1. What's the difference between a route parameter and a query string parameter, and how does ASP.NET Core decide which one to bind from?
2. When would you use a catch-all route parameter, and what restriction applies to where it can be placed?
3. What happens if a route parameter's name doesn't match the corresponding action method parameter's name?
4. How do optional route parameters (`{id?}`) interact with non-nullable value types like `int`?

## 📝 30-second revision cheat sheet

- Route parameters (`{id}`) capture dynamic segments of the URL path and bind to action method parameters.
- Types: required `{id}`, optional `{id?}`, default `{id=1}`, catch-all `{*path}` (must be last).
- Binding requires matching **names** between route template and method parameter.
- Model binding checks route data → query string → form data → body, in that general order.
