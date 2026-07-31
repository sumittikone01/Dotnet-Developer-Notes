
# 06 — Controller vs ControllerBase

---

## 🎯 One-Line Definition

> **`ControllerBase` is the lean base class for API controllers that return JSON — `Controller` extends it with everything needed for MVC (Views, ViewBag, Razor) — pick `ControllerBase` for pure API work, `Controller` when you return HTML views.**

---

## 🔷 The Inheritance Chain

```
Microsoft.AspNetCore.Mvc
│
└── ControllerBase
    │   ← ALL HTTP response helpers: Ok(), NotFound(), BadRequest()...
    │   ← Request/Response access: HttpContext, Request, Response, User
    │   ← URL helpers: Url.Action(), Url.RouteUrl()
    │   ← ModelState, TryValidateModel
    │   ← RouteData, ControllerContext
    │
    └── Controller  (inherits everything from ControllerBase)
        │
        ├── + View(), PartialView(), ViewComponent()
        ├── + Json()
        ├── + ViewBag  (dynamic object)
        ├── + ViewData (dictionary)
        ├── + TempData (survives redirect)
        └── + RedirectToPage(), Challenge(), SignIn(), SignOut()
```

```
ControllerBase   →   Use for API controllers (JSON responses)
Controller       →   Use for MVC controllers (HTML views + JSON)

Controller IS-A ControllerBase.
Everything in ControllerBase is available in Controller.
Controller just has MORE on top.
```

---

## 🔷 What Each Has — Complete Side-by-Side

```
┌──────────────────────────────────┬─────────────────────────────┐
│  ControllerBase                  │  Controller (adds these)    │
├──────────────────────────────────┼─────────────────────────────┤
│                                  │                             │
│  HTTP RESPONSE HELPERS:          │  VIEW RESULTS:              │
│  Ok()                            │  View()                     │
│  NotFound()                      │  PartialView()              │
│  BadRequest()                    │  ViewComponent()            │
│  Created() / CreatedAtAction()   │                             │
│  NoContent()                     │  DATA PASSING TO VIEW:      │
│  Unauthorized()                  │  ViewBag  (dynamic)         │
│  Forbid()                        │  ViewData (dictionary)      │
│  Conflict()                      │  TempData (post-redirect)   │
│  StatusCode()                    │                             │
│                                  │  JSON (explicit):           │
│  REQUEST ACCESS:                 │  Json()                     │
│  HttpContext                     │                             │
│  Request                         │  AUTH HELPERS:              │
│  Response                        │  Challenge()                │
│  User (ClaimsPrincipal)          │  SignIn()                   │
│  RouteData                       │  SignOut()                  │
│                                  │                             │
│  VALIDATION:                     │  PAGES (Razor Pages):       │
│  ModelState                      │  RedirectToPage()           │
│  TryValidateModel()              │                             │
│                                  │                             │
│  URL HELPERS:                    │                             │
│  Url.Action()                    │                             │
│  Url.RouteUrl()                  │                             │
│  Url.Content()                   │                             │
└──────────────────────────────────┴─────────────────────────────┘
```

---

## 🔷 When to Use Which — The Rule

```
Returning HTML to a browser?
  → Use Controller

Returning JSON to AJAX / Kendo / mobile app?
  → Use ControllerBase

Mixed — some actions return Views, some return JSON?
  → Use Controller (it has everything)
  → But keep API actions in a separate API controller
```

---

## 🔷 Full Code Examples

