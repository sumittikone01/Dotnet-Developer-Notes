# 01 — Conventional Routing

## 📌 What is it?

**Conventional routing** defines URL patterns **centrally** (in `Program.cs`) using a template that maps segments of the URL to `{controller}`, `{action}`, and parameter placeholders. Instead of specifying routes on every Controller/Action, one (or a few) route templates handle matching for the *entire* application by convention.

## 🤔 Why do we need it?

For traditional MVC apps with many Controllers, defining a route on every single action would be repetitive. Conventional routing lets you define the URL "shape" once, and every Controller/Action that follows the naming convention is automatically reachable — no per-action configuration needed.

## 💻 Code example

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

This single line defines the URL shape for the **entire app**:

| URL                     | Matches                                                  |
| ----------------------- | -------------------------------------------------------- |
| `/`                   | `HomeController.Index()` (both segments use defaults)  |
| `/Product`            | `ProductController.Index()` (action defaults to Index) |
| `/Product/Details`    | `ProductController.Details()`                          |
| `/Product/Details/42` | `ProductController.Details(int id)` — `id` = 42     |

## 🖼 Anatomy of the route template

```
{controller=Home}/{action=Index}/{id?}
     │                  │            │
     │                  │            └── optional parameter (the ? makes it optional)
     │                  └── defaults to "Index" if not in URL
     └── defaults to "Home" if not in URL
```

- `{controller}` — matches the Controller class name (minus the "Controller" suffix)
- `{action}` — matches the method name inside that Controller
- `{id?}` — an optional route parameter, bound to an action parameter of the same name
- `=Home` / `=Index` — default values used when that segment is missing from the URL

## ⚙️ How matching actually works

1. Incoming URL: `/Product/Details/42`
2. ASP.NET Core splits it into segments: `Product`, `Details`, `42`
3. Matches against the template: `controller=Product`, `action=Details`, `id=42`
4. Looks for `ProductController` with an action method `Details` that accepts a parameter compatible with `id`

## 📊 Multiple route definitions (order matters)

```csharp
app.MapControllerRoute(
    name: "productSpecial",
    pattern: "products/{category}/{id}",
    defaults: new { controller = "Product", action = "Details" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

Routes are evaluated **top to bottom** — the first matching route wins. So more specific routes should be registered **before** the general default route.

## 🚨 Common mistakes

- Defining a more general route **before** a specific one — the general route "wins" first and the specific one never gets a chance to match.
- Forgetting the `?` on optional parameters, causing `/Product` (without an id) to 404 even though you intended `id` to be optional.
- Assuming conventional routing and attribute routing (next topic) can't coexist — they absolutely can, and modern ASP.NET Core apps often mix both.

## 💡 Best practices

- Use conventional routing for **traditional MVC apps with Views** where URL patterns are fairly uniform (`/Controller/Action/id`).
- Prefer **Attribute Routing** (next topic) for Web APIs, where URL patterns tend to be more varied and resource-oriented (`/api/products/{id}/reviews`).
- Keep route ordering in mind — register specific/custom routes before the catch-all default route.

## 🎤 Interview questions

1. What does `{controller=Home}/{action=Index}/{id?}` mean, segment by segment?
2. Why does route registration order matter with conventional routing?
3. How would you add a custom route for `/products/electronics/42` that still maps to `ProductController.Details()`?
4. What's the difference between a route default value and an optional route parameter?

## 📝 30-second revision cheat sheet

- Conventional routing = one central template (`{controller}/{action}/{id?}`) applies app-wide.
- Registered in `Program.cs` via `app.MapControllerRoute(...)`.
- `?` = optional parameter, `=value` = default value.
- Route order matters — more specific routes go first

# 01 — Conventional Routing

---

## 🎯 One-Line Definition

> **Conventional Routing maps an incoming URL to a Controller and Action using a pattern defined once in `Program.cs` — the URL structure `{controller}/{action}/{id?}` drives all routing automatically without touching individual controllers.**

---

## 🔷 What Routing Does

```
Browser sends: GET /Employee/Details/5
                         │
                         ▼
ASP.NET Core Routing Engine reads the URL
                         │
                         ▼
Matches pattern: {controller}/{action}/{id?}
  controller = "Employee"
  action     = "Details"
  id         = 5
                         │
                         ▼
Finds: EmployeeController.Details(int id)
Calls it with id = 5
                         │
                         ▼
Returns response to browser
```

---

## 🔷 Defining the Default Route — Program.cs

```csharp
// Program.cs
var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();       // ← enables routing middleware
app.UseAuthorization();

// Define conventional route
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
    //        ↑               ↑              ↑
    //  segment name    default value   optional (?)
);

app.Run();
```

---

## 🔷 Route Pattern — Breaking It Down

```
Pattern:  {controller=Home}/{action=Index}/{id?}

Segment           Meaning
──────────────────────────────────────────────────────────────
{controller=Home} → Maps to controller name (strips "Controller")
                    Default: "Home" → HomeController
                    e.g. "Employee" → EmployeeController

{action=Index}    → Maps to action method name
                    Default: "Index"
                    e.g. "Create" → Create()

{id?}             → Optional route parameter
                    ? = optional — URL works with OR without it
                    e.g. "5" → int id = 5
                    e.g. (missing) → id = null or default
```

---

## 🔷 URL Examples — What Maps to What

```
URL                            Controller      Action      id
──────────────────────────────────────────────────────────────
/                              Home            Index       null
/Home                          Home            Index       null
/Home/Index                    Home            Index       null
/Employee                      Employee        Index       null
/Employee/Index                Employee        Index       null
/Employee/Create               Employee        Create      null
/Employee/Details/5            Employee        Details     5
/Employee/Edit/10              Employee        Edit        10
/Employee/Delete/3             Employee        Delete      3
/Department/Index              Department      Index       null
/Report/Annual/2024            Report          Annual      2024
```

---

## 🔷 Default Values — How They Work

```csharp
pattern: "{controller=Home}/{action=Index}/{id?}"
//                    ↑              ↑
//              default=Home   default=Index

