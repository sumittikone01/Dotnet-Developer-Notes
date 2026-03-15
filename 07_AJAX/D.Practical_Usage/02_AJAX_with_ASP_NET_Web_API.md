
# 02 — AJAX with ASP.NET Web API

---

## 🎯 One-Line Definition

> **ASP.NET Web API is a controller that speaks pure JSON — no Views, no HTML — designed specifically to answer AJAX calls from your JavaScript.**

---

## 🔑 MVC Controller vs Web API Controller

You have two types of controllers in ASP.NET Core. Know the difference:

```
┌──────────────────────────────────────────────────────────────┐
│  MVC CONTROLLER                                              │
│  ─────────────                                               │
│  Inherits: Controller                                        │
│  Returns: Views (HTML), or JsonResult for AJAX              │
│  Used for: Page rendering + some AJAX endpoints             │
│  Example: EmployeeController.cs                             │
│                                                              │
│  public IActionResult Index() => View();          ← HTML    │
│  public JsonResult Read(...) => Json(data);       ← JSON    │
├──────────────────────────────────────────────────────────────┤
│  WEB API CONTROLLER                                          │
│  ──────────────────                                          │
│  Inherits: ControllerBase  (NOT Controller)                 │
│  Returns: JSON only — no Views                              │
│  Used for: Pure AJAX/API endpoints                          │
│  Example: EmployeeApiController.cs                          │
│                                                              │
│  public IActionResult Get() => Ok(data);          ← JSON    │
│  public IActionResult Post([FromBody] emp) => ... ← JSON    │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 Web API Controller — The Structure

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]                          // ← enables automatic JSON, validation errors
[Route("api/[controller]")]              // ← URL = /api/employee
public class EmployeeApiController : ControllerBase
//                                  ↑ ControllerBase, NOT Controller
{
    private readonly AppDbContext _db;

    public EmployeeApiController(AppDbContext db) => _db = db;

    // GET /api/employee
    [HttpGet]
    public IActionResult GetAll()
    {
        var employees = _db.Employees.ToList();
        return Ok(employees);            // 200 + JSON array
    }

    // GET /api/employee/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var emp = _db.Employees.Find(id);
        if (emp == null) return NotFound();   // 404
        return Ok(emp);                       // 200 + JSON object
    }

    // POST /api/employee
    [HttpPost]
    public IActionResult Create([FromBody] Employee employee)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);   // 400 + validation errors

        _db.Employees.Add(employee);
        _db.SaveChanges();
        return CreatedAtAction(nameof(GetById),
            new { id = employee.Id }, employee);  // 201 + saved object
    }

    // PUT /api/employee/5
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Employee employee)
    {
        if (id != employee.Id) return BadRequest();

        _db.Employees.Update(employee);
        _db.SaveChanges();
        return Ok(employee);             // 200 + updated object
    }

    // DELETE /api/employee/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var emp = _db.Employees.Find(id);
        if (emp == null) return NotFound();

        _db.Employees.Remove(emp);
        _db.SaveChanges();
        return NoContent();              // 204 — success, nothing to return
    }
}
```

---

## 🔑 `[ApiController]` — What It Does for You

```
Without [ApiController]          With [ApiController]
────────────────────────         ────────────────────────────────────
You manually check                Automatically returns 400 if
ModelState.IsValid                ModelState is invalid

You manually handle               [FromBody] is assumed on complex
[FromBody] vs [FromForm]          parameters automatically

You return raw errors             Errors returned in ProblemDetails
however you want                  standard format automatically
```

```csharp
// WITHOUT [ApiController] — you write this manually in every action:
if (!ModelState.IsValid)
    return BadRequest(ModelState);

// WITH [ApiController] — this happens automatically
// You don't write the check — the framework does it for you
```

---

## 🔑 HTTP Methods → Controller Actions → Status Codes

```
OPERATION    HTTP METHOD  URL              ACTION      RETURNS
──────────   ──────────── ────────────── ─────────── ─────────────────
Read all     GET          /api/employee  GetAll()    200 + array
Read one     GET          /api/employee/5 GetById(5) 200 + object / 404
Create new   POST         /api/employee  Create()    201 + saved object
Replace all  PUT          /api/employee/5 Update(5)  200 + updated object
Update part  PATCH        /api/employee/5 Patch(5)   200 + updated object
Delete       DELETE       /api/employee/5 Delete(5)  204 (no content)
```

---

## 🔑 Calling Web API from jQuery AJAX