```csharp
// ── MVC Controller — inherits Controller ────────────────────────────

public class EmployeeController : Controller
//                                ↑ Controller (not ControllerBase)
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    // Returns HTML view — only possible with Controller
    public IActionResult Index()
    {
        var employees = _bal.GetAll();

        ViewBag.PageTitle    = "Employee List";     // ← Controller only
        ViewData["SubTitle"] = "All Employees";     // ← Controller only
        TempData["Success"]  = Session.GetString("msg"); // ← Controller only

        return View(employees);                     // ← Controller only
    }

    // Also returns partial view (HTML fragment)
    public IActionResult GetTableRow(int id)
    {
        var emp = _bal.GetById(id);
        return PartialView("_EmployeeRow", emp);    // ← Controller only
    }

    // Can also return JSON (Controller inherits ControllerBase)
    public IActionResult GetJson(int id)
    {
        var emp = _bal.GetById(id);
        return Ok(emp);     // ← works — inherited from ControllerBase
    }

    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid)
            return View(emp);                       // ← Controller only

        _bal.Add(emp);
        TempData["Success"] = "Created!";           // ← Controller only
        return RedirectToAction("Index");
    }
}
```

```csharp
// ── API Controller — inherits ControllerBase ────────────────────────

[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
//                                    ↑ ControllerBase (not Controller)
{
    private readonly EmployeeBAL _bal;
    public EmployeeApiController(EmployeeBAL bal) => _bal = bal;

    [HttpGet]
    public IActionResult GetAll(int skip = 0, int take = 10)
    {
        var data  = _bal.GetPaged(skip, take, out int total);
        return Ok(new { data, total });     // ← inherited from ControllerBase
    }

    [HttpGet("{id:int}")]
    public IActionResult GetById(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null)
            return NotFound(new { message = $"Employee {id} not found" });
        return Ok(emp);
    }

    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        int newId = _bal.Add(emp);
        emp.EmpId = newId;
        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    }

    [HttpPut("{id:int}")]
    public IActionResult Update(int id, [FromBody] Employee emp)
    {
        if (id != emp.EmpId) return BadRequest(new { message = "ID mismatch" });
        bool ok = _bal.Update(emp);
        if (!ok) return NotFound();
        return NoContent();
    }

    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id)
    {
        bool ok = _bal.Delete(id);
        if (!ok) return NotFound();
        return NoContent();
    }

    // ← View(), ViewBag, TempData etc. DO NOT EXIST here
    //   Calling View() would cause a compilation error
}
```

---

## 🔷 [ApiController] Attribute — Works With ControllerBase

```csharp
// [ApiController] is an attribute on API controllers — separate from inheritance
// It works alongside : ControllerBase to add smart API behaviors

[ApiController]          // ← Attribute: adds API-specific behaviors
[Route("api/employee")]
public class EmployeeApiController : ControllerBase  // ← Inheritance: provides helpers
{
}
```

```
What [ApiController] adds on top of ControllerBase:
──────────────────────────────────────────────────────────────
1. Auto Model Validation
   → If ModelState.IsValid is false, returns 400 automatically
   → Your action method is never called with invalid data

2. Automatic [FromBody] Inference
   → Complex type in POST/PUT → [FromBody] inferred
   → Simple type not in route → [FromQuery] inferred
   → You don't have to write [FromBody] every time

3. Structured Error Responses (ProblemDetails)
   → Validation errors returned as RFC 7807 ProblemDetails format:
   → { "type": "...", "title": "...", "errors": { "Name": ["required"] } }
   → Consistent format your AJAX error handler can always rely on

4. Requires Attribute Routing
   → [Route("api/...")] is required
   → Conventional routing ({controller}/{action}) doesn't work
   → Makes API routes explicit and predictable
```

```csharp
// Without [ApiController] — you write this in every POST action:
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
//                           ↑ must write [FromBody] explicitly
{
    if (!ModelState.IsValid)                    // ← must check manually
        return BadRequest(ModelState);
    // ...
}

// With [ApiController] — these become automatic:
[HttpPost]
public IActionResult Create(Employee emp)
//                           ↑ [FromBody] inferred automatically
{
    // If invalid → 400 returned automatically. This line is never reached.
    // No manual ModelState check needed.
    // ...
}
```

---

## 🔷 Your Real Project — Both Controllers Side by Side

