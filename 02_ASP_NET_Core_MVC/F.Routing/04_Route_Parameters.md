
# 04 — Route Parameters

---

## 🎯 One-Line Definition

> **Route Parameters are named placeholders in a URL template wrapped in curly braces — ASP.NET Core captures whatever is in that position of the URL and maps it to your action method's parameter by matching names.**

---

## 🔷 What a Route Parameter Is

```
URL template:    /Employee/Details/{id}
                                    ↑
                           Route Parameter
                           Captures whatever is at this position

Actual request:  GET /Employee/Details/5
                                        ↑
                                 Captured value: id = "5"

Actual request:  GET /Employee/Details/99
                                         ↑
                                 Captured value: id = "99"

The {id} placeholder captures ANYTHING in that URL position.
```

---

## 🔷 How Parameter Names Are Matched

```csharp
// The route parameter name must EXACTLY match the action method
// parameter name (case-insensitive):

// Route:  {id}     →  method parameter: int id       ✅
// Route:  {Id}     →  method parameter: int id       ✅ (case-insensitive)
// Route:  {empId}  →  method parameter: int empId    ✅
// Route:  {empId}  →  method parameter: int id       ❌ mismatch → id = 0

// ── Conventional routing — from route pattern ──────────────────────
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");

public IActionResult Details(int id)   // "id" matches {id} → id = 5
{
}

// ── Attribute routing — from [HttpGet] template ────────────────────
[HttpGet("{id:int}")]
public IActionResult GetById(int id)   // "id" matches {id} → populated
{
}

[HttpGet("{empId:int}")]
public IActionResult GetById(int empId)  // "empId" matches {empId} → populated
{
}
```

---

## 🔷 Required Parameters

```csharp
// Required parameter — URL MUST include this segment
// No default, no question mark

[HttpGet("{id:int}")]
public IActionResult GetById(int id) { }
// /employee/5    ✅  id = 5
// /employee/     ❌  no route match (id is required)
// /employee      ❌  no route match

// Multiple required parameters:
[HttpGet("{year:int}/{month:int}")]
public IActionResult GetByYearMonth(int year, int month) { }
// /reports/2024/3   ✅  year = 2024, month = 3
// /reports/2024     ❌  month is missing → no match

// Segment in the middle:
[HttpGet("dept/{deptId:int}/employees/{empId:int}")]
public IActionResult GetDeptEmployee(int deptId, int empId) { }
// /dept/2/employees/5  ✅  deptId = 2, empId = 5
// /dept/employees/5    ❌  deptId missing → no match
```

---

## 🔷 Optional Parameters

```csharp
// Optional parameter — URL may or may not include this segment
// Marked with ? after the name
// Must provide a default value in the method signature

[HttpGet("{id:int?}")]
public IActionResult GetById(int? id = null)
//                           ↑ nullable because segment may be absent
// /employee/5   → id = 5
// /employee/    → id = null
// /employee     → id = null

// Optional with default value:
[HttpGet("page/{page:int?}")]
public IActionResult GetPaged(int page = 1)
//                             ↑ default = 1 when segment absent
// /employee/page/3  → page = 3
// /employee/page/   → page = 1 (default)
// /employee/page    → page = 1 (default)

// In conventional routing — {id?} common pattern:
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
//                                              ↑ optional id
// /Home/Index/5  → id = 5
// /Home/Index    → id = null
```

---

## 🔷 Default Values

```csharp
// Default values fill in when the segment is missing OR when
// the URL omits the controller/action entirely

// In conventional routing:
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
//                       ↑             ↑
//              controller default   action default

// /            → controller = "Home", action = "Index"
// /Employee    → controller = "Employee", action = "Index"
// /Employee/Create → controller = "Employee", action = "Create"
// /Employee/Details/5 → controller = "Employee", action = "Details", id = "5"

// Inline default in attribute routing:
[HttpGet("{id:int}/{format=json}")]
public IActionResult Export(int id, string format)
// /export/5         → id = 5, format = "json"  (default)
// /export/5/excel   → id = 5, format = "excel"
// /export/5/csv     → id = 5, format = "csv"
```

---

## 🔷 Catch-All Parameters

