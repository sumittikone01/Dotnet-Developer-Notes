
# 02 — Attribute Routing

---

## 🎯 One-Line Definition

> **Attribute Routing puts the URL pattern directly on the Controller class or Action method using `[Route("...")]` — giving you precise control over every URL in your app, independent of naming conventions.**

---

## 🔷 Conventional vs Attribute Routing — The Key Difference

```
CONVENTIONAL ROUTING (Program.cs):
──────────────────────────────────────────────────────────
One pattern controls ALL routes:
  pattern: "{controller}/{action}/{id?}"

  /Employee/Index   → EmployeeController.Index()
  /Employee/Create  → EmployeeController.Create()
  You don't control the URL — the convention does.

ATTRIBUTE ROUTING (on the class/method):
──────────────────────────────────────────────────────────
YOU define the exact URL for each endpoint:
  [Route("employees")]           → /employees
  [Route("employees/{id}")]      → /employees/5
  [Route("api/v1/employees")]    → /api/v1/employees

  Full control. No convention constraints.
  Perfect for Web APIs and custom URL designs.
```

---

## 🔷 `[Route]` on a Controller — Class-Level Prefix

```csharp
[Route("employees")]        // ← prefix for ALL actions in this controller
public class EmployeeController : Controller
{
    // GET /employees
    [Route("")]              // ← empty = just the prefix
    public IActionResult Index() => View(_bal.GetAll());

    // GET /employees/create
    [Route("create")]
    [HttpGet]
    public IActionResult Create() => View(new Employee());

    // POST /employees/create
    [Route("create")]
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);
        _bal.Insert(emp);
        return RedirectToAction(nameof(Index));
    }

    // GET /employees/5
    [Route("{id:int}")]
    public IActionResult Details(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // GET /employees/5/edit
    [Route("{id:int}/edit")]
    [HttpGet]
    public IActionResult Edit(int id) => View(_bal.GetById(id));
}
```

---

## 🔷 `[Route]` on Actions Only — Without Class-Level Route

```csharp
public class EmployeeController : Controller
{
    // GET /all-employees
    [Route("all-employees")]
    public IActionResult Index() => View(_bal.GetAll());

    // GET /employee/5/details
    [Route("employee/{id}/details")]
    public IActionResult Details(int id) => View(_bal.GetById(id));

    // GET /employee/new
    [Route("employee/new")]
    public IActionResult Create() => View(new Employee());
}
```

---

## 🔷 HTTP Verb Attributes — Combine Route + Verb

HTTP verb attributes (`[HttpGet]`, `[HttpPost]`) can also carry the route:

```csharp
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    // GET /api/employees
    [HttpGet]
    public IActionResult GetAll() => Ok(_bal.GetAll());

    // GET /api/employees/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var emp = _bal.GetById(id);
        return emp == null ? NotFound() : Ok(emp);
    }

    // POST /api/employees
    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        int newId = _bal.InsertGetId(emp);
        emp.Id = newId;
        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    }

    // PUT /api/employees/5
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Employee emp)
    {
        if (id != emp.Id) return BadRequest("ID mismatch.");
        int rows = _bal.Update(emp);
        return rows > 0 ? NoContent() : NotFound();
    }

    // DELETE /api/employees/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        int rows = _bal.Delete(id);
        return rows > 0 ? NoContent() : NotFound();
    }
}
```

---

## 🔷 `[controller]` and `[action]` Tokens

Use tokens to avoid repeating names:

```csharp
// WITHOUT tokens — names hardcoded
[Route("employee")]
public class EmployeeController : Controller
{
    [Route("index")]
    public IActionResult Index() => View();
}

// WITH tokens — automatically replaced with class/method name
[Route("[controller]")]           // ← replaced with "Employee" (strips "Controller")
public class EmployeeController : Controller
{
    [Route("[action]")]            // ← replaced with method name "Index"
    public IActionResult Index() => View();

    [Route("[action]/{id?}")]      // ← "Edit" + optional id
    public IActionResult Edit(int id) => View(_bal.GetById(id));
}

// Combined — most common Web API pattern:
[Route("api/[controller]")]
public class EmployeeController : ControllerBase
{
    // GET /api/Employee
    [HttpGet]
    public IActionResult GetAll() => Ok(_bal.GetAll());

    // GET /api/Employee/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok(_bal.GetById(id));
}

// Tokens:
//  [controller] → class name without "Controller" suffix
//  [action]     → method name
//  [area]       → area name (if using areas)
```

---

## 🔷 Route Templates — Segment Syntax

```csharp
// Literal segment
[Route("employees/list")]         // must be: /employees/list

// Parameter segment
[Route("employees/{id}")]         // /employees/5  → id = 5

// Optional parameter
[Route("employees/{id?}")]        // /employees  OR  /employees/5

// Parameter with default
[Route("employees/{id=1}")]       // /employees  → id = 1 (default)

// Multiple parameters
[Route("employees/{deptId}/staff/{id}")]
// /employees/3/staff/7  → deptId=3, id=7

// Catch-all parameter (matches rest of URL)
[Route("files/{*filePath}")]
// /files/docs/2024/report.pdf  → filePath = "docs/2024/report.pdf"
```