// URL: /
//   controller not given → uses "Home" → HomeController
//   action not given     → uses "Index" → Index()
//   id not given         → null

// URL: /Employee
//   controller = "Employee" → EmployeeController
//   action not given        → uses "Index" → Index()
//   id not given            → null

// URL: /Employee/Details/5
//   controller = "Employee" → EmployeeController
//   action     = "Details"  → Details()
//   id         = 5
```

---

## 🔷 How the Route Name Works

```csharp
app.MapControllerRoute(
    name:    "default",       // ← name used in Url.RouteUrl("default", ...)
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

// The name is mainly used for URL generation in code:
string url = Url.RouteUrl("default", new { controller = "Employee", action = "Index" });
// = "/Employee/Index"

// But in practice you use Url.Action() — much easier:
string url2 = Url.Action("Index", "Employee");
// = "/Employee/Index"
```

---

## 🔷 Multiple Routes — Order Matters

Routes are checked top to bottom — first match wins:

```csharp
// Register more specific routes BEFORE less specific ones

// Route 1: special admin route
app.MapControllerRoute(
    name:    "admin",
    pattern: "admin/{controller=Dashboard}/{action=Index}/{id?}"
);

// Route 2: default app route
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

// URL: /admin/Employee/Index
//   → matches Route 1: AdminArea, EmployeeController, Index()
//   → Route 2 never checked

// URL: /Employee/Index
//   → doesn't match Route 1 (no "admin" prefix)
//   → matches Route 2: EmployeeController, Index()
```

---

## 🔷 Generating URLs from Routes — In Controllers and Views

### In Controllers:

```csharp
// Redirect using route values
return RedirectToAction("Index", "Employee");
// Generates: /Employee/Index

return RedirectToAction("Details", "Employee", new { id = 5 });
// Generates: /Employee/Details/5

// Generate URL string
string url = Url.Action("Edit", "Employee", new { id = 5 });
// = "/Employee/Edit/5"

string url2 = Url.Action("Index", "Employee");
// = "/Employee/Index"
```

### In Views:

```cshtml
@* asp-action and asp-controller generate the URL via routing *@

<a asp-controller="Employee" asp-action="Index">All Employees</a>
@* Generates: <a href="/Employee/Index">All Employees</a> *@

<a asp-controller="Employee" asp-action="Details" asp-route-id="@emp.Id">
    View
</a>
@* Generates: <a href="/Employee/Details/5">View</a> *@

<a asp-controller="Employee" asp-action="Edit" asp-route-id="@emp.Id">
    Edit
</a>
@* Generates: <a href="/Employee/Edit/5">Edit</a> *@

@* Form action *@
<form asp-controller="Employee" asp-action="Create" method="post">
@* Generates: <form action="/Employee/Create" method="post"> *@
```

---

## 🔷 Area Routing — Organising Large Apps

For large apps with multiple sections:

```csharp
// Register area route BEFORE the default route
app.MapControllerRoute(
    name:    "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}"
);

app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);
```

```csharp
// Areas/Admin/Controllers/EmployeeController.cs
[Area("Admin")]
public class EmployeeController : Controller
{
    public IActionResult Index() => View();
}

// URL: /Admin/Employee/Index
//   area = "Admin", controller = "Employee", action = "Index"
```

---

## 🔷 Conventional vs Attribute Routing — Quick Contrast

```
CONVENTIONAL ROUTING:
  Defined ONCE in Program.cs
  All controllers follow the same pattern
  Great for standard MVC apps
  URL structure: /Controller/Action/id

ATTRIBUTE ROUTING:
  Defined ON each controller or action with [Route("...")]
  Each action can have its own URL pattern
  Great for APIs, custom URL shapes
  URL structure: whatever you define

Use conventional for standard MVC pages.
Use attribute for Web APIs and custom URL patterns.
```

---

## ⚠️ Common Conventional Routing Mistakes

| Mistake                                    | What Happens                 | Fix                                                   |
| ------------------------------------------ | ---------------------------- | ----------------------------------------------------- |
| `app.UseRouting()`not called             | 404 for all routes           | Add`app.UseRouting()`before `MapControllerRoute`  |
| More specific route after default          | Specific route never matched | Always register specific routes BEFORE default        |
| Controller name not ending in "Controller" | Not found by routing         | Class must be`EmployeeController`, not `Employee` |
| `{id}`without `?`                      | URL without id = 404         | Use`{id?}`for optional id                           |

---

## ⭐ Interview Quick-Fire

| Question                                      | Answer                                                                                                            |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| What is Conventional Routing?                 | Routing defined once in`Program.cs`using a pattern —`{controller}/{action}/{id?}`maps all URLs automatically |
| Where is the default route defined?           | `app.MapControllerRoute()`in `Program.cs`                                                                     |
| What does`{controller=Home}`mean?           | The controller segment with a default value of "Home" — used when no controller in URL                           |
| What does`{id?}`mean?                       | `?`makes id optional — URL works with or without it                                                            |
| What URL does`/`map to by default?          | `HomeController.Index()`— both controller and action use their defaults                                        |
| Which route wins when multiple match?         | The first one registered — order in`Program.cs`matters                                                         |
| How to generate a URL from a route in a view? | `asp-controller="Employee" asp-action="Index"`Tag Helpers                                                       |
| How to generate a URL in a controller?        | `Url.Action("Index", "Employee")`or `RedirectToAction("Index", "Employee")`                                   |