```csharp
// Catch-all parameter {*param} captures the ENTIRE remaining URL path
// including slashes

[HttpGet("files/{*filePath}")]
public IActionResult GetFile(string filePath)
// /files/docs/reports/2024/january.pdf
//       → filePath = "docs/reports/2024/january.pdf"  ← slashes included

// /files/logo.png
//       → filePath = "logo.png"

// Without catch-all:
[HttpGet("files/{folder}/{fileName}")]
// /files/docs/report.pdf   → folder = "docs", fileName = "report.pdf"  ✅
// /files/docs/sub/report   ❌ too many segments → no match

// With catch-all:
[HttpGet("files/{folder}/{*rest}")]
// /files/docs/sub/deep/report.pdf
//       → folder = "rest", rest = "sub/deep/report.pdf"

// Real use case — catch-all for SPA fallback:
app.MapControllerRoute(
    name:    "spa-fallback",
    pattern: "{*path}",
    defaults: new { controller = "Home", action = "Index" });
// Every unknown URL → HomeController.Index() → return SPA shell
```

---

## 🔷 Multiple Parameters — Patterns You Use Daily

```csharp
// ── Single ID ────────────────────────────────────────────────────
[HttpGet("{id:int}")]
public IActionResult GetById(int id) { }
// /api/employees/5

// ── Nested resource (RESTful sub-resource) ────────────────────────
[HttpGet("{deptId:int}/employees")]
public IActionResult GetByDept(int deptId) { }
// /api/departments/2/employees

[HttpGet("{deptId:int}/employees/{empId:int}")]
public IActionResult GetDeptEmployee(int deptId, int empId) { }
// /api/departments/2/employees/5

// ── Parameters with literal segments in between ───────────────────
[HttpGet("from/{fromDate:datetime}/to/{toDate:datetime}")]
public IActionResult GetRange(DateTime fromDate, DateTime toDate) { }
// /api/reports/from/2024-01-01/to/2024-12-31

// ── Version in route ──────────────────────────────────────────────
[Route("api/v{version:int}/employees")]
public class EmployeeV2Controller : ControllerBase { }
// /api/v1/employees
// /api/v2/employees

// ── Mixed route and query ─────────────────────────────────────────
// Route:  /api/employees/5/reports?year=2024
[HttpGet("{id:int}/reports")]
public IActionResult GetReports(int id, int year = DateTime.Now.Year)
// id   = 5    ← from route
// year = 2024 ← from query string ?year=2024
```

---

## 🔷 Parameter Binding Source — Automatic Rules

```csharp
// ASP.NET Core figures out WHERE to read each parameter from:

[HttpGet("{id:int}/reports")]
public IActionResult GetReports(
    int    id,          // IN route template → reads from URL route segment
    int    year = 2024, // NOT in route      → reads from ?year=2024 query string
    string sort = "asc")// NOT in route      → reads from ?sort=desc query string
{ }

// URL: GET /employee/5/reports?year=2024&sort=desc
// id   = 5     (from route segment)
// year = 2024  (from query string)
// sort = "desc"(from query string)
```

```
Inference rule (without [ApiController] attribute, with attribute routing):
──────────────────────────────────────────────────────────────
Parameter name exists in route template   →  [FromRoute]
Parameter name NOT in route template
  AND simple type (int, string, bool...)  →  [FromQuery]
  AND complex type (class)                →  [FromBody]  (POST/PUT)
```

---

## 🔷 Route Parameter vs Query String — When to Use Which

```
USE ROUTE PARAMETER when:
──────────────────────────────────────────────────────────────
✅ Identifies a SPECIFIC resource
   /api/employees/5           ← "which employee" = route param
   /api/departments/2/head    ← "which department" = route param

✅ Required — always present
   ID of a record, resource identifier

✅ Short, clean-looking URLs
   /report/2024/january  ← year/month as route params

USE QUERY STRING when:
──────────────────────────────────────────────────────────────
✅ Optional filters, sort, search
   /api/employees?dept=HR&active=true&sort=salary

✅ Pagination — skip/take sent by Kendo
   /api/employees?skip=0&take=10

✅ Multiple optional values
   /api/employees?minSalary=30000&maxSalary=80000&dept=IT

DO NOT:
──────────────────────────────────────────────────────────────
❌ /api/employees?id=5         → use /api/employees/5 instead
❌ /api/employees/HR/active/30000/80000  → use query string for filters
```

---

## 🔷 Route Parameters in MVC Controller

```csharp
public class EmployeeController : Controller
{
    // ── Conventional route parameter ──────────────────────────────
    // Route: /Employee/Details/{id?}
    public IActionResult Details(int? id)
    {
        if (!id.HasValue) return RedirectToAction("Index");
        var emp = _bal.GetById(id.Value);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // ── Generating URLs with route parameters in views ─────────────
    // Tag Helper:
    // <a asp-action="Details" asp-route-id="@emp.EmpId">View</a>
    // → <a href="/Employee/Details/5">View</a>

    // RedirectToAction with route values:
    public IActionResult Create(Employee emp)
    {
        int newId = _bal.Add(emp);
        return RedirectToAction("Details", new { id = newId });
        // → redirects to /Employee/Details/5
    }
}
```