---

## 🔷 Route Constraints in Attribute Routes

```csharp
// int constraint — only matches integers
[Route("employees/{id:int}")]         // /employees/5 ✅  /employees/abc ❌

// min/max value
[Route("employees/{id:int:min(1)}")]  // /employees/0 ❌  /employees/1 ✅

// string length
[Route("search/{term:minlength(3)}")] // /search/ab ❌  /search/ali ✅

// regex
[Route("code/{code:regex(^[A-Z]{{3}}[0-9]{{3}}$)}")] // /code/EMP001 ✅

// guid
[Route("sessions/{token:guid}")]      // only GUIDs

// alpha — letters only
[Route("departments/{name:alpha}")]   // /departments/IT ✅  /departments/3 ❌

// Common constraints:
// :int       :long      :double    :decimal
// :bool      :datetime  :guid      :alpha
// :minlength(n)  :maxlength(n)  :length(n)
// :min(n)    :max(n)    :range(n,m)
// :regex(pattern)
```

---

## 🔷 Multiple Routes on One Action

An action can respond to more than one URL:

```csharp
// Action responds to BOTH /employees AND /staff
[Route("employees")]
[Route("staff")]
public IActionResult Index() => View(_bal.GetAll());


// Action responds to /emp/5 AND /employee/5
[Route("emp/{id}")]
[Route("employee/{id}")]
public IActionResult Details(int id) => View(_bal.GetById(id));
```

---

## 🔷 Mixing Conventional and Attribute Routing

```csharp
// Conventional route in Program.cs still active
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");

// This controller overrides with attribute routes — attribute wins
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(_bal.GetAll());
    // URL: /api/employees  (attribute route)
    // NOT: /EmployeeApi/GetAll  (conventional pattern ignored)
}

// This controller uses conventional routing (no [Route] attribute)
public class EmployeeController : Controller
{
    public IActionResult Index() => View();
    // URL: /Employee/Index  (conventional route)
}
```

> 📌 **Rule:** If a controller has `[Route]` → attribute routing applies to that controller. Without `[Route]` → conventional routing applies.

---

## 🔷 URL Generation with Attribute Routes

```cshtml
@* In views — same Tag Helpers work *@
<a asp-controller="Employee" asp-action="Details" asp-route-id="5">View</a>
@* ASP.NET Core matches the attribute route and generates the URL *@

@* If route is [Route("employees/{id}")] *@
@* Generates: /employees/5 *@
```

```csharp
// In controllers:
return RedirectToAction("Index", "Employee");
// Generates URL based on attribute route for Employee.Index

string url = Url.Action("Details", "Employee", new { id = 5 });
// Generates: /employees/5  (based on [Route("employees/{id}")])
```

---

## 🔷 Conventional vs Attribute Routing — When to Use Which

| Use Case                                             | Routing Type |
| ---------------------------------------------------- | ------------ |
| Standard MVC CRUD pages (Employee, Department)       | Conventional |
| REST API endpoints (`api/employees`)               | Attribute    |
| Custom friendly URLs (`/our-team`,`/contact-us`) | Attribute    |
| Multiple URL patterns for same action                | Attribute    |
| App with mostly default `{controller}/{action}`    | Conventional |
| `[ApiController]`Web API controllers               | Attribute    |

---

## ⚠️ Common Attribute Routing Mistakes

| Mistake                                              | What Happens                                         | Fix                                                                    |
| ---------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| Forgetting `[HttpGet]`/`[HttpPost]`on same route | Ambiguous match — runtime error                     | Always specify HTTP verb on routes that differ only by verb            |
| `[Route]`on class but no route on action           | Only the class prefix — no route for action         | Add `[Route("[action]")]`or `[HttpGet]`with route on each action   |
| Missing `/`vs using `/`at start                  | Leading `/`= absolute route (ignores class prefix) | `[Route("{id}")]`not `[Route("/{id}")]`unless you want absolute    |
| Token `[controller]`in wrong case                  | Not found                                            | `[controller]`and `[action]`are case-sensitive — always lowercase |

---

## ⭐ Interview Quick-Fire

| Question                                              | Answer                                                                                               |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| What is Attribute Routing?                            | Placing `[Route("...")]`directly on controllers/actions to define exact URL patterns               |
| How does `[Route("[controller]")]`work?             | `[controller]`token is replaced with the controller name (minus "Controller" suffix)               |
| Can one action respond to multiple URLs?              | ✅ Yes — add multiple `[Route]`attributes to the same action                                      |
| Can you combine `[HttpGet("{id}")]`and `[Route]`? | ✅ Yes — HTTP verb attributes carry route templates too                                             |
| When does attribute routing override conventional?    | When the controller has any `[Route]`attribute — attribute routing takes over for that controller |
| What does `{id?}`mean in an attribute route?        | The `id`parameter is optional — action works with or without it                                   |
| What does `{id:int}`mean?                           | Route constraint — only matches if `id`is an integer                                              |
| What is the `[action]`token?                        | Replaced automatically with the action method's name                                                 |
