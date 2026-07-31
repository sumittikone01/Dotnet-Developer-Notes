
# 04 — Action Result Types

---

## 🎯 One-Line Definition

> **Action Result Types are the specific return values from controller action methods — each one tells ASP.NET Core exactly what HTTP response to build: what status code, what body (HTML, JSON, redirect, file), and what headers to set.**

---

## 🔷 The IActionResult Family Tree

```
IActionResult  (interface — the contract)
│
├── ActionResult  (base class)
│   │
│   ├── ViewResult              → renders a Razor view (HTML)
│   ├── PartialViewResult       → renders a partial view (HTML fragment)
│   ├── JsonResult              → returns JSON
│   ├── RedirectResult          → 302 redirect to URL
│   ├── RedirectToActionResult  → 302 redirect to controller/action
│   ├── ContentResult           → returns plain text / custom content
│   ├── FileResult              → returns a file download
│   │   ├── FileContentResult
│   │   ├── FileStreamResult
│   │   └── PhysicalFileResult
│   │
│   └── StatusCodeResult        → returns a status code
│       ├── OkResult            → 200
│       ├── OkObjectResult      → 200 + JSON body
│       ├── CreatedResult       → 201
│       ├── NoContentResult     → 204
│       ├── BadRequestResult    → 400
│       ├── UnauthorizedResult  → 401
│       ├── ForbidResult        → 403
│       └── NotFoundResult      → 404
```

---

## 🔷 Group 1 — View Results (MVC Controller)

These are only available in controllers inheriting from `Controller` (not `ControllerBase`):

```csharp
public class EmployeeController : Controller
{
    // ── View() — render a Razor template ───────────────────────────

    public IActionResult Index()
        => View();
    // Finds: Views/Employee/Index.cshtml
    // No model passed → @Model is null in view

    public IActionResult Index()
        => View(employees);
    // Finds: Views/Employee/Index.cshtml
    // employees passed → @Model is List<Employee>

    public IActionResult ShowReport()
        => View("CustomReport", reportData);
    // Finds: Views/Employee/CustomReport.cshtml
    // Explicit view name — overrides convention

    public IActionResult ShowReport()
        => View("~/Views/Shared/Report.cshtml", reportData);
    // Explicit full path — ignores all convention


    // ── PartialView() — HTML fragment for AJAX updates ──────────────

    public IActionResult GetRow(int id)
    {
        var emp = _bal.GetById(id);
        return PartialView("_EmployeeRow", emp);
    }
    // Renders Views/Employee/_EmployeeRow.cshtml (or Views/Shared/)
    // Returns only the fragment — no layout, no full page
    // Used when AJAX calls need to update PART of a page
}
```

```
View Search Order:
──────────────────────────────────────────────────────────────
1. Views/{ControllerName}/{ActionName}.cshtml
2. Views/Shared/{ActionName}.cshtml
3. Error thrown: "The view '{name}' was not found"
```

---

## 🔷 Group 2 — Redirect Results (MVC Controller)

```csharp
public class EmployeeController : Controller
{
    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        _bal.AddEmployee(emp);

        // ── RedirectToAction — redirect within your app ─────────────

        return RedirectToAction("Index");
        // → GET /Employee/Index   (same controller)

        return RedirectToAction("Index", "Home");
        // → GET /Home/Index   (different controller)

        return RedirectToAction("Details", new { id = newId });
        // → GET /Employee/Details/5   (with route values)

        return RedirectToAction("Index", "Home", new { area = "Admin" });
        // → GET /Admin/Home/Index   (with area)


        // ── RedirectToRoute — redirect by route name ────────────────

        return RedirectToRoute("default", new { controller = "Home", action = "Index" });


        // ── Redirect — redirect to any URL ──────────────────────────

        return Redirect("/Employee/Index");
        // Hardcoded URL — works but RedirectToAction is safer

        return Redirect("https://google.com");
        // External URL redirect


        // ── RedirectPermanent — 301 permanent redirect ──────────────

        return RedirectPermanent("/new-url");
        // 301 — browsers cache this, search engines update index
        // Use for: old URLs that have moved permanently
    }
}
```

```
302 vs 301:
──────────────────────────────────────────────────────────────
302 Found (temporary)  → browser won't cache it
                         use for: PRG pattern, login redirects

301 Moved Permanently  → browser caches it, SEO transfers
                         use for: renamed URLs, old pages moved
```

---

## 🔷 Group 3 — Status Code Results (API Controller / JSON)

These are the ones you use in API controllers that return JSON to Kendo and AJAX:

