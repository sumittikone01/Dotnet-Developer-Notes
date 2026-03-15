
# 03 — Route Constraints

---

## 🎯 One-Line Definition

> **Route Constraints are rules added directly to route parameters that restrict what values a URL segment can contain — if the URL doesn't match the constraint, the route doesn't match at all and ASP.NET Core moves on to the next route, returning 404 if nothing matches.**

---

## 🔷 What Problem Constraints Solve

```
WITHOUT constraints:
──────────────────────────────────────────────────────────────
Route: /Employee/Details/{id}

Request: GET /Employee/Details/5      → id = "5"   ← intended
Request: GET /Employee/Details/abc    → id = "abc" ← NOT intended
Request: GET /Employee/Details/-99   → id = "-99"  ← NOT intended

Your action runs in all three cases.
You then crash inside the action:
  int id = int.Parse("abc");  → FormatException
  _bal.GetById(-99);          → returns null or DB error

WITH constraints:
──────────────────────────────────────────────────────────────
Route: /Employee/Details/{id:int}

Request: GET /Employee/Details/5      → id = 5     ✅ route matches
Request: GET /Employee/Details/abc    →             ❌ route doesn't match → 404
Request: GET /Employee/Details/-99   → id = -99    ✅ matches (int, just negative)
```

---

## 🔷 Constraint Syntax

```
Route template syntax:
{parameterName:constraintName}
{parameterName:constraintName(argument)}
{parameterName:constraint1:constraint2}   ← chain multiple

Examples:
{id:int}                    ← id must be an integer
{id:int:min(1)}             ← id must be int AND minimum value 1
{name:alpha:minlength(2)}   ← name must be letters AND at least 2 chars
{slug:regex(^[a-z0-9-]+$)} ← slug must match regex pattern
```

---

## 🔷 All Built-In Constraints — Complete Reference

### Type Constraints

```csharp
// ── Numeric ────────────────────────────────────────────────────────
{id:int}        // 32-bit integer: 0, 5, -3, 2147483647
{id:long}       // 64-bit integer: large numbers
{id:decimal}    // decimal: 5.5, 19.99
{id:double}     // double: 3.14
{id:float}      // float

// ── Other types ────────────────────────────────────────────────────
{id:bool}       // true or false (case-insensitive): true, True, TRUE, false
{id:guid}       // GUID format: 3fa85f64-5717-4562-b3fc-2c963f66afa6
{id:datetime}   // valid datetime string: 2024-01-15

// Examples:
[Route("employee/{id:int}")]               // /employee/5
[Route("report/{date:datetime}")]          // /report/2024-01-15
[Route("toggle/{active:bool}")]            // /toggle/true
[Route("file/{fileId:guid}")]              // /file/3fa85f64-5717-4562-b3fc-2c963f66afa6
```

### Value Range Constraints

```csharp
{id:min(1)}           // integer >= 1         (no zero, no negative)
{id:max(100)}         // integer <= 100
{id:range(1,100)}     // 1 <= id <= 100       (inclusive both ends)

// Examples:
[HttpGet("{id:int:min(1)}")]
// /employee/5   ✅    /employee/0  ❌    /employee/-1  ❌

[HttpGet("{page:int:range(1,50)}")]
// /page/1  ✅    /page/50  ✅    /page/51  ❌    /page/0  ❌
```

### String Constraints

```csharp
{name:alpha}               // letters only: a-z, A-Z (no digits, no spaces)
{name:length(5)}           // exactly 5 characters
{name:minlength(2)}        // at least 2 characters
{name:maxlength(50)}       // at most 50 characters
{name:length(2,50)}        // between 2 and 50 characters (inclusive)
{code:regex(^[A-Z]{3}$)}   // exactly 3 uppercase letters (regex)

// Examples:
[HttpGet("{code:alpha:length(3)}")]
// /products/PEN  ✅    /products/AB  ❌    /products/123  ❌

[HttpGet("{slug:regex(^[a-z0-9-]+$)}")]
// /article/my-post-title  ✅    /article/My Post  ❌
```

### Optional and Required

```csharp
{id}          // required — URL must include this segment
{id?}         // optional — segment may be absent
{id:int?}     // optional integer

// Examples:
[HttpGet("{id:int?}")]
// /employee/5    → id = 5
// /employee/     → id = null (int?)
// /employee      → id = null (int?)
```