```javascript
// ── GET all employees ─────────────────────────────────────────
$.get("/api/employee", function(employees) {
    renderTable(employees);
}).fail(function(xhr) {
    if (xhr.status === 404) showError("No employees found");
});


// ── GET one employee ──────────────────────────────────────────
$.get("/api/employee/5", function(emp) {
    $("#Name").val(emp.name);
    $("#Salary").val(emp.salary);
});


// ── POST — create new ─────────────────────────────────────────
$.ajax({
    url:         "/api/employee",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify({
                     name:       $("#Name").val(),
                     department: $("#Department").val(),
                     salary:     parseFloat($("#Salary").val())
                 }),
    success: function(saved) {
        // saved = the created employee with its new ID
        showNotification("Created! ID = " + saved.id);
        refreshGrid();
    },
    error: function(xhr) {
        if (xhr.status === 400) {
            // validation errors from [ApiController]
            displayErrors(xhr.responseJSON.errors);
        }
    }
});


// ── PUT — update existing ─────────────────────────────────────
var empId = 5;
$.ajax({
    url:         "/api/employee/" + empId,
    type:        "PUT",
    contentType: "application/json",
    data:        JSON.stringify({ id: empId, name: "Alice Updated", salary: 80000 }),
    success: function(updated) {
        showNotification("Updated successfully");
        refreshGrid();
    }
});


// ── DELETE ────────────────────────────────────────────────────
$.ajax({
    url:  "/api/employee/" + empId,
    type: "DELETE",
    success: function() {
        // 204 No Content — success function still fires
        $("tr[data-id='" + empId + "']").remove();
        showNotification("Deleted");
    },
    error: function(xhr) {
        if (xhr.status === 404) showError("Employee not found");
    }
});
```

---

## 🔑 Response Helpers — `Ok()`, `NotFound()`, `BadRequest()`

These are shortcut methods in `ControllerBase` that set the correct status code:

| Method                     | Status Code | Use When                              |
| -------------------------- | ----------- | ------------------------------------- |
| `Ok(data)`               | 200         | Success — return data                |
| `Ok()`                   | 200         | Success — no data to return          |
| `Created(url, data)`     | 201         | New resource created                  |
| `CreatedAtAction(...)`   | 201         | Created — with location header       |
| `NoContent()`            | 204         | Success — nothing to return (DELETE) |
| `BadRequest()`           | 400         | Client sent bad data                  |
| `BadRequest(ModelState)` | 400         | Validation failed                     |
| `Unauthorized()`         | 401         | Not logged in                         |
| `Forbid()`               | 403         | Logged in but no permission           |
| `NotFound()`             | 404         | Resource doesn't exist                |
| `StatusCode(500, msg)`   | 500         | Custom error                          |

---

## 🔑 Routing — How URLs Map to Actions

```csharp
[Route("api/[controller]")]   // api/employee (controller name without "Controller")
public class EmployeeApiController : ControllerBase
{
    [HttpGet]                  // GET  /api/employee
    [HttpGet("{id}")]          // GET  /api/employee/5
    [HttpGet("active")]        // GET  /api/employee/active
    [HttpGet("{id}/projects")] // GET  /api/employee/5/projects

    [HttpPost]                 // POST /api/employee
    [HttpPost("import")]       // POST /api/employee/import

    [HttpPut("{id}")]          // PUT  /api/employee/5
    [HttpDelete("{id}")]       // DELETE /api/employee/5
}
```

```csharp
// Route parameters become action parameters automatically
[HttpGet("{id}")]
public IActionResult GetById(int id)   // id comes from URL /api/employee/5
{...}

[HttpGet("{id}/projects")]
public IActionResult GetProjects(int id)   // id from URL
{...}

// Query string parameters
// GET /api/employee?dept=IT&page=2
[HttpGet]
public IActionResult GetAll(string dept, int page = 1)
{...}
```

---

## 🔑 MVC Controller `JsonResult` vs API `IActionResult`

You use both in your daily work. Key differences:

```csharp
// ── MVC CONTROLLER (for Kendo Grid, forms) ────────────────────
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    return Json(_db.Employees.ToDataSourceResult(request));
    // Always returns 200 — no status code control
    // Kendo expects this format
}


// ── WEB API CONTROLLER (for pure JSON endpoints) ─────────────
[HttpGet]
public IActionResult GetAll()
{
    return Ok(_db.Employees.ToList());
    // Returns 200 with JSON
    // Can also return NotFound(), BadRequest() etc.
    // More control over status codes
}
```

---

## ❓ Interview Questions

**Q: What is the difference between `Controller` and `ControllerBase`?**

> `Controller` inherits from `ControllerBase` and adds View support — methods like `View()`, `PartialView()`, `ViewBag`. `ControllerBase` is the base for API controllers — it has all the HTTP result helpers (`Ok()`, `NotFound()`, `BadRequest()`) but no View support. Use `ControllerBase` when the controller only returns JSON.

**Q: What does `[ApiController]` do?**

> It enables three automatic behaviours: automatic 400 responses when ModelState is invalid (no need to check manually), automatic `[FromBody]` binding for complex parameters, and standardised `ProblemDetails` error format.

**Q: What is the difference between `return Json(data)` and `return Ok(data)`?**

> `Json(data)` is from MVC's `Controller` class — always returns 200, used with Kendo's DataSource. `Ok(data)` is from `ControllerBase` — returns 200 with proper content negotiation. In Web API controllers, use `Ok()`, `NotFound()`, etc. for correct status codes.

**Q: What status code should a successful DELETE return?**

> 204 No Content — the operation succeeded but there's nothing to return. In code: `return NoContent()`.

**Q: What status code should a successful POST (create) return?**

> 201 Created — a new resource was created. Use `return CreatedAtAction(...)` which also sets the `Location` header pointing to the new resource's URL.
>