```csharp
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
{
    // ── 2xx SUCCESS ─────────────────────────────────────────────────

    // 200 OK — request succeeded, returning data
    return Ok(employees);
    // HTTP: 200 OK
    // Body: [ {"empId":1,"empName":"John",...}, ... ]

    return Ok(new { data = list, total = count });
    // Body: { "data": [...], "total": 47 }
    // ↑ Kendo DataSource expects this exact shape

    return Ok();
    // 200 with no body — rarely used, prefer NoContent() for empty success

    // 201 Created — new resource was created
    return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    // HTTP: 201 Created
    // Header: Location: /api/employee/5
    // Body: the created employee object with its new ID
    //
    // nameof(GetById)    = name of the GET action that fetches this resource
    // new { id = newId } = route values to build the Location URL
    // emp                = the created object to return in body

    return Created("/api/employee/" + newId, emp);
    // Same as above but hardcoded URL — less ideal

    // 204 No Content — success, nothing to return
    return NoContent();
    // HTTP: 204 No Content
    // Body: empty
    // Use for: PUT (update) and DELETE
    // Kendo DataSource expects 204 for successful update/delete

    // ── 4xx CLIENT ERRORS ──────────────────────────────────────────

    // 400 Bad Request — client sent invalid data
    return BadRequest();
    // No body — rarely useful

    return BadRequest("Name is required");
    // Body: "Name is required"  (plain string)

    return BadRequest(new { message = "Name is required", field = "EmpName" });
    // Body: { "message": "...", "field": "..." }
    // ↑ AJAX error handler can read xhr.responseJSON.message

    return BadRequest(ModelState);
    // Body: ProblemDetails with all validation errors:
    // {
    //   "errors": {
    //     "EmpName": ["Name is required"],
    //     "Salary": ["Must be between 0 and 999999"]
    //   }
    // }

    // 401 Unauthorized — not authenticated
    return Unauthorized();
    return Unauthorized(new { message = "Please log in" });

    // 403 Forbidden — authenticated but not authorized
    return Forbid();
    // Note: Forbid() — no parentheses on the Forbid result in Razor

    // 404 Not Found — resource doesn't exist
    return NotFound();
    return NotFound(new { message = $"Employee {id} not found" });

    // 409 Conflict — duplicate or constraint violation
    return Conflict();
    return Conflict(new { message = "Employee with this email already exists" });

    // ── 5xx SERVER ERRORS ──────────────────────────────────────────

    // 500 Internal Server Error
    return StatusCode(500, "An internal error occurred");
    return StatusCode(500, new { message = "Database error", error = ex.Message });

    // Any custom status code:
    return StatusCode(422, new { message = "Business rule violated" });
    // 422 Unprocessable Entity — valid format, but fails business rules
}
```

---

## 🔷 Group 4 — Content and File Results

```csharp
public class ReportController : Controller
{
    // ── ContentResult — raw text/HTML/XML ───────────────────────────

    public IActionResult GetPlainText()
        => Content("Hello World");
    // Content-Type: text/plain

    public IActionResult GetHtml()
        => Content("<b>Hello</b>", "text/html");

    public IActionResult GetXml()
        => Content("<employee><name>John</name></employee>", "application/xml");


    // ── JsonResult — explicit JSON (rarely needed — prefer Ok()) ────

    public IActionResult GetData()
        => Json(new { name = "John", dept = "HR" });
    // Content-Type: application/json
    // Note: In [ApiController], return Ok(data) is preferred over Json()


    // ── File Results — return files for download ────────────────────

    // Return file from byte array (generated in memory):
    public IActionResult DownloadReport()
    {
        byte[] fileBytes = GenerateExcelReport();
        return File(fileBytes,
                    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                    "EmployeeReport.xlsx");
    }

    // Return file from disk (physical file path):
    public IActionResult DownloadTemplate()
        => PhysicalFile("/var/app/templates/import.xlsx",
                        "application/vnd.ms-excel",
                        "ImportTemplate.xlsx");

    // Return file from wwwroot (virtual path):
    public IActionResult DownloadLogo()
        => File("~/images/logo.png", "image/png");

    // Return file as stream (large files — no memory buffering):
    public IActionResult DownloadLargeFile()
    {
        var stream = new FileStream("/path/to/large.zip", FileMode.Open);
        return new FileStreamResult(stream, "application/zip")
        {
            FileDownloadName = "backup.zip"
        };
    }
}
```

---

## 🔷 Choosing the Right Return Type — Decision Chart