---

## 🔷 Constraints in Conventional Routing

```csharp
// Program.cs — conventional routes with constraints
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id:int?}");
//                                              ↑ id is optional integer

// Named route with constraint:
app.MapControllerRoute(
    name:    "employeeDetails",
    pattern: "employees/{id:int:min(1)}",
    defaults: new { controller = "Employee", action = "Details" });
// /employees/5   ✅    /employees/0   ❌ (min 1)    /employees/abc  ❌
```

---

## 🔷 Constraints in Attribute Routing

```csharp
[ApiController]
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    // ── Basic type constraint ──────────────────────────────────────
    [HttpGet("{id:int}")]
    public IActionResult GetById(int id)
    // /api/employees/5    ✅
    // /api/employees/abc  ❌ → 404, this action never called

    // ── Range constraint ──────────────────────────────────────────
    [HttpGet("{id:int:min(1)}")]
    public IActionResult GetById(int id)
    // /api/employees/0   ❌ → 404
    // /api/employees/-5  ❌ → 404
    // /api/employees/1   ✅

    // ── Chained constraints ───────────────────────────────────────
    [HttpGet("page/{page:int:min(1):max(100)}")]
    public IActionResult GetPage(int page)
    // /api/employees/page/1    ✅
    // /api/employees/page/100  ✅
    // /api/employees/page/101  ❌
    // /api/employees/page/0    ❌

    // ── String constraints ────────────────────────────────────────
    [HttpGet("code/{code:alpha:length(3)}")]
    public IActionResult GetByCode(string code)
    // /api/employees/code/EMP  ✅
    // /api/employees/code/12   ❌ (not alpha)
    // /api/employees/code/EMPL ❌ (not length 3)

    // ── GUID constraint ───────────────────────────────────────────
    [HttpGet("external/{externalId:guid}")]
    public IActionResult GetByExternalId(Guid externalId)
    // /api/employees/external/3fa85f64-5717-4562-b3fc-2c963f66afa6  ✅
    // /api/employees/external/123  ❌

    // ── Regex constraint ──────────────────────────────────────────
    [HttpGet("dept/{deptCode:regex(^[A-Z]{{2,4}}$)}")]
    public IActionResult GetByDept(string deptCode)
    // /api/employees/dept/HR    ✅ (2 uppercase letters)
    // /api/employees/dept/ITDV  ✅ (4 uppercase letters)
    // /api/employees/dept/hr    ❌ (lowercase)
    // /api/employees/dept/X     ❌ (too short)
    // Note: {{ and }} in route regex = escaped { and } in C# string interpolation
}
```

---

## 🔷 Constraint Resolves Route Ambiguity

```csharp
// WITHOUT constraints — AMBIGUOUS: both could match /employee/5
[HttpGet("{id}")]           // matches /employee/5  ← conflict
[HttpGet("{name}")]         // matches /employee/5  ← conflict
// → AmbiguousMatchException at startup

// WITH constraints — UNAMBIGUOUS:
[HttpGet("{id:int}")]       // matches /employee/5   (it's an int)
[HttpGet("{name:alpha}")]   // matches /employee/John (it's letters)
// /employee/5    → first action  (5 is int, not alpha)
// /employee/John → second action (John is alpha, not int)
// No conflict — constraints differentiate them
```

```csharp
// Real example — same controller, overlapping routes resolved by constraints:
[ApiController]
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    // Matches: /api/employees/5
    [HttpGet("{id:int:min(1)}")]
    public IActionResult GetById(int id) { }

    // Matches: /api/employees/johndoe
    [HttpGet("{username:alpha}")]
    public IActionResult GetByUsername(string username) { }

    // Matches: /api/employees/EMP-001 (letters, digits, hyphens)
    [HttpGet("{empCode:regex(^[A-Z]{{3}}-\\d{{3}}$)}")]
    public IActionResult GetByCode(string empCode) { }
}
```

---

## 🔷 Custom Route Constraint

