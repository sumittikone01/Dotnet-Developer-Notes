
# 04 — MVC Request Flow

---

## 🎯 One-Line Definition

> **The MVC request flow is the complete journey of an HTTP request — from the browser hitting a URL, through routing → controller → model → view, all the way back to the browser rendering HTML — every step follows a predictable, traceable path.**

---

## 🔷 The Full MVC Request Flow — Master Diagram

```
BROWSER (or AJAX / Kendo)
     │
     │  1. HTTP Request: GET /Employee/Index
     ▼
┌─────────────────────────────────────────────────────────────────┐
│  KESTREL (Web Server)                                           │
│  Accepts TCP connection, reads raw HTTP bytes                   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  2. Passed to ASP.NET Core
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  MIDDLEWARE PIPELINE                                            │
│                                                                 │
│  UseExceptionHandler  → wraps everything in try-catch          │
│  UseHttpsRedirection  → redirect HTTP to HTTPS if needed       │
│  UseStaticFiles       → is this a CSS/JS file? serve & stop    │
│  UseRouting           → 3. figure out which controller/action  │
│  UseAuthentication    → 4. read JWT/cookie, identify user      │
│  UseAuthorization     → 5. is user allowed? else 401/403       │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  6. Route matched: EmployeeController.Index
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  CONTROLLER (EmployeeController.Index)                          │
│                                                                 │
│  7. Action method called                                        │
│  8. Model Binding runs (extract id, form data, JSON)           │
│  9. [Authorize] / Filters execute (if any)                     │
│  10. Controller calls BAL                                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  11. BAL call: _bal.GetActiveEmployees()
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  BAL (EmployeeBAL)                                              │
│                                                                 │
│  12. Business rules applied                                     │
│  13. Calls DAL                                                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  14. DAL call: _dal.GetAll()
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  DAL (EmployeeDAL)                                              │
│                                                                 │
│  15. SqlConnection opened                                       │
│  16. SqlCommand executed                                        │
│  17. SqlDataReader reads rows                                   │
│  18. List<Employee> built and returned                          │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  Data travels back up: DAL → BAL → Controller
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  CONTROLLER — return View(employees)                            │
│                                                                 │
│  19. Controller calls return View(data)                        │
│  20. Razor Engine finds Views/Employee/Index.cshtml            │
│  21. Razor renders HTML: merges template + data               │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  22. HTML response
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  MIDDLEWARE PIPELINE (response travels back out)                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
                    BROWSER renders HTML page
```

---

## 🔷 Flow 1 — Standard MVC Page Request

Step-by-step trace for: `GET /Employee/Index`

```
STEP 1: Browser sends
────────────────────────────────────────────────────────────────────
GET /Employee/Index HTTP/1.1
Host: localhost:7001
Cookie: .AspNetCore.Session=abc123   ← session cookie (if logged in)


STEP 2: UseRouting parses the URL
────────────────────────────────────────────────────────────────────
Pattern:    {controller=Home}/{action=Index}/{id?}
URL:        /Employee/Index
Parsed as:  controller = "Employee"
            action     = "Index"
            id         = null (not in URL)


STEP 3: EmployeeController.Index() is called
────────────────────────────────────────────────────────────────────
public IActionResult Index()
{
    var employees = _bal.GetActiveEmployees();
    return View(employees);
}


STEP 4: BAL → DAL → SQL Server → data returns
────────────────────────────────────────────────────────────────────
BAL.GetActiveEmployees()
  └─ DAL.GetAll()
       └─ SqlConnection.Open()
            └─ SqlCommand("SELECT * FROM Employees WHERE IsActive = 1")
                 └─ ExecuteReader()
                      └─ while(reader.Read()) → build List<Employee>
  ← List<Employee> returned
  ← BAL filters/sorts if needed
← Controller has List<Employee>


STEP 5: return View(employees)
────────────────────────────────────────────────────────────────────
Controller says "return View(employees)"
ASP.NET Core looks for:  Views/Employee/Index.cshtml
Razor engine renders it:
  @model List<Employee>        ← type set to List<Employee>
  @foreach(var e in Model)     ← Model = the employees list
  {
      <tr><td>@e.EmpName</td><td>@e.Department</td></tr>
  }
Outputs complete HTML string


STEP 6: Response sent
────────────────────────────────────────────────────────────────────
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 12453

<!DOCTYPE html>
<html>
  <head>...</head>
  <body>
    <table>
      <tr><td>John Smith</td><td>HR</td></tr>
      ...
    </table>
  </body>
</html>
```

---

## 🔷 Flow 2 — POST → Redirect → GET (PRG Pattern)

The most important MVC pattern — prevents duplicate form submissions:

```
WHY PRG EXISTS:
────────────────────────────────────────────────────────────────────
Without PRG:
  User fills form → POST /Employee/Create → success → stays on POST URL
  User hits F5 (refresh) → browser re-submits POST → duplicate record!

With PRG (Post-Redirect-Get):
  User fills form → POST /Employee/Create → success → REDIRECT to GET
  User hits F5 → browser just re-requests the GET → no duplicate
```

```
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: User fills form and clicks Submit                      │
│  Browser sends: POST /Employee/Create                           │
│  Body: EmpName=John&Department=HR&Salary=50000                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  CONTROLLER: [HttpPost] Create(Employee emp)                    │
│                                                                 │
│  ModelState.IsValid? → YES                                      │
│  _bal.AddEmployee(emp) → inserts into DB                        │
│  TempData["Success"] = "Employee created!"                      │
│  return RedirectToAction("Index")   ← 302 REDIRECT             │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │  HTTP 302 Found
                            │  Location: /Employee/Index
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  BROWSER automatically follows redirect                         │
│  Browser sends: GET /Employee/Index                             │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  CONTROLLER: [HttpGet] Index()                                  │
│  @TempData["Success"] shown in view                             │
│  return View(employees)                                         │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
                    Browser shows employee list
                    with "Employee created!" message
```

```csharp
// The code:
[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
        return View(emp);   // stay on form, show errors

    _bal.AddEmployee(emp);

    TempData["Success"] = "Employee created successfully!";
    return RedirectToAction("Index");   // ← the REDIRECT in PRG
}

[HttpGet]
public IActionResult Index()
{
    var employees = _bal.GetActiveEmployees();
    return View(employees);   // ← the GET in PRG
}
```

---

## 🔷 Flow 3 — Kendo Grid AJAX Request

When Kendo Grid loads data (no page reload):

```
STEP 1: MVC page loads first
────────────────────────────────────────────────────────────────────
Browser: GET /Employee/Index
  → EmployeeController.Index() returns View
  → HTML page with empty Kendo Grid sent to browser
  → Browser renders the page


STEP 2: Kendo DataSource auto-fires AJAX request
────────────────────────────────────────────────────────────────────
Kendo Grid JavaScript runs after page loads:
  GET /api/employee?skip=0&take=10
  Headers: { Accept: application/json }


STEP 3: API Controller handles it
────────────────────────────────────────────────────────────────────
UseRouting: matches EmployeeApiController.GetAll(skip=0, take=10)

[HttpGet] GetAll(int skip = 0, int take = 10)
{
    int total = 0;
    var data = _bal.GetPaged(skip, take, out total);
    return Ok(new { data, total });
}


STEP 4: JSON serialized and returned
────────────────────────────────────────────────────────────────────
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "empId": 1, "empName": "John Smith", "department": "HR" },
    { "empId": 2, "empName": "Jane Doe",   "department": "IT" }
  ],
  "total": 47
}


STEP 5: Kendo DataSource receives JSON
────────────────────────────────────────────────────────────────────
Kendo Grid reads schema.data = "data", schema.total = "total"
Grid renders 10 rows in the table
Pagination controls show "1-10 of 47"

NO PAGE RELOAD happened.
```

---

## 🔷 Flow 4 — Validation Failure Flow

When a form is submitted with invalid data:

```
User submits form: POST /Employee/Create
Body: EmpName=  (empty) &Salary=-100
                ↓
┌─────────────────────────────────────────────────────────────────┐
│  Model Binding creates Employee object:                         │
│    emp.EmpName = ""  (empty)                                    │
│    emp.Salary  = -100                                           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  Data Annotations checked:                                      │
│  [Required] EmpName    → FAILS  (empty)                         │
│  [Range(0,999999)] Salary → FAILS (-100)                        │
│                                                                 │
│  ModelState.IsValid = FALSE                                     │
│  ModelState.Errors populated with messages                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  Controller:                                                    │
│  if (!ModelState.IsValid)                                       │
│      return View(emp);   ← re-renders form WITH errors          │
│                                                                 │
│  BAL and DAL are NEVER called when model is invalid            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  View: Create.cshtml                                            │
│                                                                 │
│  <span asp-validation-for="EmpName">Name is required</span>    │
│  <span asp-validation-for="Salary">Salary must be ≥ 0</span>   │
│                                                                 │
│  Input fields retain the submitted values                       │
│  User sees errors inline — does not lose their input           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Flow 5 — Authorization Failure Flow

```
User (not logged in) tries to access a protected page:
GET /Employee/Create

UseAuthentication reads request → no cookie/JWT → User = Anonymous
                │
UseAuthorization checks [Authorize] on EmployeeController
                │
                ├─ Logged in user?       → Controller runs
                │
                └─ Not logged in?        → 302 redirect to login
                   Or [Authorize(Roles="Admin")] but user is not Admin?
                                         → 403 Forbidden

// Setup in Program.cs (cookie auth example):
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";   // redirect unauthenticated here
        options.AccessDeniedPath = "/Account/AccessDenied";
    });