```
What does your action return?
│
├── HTML page for browser?
│   └── return View(model)
│
├── HTML fragment for AJAX partial update?
│   └── return PartialView("_PartialName", model)
│
├── After successful POST?
│   └── return RedirectToAction("Index")   ← PRG pattern
│
├── JSON for AJAX / Kendo Grid?
│   ├── Success with data        → return Ok(data)
│   ├── Created new resource     → return CreatedAtAction(...)
│   ├── Updated / deleted        → return NoContent()
│   ├── Not found                → return NotFound(new { message })
│   ├── Validation error         → return BadRequest(ModelState)
│   └── Server error             → return StatusCode(500, new { message })
│
├── File download?
│   └── return File(bytes, contentType, fileName)
│
└── Redirect?
    ├── After form submit        → return RedirectToAction("Index")
    └── External URL             → return Redirect("https://...")
```

---

## 🔷 Real Patterns in Your Stack

```csharp
// PATTERN 1: Kendo Grid — Read (paged)
[HttpGet]
public IActionResult GetAll(int skip = 0, int take = 10)
{
    var data  = _bal.GetPaged(skip, take, out int total);
    return Ok(new { data, total });    // ← Kendo DataSource requires this shape
}

// PATTERN 2: Kendo Grid — Create
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
{
    int newId = _bal.Add(emp);
    emp.EmpId = newId;
    return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    // ← Returns 201 + Location header + the new employee with its DB-assigned ID
}

// PATTERN 3: Kendo Grid — Update
[HttpPut("{id:int}")]
public IActionResult Update(int id, [FromBody] Employee emp)
{
    if (id != emp.EmpId)
        return BadRequest(new { message = "ID mismatch" });

    bool updated = _bal.Update(emp);
    if (!updated)
        return NotFound(new { message = $"Employee {id} not found" });

    return NoContent();   // ← 204: Kendo expects this for successful PUT
}

// PATTERN 4: Kendo Grid — Delete
[HttpDelete("{id:int}")]
public IActionResult Delete(int id)
{
    bool deleted = _bal.Delete(id);
    if (!deleted)
        return NotFound(new { message = $"Employee {id} not found" });

    return NoContent();   // ← 204: Kendo expects this for successful DELETE
}

// PATTERN 5: AJAX — search autocomplete
[HttpGet("search")]
public IActionResult Search(string term)
{
    if (string.IsNullOrEmpty(term))
        return Ok(new List<object>());

    var results = _bal.Search(term);
    return Ok(results);   // ← returns matching items as JSON array
}
```

---

## 🔷 What Kendo DataSource Expects — HTTP Response Map

```
Kendo Operation  →  HTTP Method  →  Expected Response
──────────────────────────────────────────────────────────────────
Read (load grid) →  GET          →  200 OK + { data: [...], total: N }
Create (add row) →  POST         →  201 Created + the new object with ID
Update (edit row)→  PUT          →  204 No Content  (empty body)
Delete           →  DELETE       →  204 No Content  (empty body)

Validation fail  →  POST/PUT     →  400 Bad Request + { errors: {...} }
Not found        →  GET/PUT/DEL  →  404 Not Found + { message: "..." }
Server crash     →  any          →  500 Internal Server Error + { message }
```

---

## ⭐ Interview Quick-Fire

| Question                                                             | Answer                                                                                                                |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| What is `IActionResult`?                                           | Interface representing any possible HTTP response —`Ok()`,`NotFound()`,`View()`all implement it                |
| What does `return Ok(data)`return?                                 | HTTP 200 with `data`serialized as JSON in the body                                                                  |
| What does `return NoContent()`return?                              | HTTP 204 — success with an empty body — used for PUT and DELETE                                                     |
| What does `return CreatedAtAction(...)`return?                     | HTTP 201 with a `Location`header pointing to the new resource and the created object in the body                    |
| What does `return View(model)`return?                              | Renders the matching Razor `.cshtml`file with `model`as `@Model`and returns HTML                                |
| What does `return RedirectToAction("Index")`return?                | HTTP 302 — browser redirects to that action's URL                                                                    |
| Why use `RedirectToAction`after a POST?                            | PRG pattern — prevents duplicate form submission on browser refresh                                                  |
| What does `return BadRequest(ModelState)`return?                   | HTTP 400 + JSON object with all validation errors — AJAX can read `xhr.responseJSON.errors`                        |
| What HTTP code should a successful DELETE return?                    | 204 No Content — success with no body                                                                                |
| What is the difference between `Redirect`and `RedirectToAction`? | `Redirect`takes a hardcoded URL string.`RedirectToAction`takes controller/action names — safer, respects routing |