```
EmployeeManagement/
├── Controllers/
│   │
│   ├── EmployeeController.cs        ← : Controller
│   │                                   Returns Views for the browser
│   │                                   Has ViewBag, TempData etc.
│   │                                   URLs: /Employee/Index, /Employee/Create
│   │
│   └── API/
│       └── EmployeeApiController.cs ← : ControllerBase + [ApiController]
│                                       Returns JSON for Kendo/AJAX
│                                       No views, no ViewBag
│                                       URLs: /api/employee, /api/employee/5
```

```csharp
// Both controllers use the SAME BAL — one source of truth:

// MVC Controller (renders the page):
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public IActionResult Index()
    {
        return View(_bal.GetAll());  // same BAL call
    }
}

// API Controller (serves data to Kendo):
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
{
    private readonly EmployeeBAL _bal;
    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(_bal.GetAll());   // same BAL call, different return type
    }
}
```

---

## 🔷 Common Mistake — Wrong Base Class

```csharp
// ❌ MISTAKE: Using Controller for API
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : Controller  // ← should be ControllerBase
{
    // Works, but:
    // Loads View infrastructure that API controllers don't need
    // Wastes memory (ViewBag, TempData etc. all initialized)
    // Misleading — developer seeing it thinks it returns views
}

// ❌ MISTAKE: Using ControllerBase for MVC
public class EmployeeController : ControllerBase  // ← should be Controller
{
    public IActionResult Index()
    {
        return View();   // ← COMPILATION ERROR: 'ControllerBase' does not
                         //   contain a definition for 'View'
    }
}

// ✅ CORRECT:
public class EmployeeController    : Controller        // MVC — returns HTML
public class EmployeeApiController : ControllerBase    // API — returns JSON
//   + [ApiController] attribute on the API controller
```

---

## 🔷 Summary — The Decision Table

```
┌──────────────────────────────────────────────────────────────────┐
│         You need to...              Use...                        │
├──────────────────────────────────────────────────────────────────┤
│  Return HTML views (Razor)          : Controller                  │
│  Use ViewBag / ViewData             : Controller                  │
│  Use TempData                       : Controller                  │
│  Return JSON to AJAX / Kendo        : ControllerBase              │
│  Build a REST API                   : ControllerBase + [ApiController] │
│  Return both views AND JSON         : Controller                  │
│  Maximize performance (API only)    : ControllerBase              │
└──────────────────────────────────────────────────────────────────┘

What both have in common (from ControllerBase):
  Ok(), NotFound(), BadRequest(), CreatedAtAction(), NoContent()
  HttpContext, Request, Response, User
  ModelState, RouteData, Url
```

---

## ⭐ Interview Quick-Fire

| Question                                                             | Answer                                                                                                                                        |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| What is the difference between `Controller`and `ControllerBase`? | `ControllerBase`has HTTP response helpers only.`Controller`extends it with View(), ViewBag, TempData — everything needed for Razor views |
| Which base class should an API controller inherit from?              | `ControllerBase`— it's lighter, no view infrastructure loaded                                                                              |
| Which base class should an MVC controller inherit from?              | `Controller`— it has View(), ViewBag, TempData etc.                                                                                        |
| Can a `ControllerBase`controller call `return View()`?           | ❌ No — compilation error.`View()`only exists in `Controller`                                                                            |
| Can a `Controller`call `return Ok(data)`?                        | ✅ Yes —`Ok()`is in `ControllerBase`, which `Controller`inherits                                                                       |
| What does `[ApiController]`do that `ControllerBase`doesn't?      | Auto validates ModelState, infers `[FromBody]`for complex types, returns structured ProblemDetails errors                                   |
| Is `[ApiController]`the same as `: ControllerBase`?              | No —`[ApiController]`is an attribute that adds behaviors.`: ControllerBase`is the inheritance that provides helpers                      |
| Why not just use `Controller`for everything?                       | API controllers using `: Controller`load unnecessary view infrastructure — wastes memory and misleads developers                           |
| In your project, which controllers use which base class?             | `EmployeeController : Controller`for MVC pages,`EmployeeApiController : ControllerBase`with `[ApiController]`for Kendo/AJAX             |