// On controller:
[Authorize]                          // any logged-in user
[Authorize(Roles = "Admin")]         // Admin role only
[Authorize(Roles = "Admin,Manager")] // Admin OR Manager
[AllowAnonymous]                     // override — anyone can access
```

---

## 🔷 Two Parallel Flows — MVC vs API Side by Side

```
┌──────────────────────────────────┬──────────────────────────────────┐
│  MVC FLOW (returns HTML)         │  API FLOW (returns JSON)          │
├──────────────────────────────────┼──────────────────────────────────┤
│  Browser GET /Employee/Index     │  AJAX GET /api/employee           │
│           │                      │            │                      │
│  UseRouting → EmployeeController │  UseRouting → EmployeeApiController│
│  .Index()                        │  .GetAll()                        │
│           │                      │            │                      │
│  _bal.GetActiveEmployees()       │  _bal.GetPaged(skip, take, out n) │
│           │                      │            │                      │
│  return View(list)               │  return Ok(new {data, total})     │
│           │                      │            │                      │
│  Razor renders Index.cshtml      │  JSON serialized                  │
│           │                      │            │                      │
│  200 OK + HTML page              │  200 OK + JSON                    │
│           │                      │            │                      │
│  Browser renders page            │  Kendo Grid populates rows        │
└──────────────────────────────────┴──────────────────────────────────┘

Both live in the SAME ASP.NET Core application.
Both share the same BAL and DAL.
Only the controller type and return type differ.
```

---

## 🔷 View Resolution — How Razor Finds the Right File

```csharp
// In EmployeeController:
return View();
// → looks for: Views/Employee/Index.cshtml
//   (folder = controller name, file = action name)

return View("Edit");
// → looks for: Views/Employee/Edit.cshtml

return View("Edit", model);
// → looks for: Views/Employee/Edit.cshtml, passes model

return View("~/Views/Shared/SpecialView.cshtml", model);
// → explicit path — bypasses convention

return PartialView("_EmployeeRow", emp);
// → looks for: Views/Employee/_EmployeeRow.cshtml
//   or: Views/Shared/_EmployeeRow.cshtml
```

```
View Search Order (when you just say return View()):
──────────────────────────────────────────────────────────────
1. Views/{ControllerName}/{ActionName}.cshtml
2. Views/Shared/{ActionName}.cshtml
3. Not found → exception: view not found
```

---

## 🔷 Complete Request Summary — One Diagram

```
URL → Routing → Auth → Controller → BAL → DAL → SQL → Data back → View/JSON → Browser

                          ┌──────────────────────────────────────────┐
     [Middleware Chain]   │  [Your Code]                             │
                          │                                          │
 Request ─────────────────┼──────────────────────────────────────►  │
  GET /Employee/Index      │  Routing matches EmployeeController.Index│
                          │  ModelBinding extracts parameters        │
                          │  Filters / Authorization run            │
                          │  Controller calls BAL                   │
                          │  BAL calls DAL                          │
                          │  DAL executes SQL                       │
                          │  Data returned: List<Employee>          │
                          │  Controller: return View(data)          │
                          │  Razor renders HTML                     │
 Response ◄───────────────┼──────────────────────────────────────── │
  200 OK + HTML            │                                          │
                          └──────────────────────────────────────────┘
```

---

## ⭐ Interview Quick-Fire

| Question                                                         | Answer                                                                                                                                |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| What is the MVC request flow?                                    | URL → Routing → Middleware (auth) → Controller → BAL → DAL → SQL → Data back → View or JSON                                   |
| What is the PRG pattern?                                         | Post-Redirect-Get — after a successful POST, redirect to a GET to prevent form re-submission on refresh                              |
| Why does routing run before authentication?                      | Routing identifies the endpoint; authorization needs to know which endpoint to check its policies                                     |
| What happens if `ModelState.IsValid`is false?                  | Controller returns the View with model — BAL and DAL are never called                                                                |
| How does Razor find the right view file?                         | Convention:`Views/{ControllerName}/{ActionName}.cshtml`                                                                             |
| What is the role of `TempData`in PRG?                          | Store a success/error message that survives ONE redirect — displayed on the next GET page                                            |
| In your stack, does the MVC page and Kendo API use the same BAL? | ✅ Yes —`EmployeeController`and `EmployeeApiController`both inject and call the same `EmployeeBAL`                             |
| What does `return View(data)`tell ASP.NET Core?                | Find the Razor template for this action, render it with `data`as the `@Model`, return HTML                                        |
| What does `return Ok(data)`tell ASP.NET Core?                  | Serialize `data`to JSON, return HTTP 200 with JSON body                                                                             |
| When does `UseAuthorization`short-circuit the pipeline?        | When the user is not authenticated (for `[Authorize]`) or lacks the required role — returns 401/403 without calling the controller |