```csharp
// When built-in constraints don't cover your rule — build your own

// Step 1: Implement IRouteConstraint
public class EvenNumberConstraint : IRouteConstraint
{
    public bool Match(
        HttpContext?         httpContext,
        IRouter?             route,
        string               routeKey,
        RouteValueDictionary values,
        RouteDirection       routeDirection)
    {
        if (values.TryGetValue(routeKey, out var value))
        {
            if (int.TryParse(value?.ToString(), out int number))
                return number % 2 == 0;   // true = matches, false = doesn't match
        }
        return false;
    }
}

// Step 2: Register in Program.cs
builder.Services.AddRouting(options =>
{
    options.ConstraintMap.Add("even", typeof(EvenNumberConstraint));
    //                          ↑ name used in route template
});

// Step 3: Use in routes
[HttpGet("{id:even}")]
public IActionResult GetEvenOnly(int id)
// /employee/4   ✅ (4 is even)
// /employee/5   ❌ (5 is odd → 404)
```

---

## 🔷 Constraints vs Validation — Key Difference

```
ROUTE CONSTRAINT:
  Lives in the route template
  Checked BEFORE your action is called
  Fail = 404 Not Found (route doesn't match)
  Purpose: route selection / URL format enforcement

MODEL VALIDATION:
  Lives in Data Annotations on your model class
  Checked INSIDE your action (after model binding)
  Fail = 400 Bad Request (action runs, ModelState.IsValid = false)
  Purpose: business data rules

EXAMPLE:
  Route:  /Employee/Details/{id:int:min(1)}
    → {id:int}    = constraint: URL must be integer, else 404
    → {id:min(1)} = constraint: must be >= 1, else 404

  Model:  [Range(1, 9999, ErrorMessage = "ID out of range")]
    → validation: returns 400 with error message if fails

Both check the same value — but at different stages and for different purposes.
Use route constraints for URL format enforcement.
Use validation for business rules and helpful error messages.
```

---

## 🔷 Complete Constraint Reference — Quick Lookup

```
┌─────────────────────────┬──────────────────────────────────────────────┐
│  Constraint             │  Matches                                     │
├─────────────────────────┼──────────────────────────────────────────────┤
│  :int                   │  32-bit integer                              │
│  :long                  │  64-bit integer                              │
│  :decimal               │  decimal number                              │
│  :double                │  double                                      │
│  :bool                  │  true / false                                │
│  :guid                  │  GUID format                                 │
│  :datetime              │  valid date/time string                      │
│  :min(n)                │  integer >= n                                │
│  :max(n)                │  integer <= n                                │
│  :range(n,m)            │  n <= value <= m                             │
│  :alpha                 │  letters only (a-z, A-Z)                     │
│  :length(n)             │  exactly n characters                        │
│  :length(n,m)           │  n to m characters                           │
│  :minlength(n)          │  at least n characters                       │
│  :maxlength(n)          │  at most n characters                        │
│  :regex(pattern)        │  matches regex pattern                       │
│  :required              │  value must be non-empty                     │
└─────────────────────────┴──────────────────────────────────────────────┘
```

---

## ⭐ Interview Quick-Fire

| Question                                                            | Answer                                                                                                                                        |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| What is a route constraint?                                         | A rule in the route template that restricts what values a URL segment can have — if it fails, the route doesn't match (404)                  |
| What does `{id:int}`do?                                           | Constrains the `id`segment to integers only — non-integer URLs return 404                                                                  |
| What is the difference between `{id:int}`and `{id:int:min(1)}`? | `int`only requires it to be an integer.`int:min(1)`additionally requires it to be 1 or greater                                            |
| What happens when a route constraint fails?                         | The route doesn't match — 404 returned. The action is never called                                                                           |
| What is the difference between a constraint and model validation?   | Constraint = URL format check before action runs → 404 on failure. Validation = data rule check inside action → 400 on failure              |
| How do constraints resolve route ambiguity?                         | Two routes like `{id:int}`and `{name:alpha}`can both exist — constraints make them unambiguous by restricting which URLs each matches    |
| How do you create a custom constraint?                              | Implement `IRouteConstraint`, register it in `builder.Services.AddRouting(o => o.ConstraintMap.Add(...))`, use it as `{param:yourname}` |
| What does `{id:int?}`mean?                                        | Optional integer — the segment may be absent; if present, it must be an integer                                                              |