---

## 🔷 Everything Together — Real API Controller

```csharp
[ApiController]
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    private readonly EmployeeBAL _bal;
    public EmployeeApiController(EmployeeBAL bal) => _bal = bal;

    // GET /api/employees?skip=0&take=10&dept=HR&active=true
    [HttpGet]
    public IActionResult GetAll(
        int    skip   = 0,
        int    take   = 10,
        string dept   = null,
        bool?  active = null)
    // All from query string — none in route template
    {
        var data = _bal.GetPaged(skip, take, dept, active, out int total);
        return Ok(new { data, total });
    }

    // GET /api/employees/5
    [HttpGet("{id:int:min(1)}")]
    public IActionResult GetById(int id)    // required, from route
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound(new { message = $"Employee {id} not found" });
        return Ok(emp);
    }

    // GET /api/employees/5/payslips?year=2024&month=3
    [HttpGet("{id:int:min(1)}/payslips")]
    public IActionResult GetPayslips(
        int id,          // required, from route /employees/5/payslips
        int year  = 2024,// optional, from ?year=2024
        int month = 0)   // optional, from ?month=3
    {
        var slips = _bal.GetPayslips(id, year, month);
        return Ok(slips);
    }

    // PUT /api/employees/5
    [HttpPut("{id:int:min(1)}")]
    public IActionResult Update(
        int      id,   // from route
        [FromBody] Employee emp)  // from JSON body
    {
        if (id != emp.EmpId)
            return BadRequest(new { message = "Route id does not match body id" });
        bool ok = _bal.Update(emp);
        if (!ok) return NotFound();
        return NoContent();
    }

    // DELETE /api/employees/5
    [HttpDelete("{id:int:min(1)}")]
    public IActionResult Delete(int id)    // required, from route
    {
        bool ok = _bal.Delete(id);
        if (!ok) return NotFound();
        return NoContent();
    }
}
```

---

## 🔷 Route Parameter Summary — Quick Reference

```
┌──────────────────────────────┬────────────────────────────────────────────┐
│  Syntax                      │  Meaning                                   │
├──────────────────────────────┼────────────────────────────────────────────┤
│  {id}                        │  Required, captures any string             │
│  {id:int}                    │  Required, must be integer                 │
│  {id:int:min(1)}             │  Required, integer >= 1                    │
│  {id?}                       │  Optional, any string                      │
│  {id:int?}                   │  Optional integer                          │
│  {page:int=1}                │  Integer with default value 1              │
│  {*filePath}                 │  Catch-all, captures rest of URL + slashes │
│  {name:alpha}                │  Required, letters only                    │
│  {code:length(3)}            │  Required, exactly 3 chars                 │
│  {slug:regex(^[a-z0-9-]+$)} │  Required, matches regex                   │
└──────────────────────────────┴────────────────────────────────────────────┘
```

---

## ⭐ Interview Quick-Fire

| Question                                                                             | Answer                                                                                                                                                                             |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What is a route parameter?                                                           | A named placeholder `{name}`in a URL template that captures the value at that URL position and maps it to an action method parameter                                             |
| How does ASP.NET Core match a route parameter to a method parameter?                 | By name — the route parameter name `{id}`must match the method parameter name `int id`(case-insensitive)                                                                      |
| What is an optional route parameter?                                                 | `{id?}`— the segment may be absent. The method parameter must be nullable or have a default value                                                                               |
| What is a catch-all parameter?                                                       | `{*param}`— captures the entire remaining URL including slashes. Used for file paths or SPA fallback routes                                                                     |
| What happens when a required route parameter is missing from the URL?                | The route doesn't match — 404 returned                                                                                                                                            |
| How do you add a default value to a route parameter?                                 | `{page:int=1}`— if the segment is absent,`page`defaults to 1                                                                                                                  |
| What is the difference between a route parameter and a query string parameter?       | Route = part of the URL path `/employee/5`. Query string = after `?`in the URL `/employee?id=5`. Route is for resource identification; query string is for filtering/sorting |
| In `[HttpGet("{id:int}/reports")]`what is the source of `int id`vs `int year`? | `id`comes from the route segment.`year`comes from the query string (`?year=2024`) — inferred because it's not in the route template                                         |
