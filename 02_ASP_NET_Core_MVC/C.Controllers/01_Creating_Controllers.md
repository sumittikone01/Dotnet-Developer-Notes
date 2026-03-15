
# 01 — Creating Controllers

---

## 🎯 One-Line Definition

> **A Controller is a C# class that handles HTTP requests — it receives the request, calls the BAL/service for data, and returns a response (View, JSON, redirect) — one Controller per feature area of your app.**

---

## 🔷 What a Controller Is

```
Browser sends: GET /Employee/Index
                    ↓
ASP.NET Core routing reads: "Employee" + "Index"
                    ↓
Finds:  EmployeeController  →  Index() method
                    ↓
Index() runs → gets data → returns View(data)
                    ↓
Browser receives HTML page

The Controller is the traffic director — it receives
requests and decides what to do with them.
```

---

## 🔷 Rules for Creating a Controller

```
RULE 1: Class name must end with "Controller"
  EmployeeController ✅
  Employee           ❌ — not found by routing

RULE 2: Must inherit from Controller (or ControllerBase for APIs)
  public class EmployeeController : Controller

RULE 3: Must be in the Controllers/ folder (by convention)
  OR have [Controller] attribute

RULE 4: Must be public
  public class EmployeeController ✅
  class EmployeeController        ❌ — not routed

RULE 5: Constructor injection for dependencies
  public EmployeeController(EmployeeBAL bal) { _bal = bal; }
```

---

## 🔷 Minimum Working Controller

```csharp
using Microsoft.AspNetCore.Mvc;

public class EmployeeController : Controller
{
    // GET /Employee    OR    GET /Employee/Index
    public IActionResult Index()
    {
        return View();
    }
}
```

---

## 🔷 Complete Controller — With DI and All CRUD Actions

```csharp
using Microsoft.AspNetCore.Mvc;
using EmployeeApp.Models;
using EmployeeApp.BAL;

public class EmployeeController : Controller
{
    // ── Dependency Injection ──────────────────────────────────────
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal)
    {
        _bal = bal;
        // ASP.NET Core injects EmployeeBAL automatically
        // because it's registered in Program.cs
    }

    // ── INDEX: GET /Employee ──────────────────────────────────────
    public IActionResult Index()
    {
        var employees = _bal.GetAll();
        return View(employees);
    }

    // ── DETAILS: GET /Employee/Details/5 ─────────────────────────
    public IActionResult Details(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // ── CREATE: GET /Employee/Create ──────────────────────────────
    public IActionResult Create()
    {
        return View(new Employee());
    }

    // ── CREATE: POST /Employee/Create ─────────────────────────────
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Insert(emp);

        if (result == "success")
        {
            TempData["Success"] = "Employee added successfully!";
            return RedirectToAction(nameof(Index));
        }

        ModelState.AddModelError("", result);
        return View(emp);
    }

    // ── EDIT: GET /Employee/Edit/5 ────────────────────────────────
    public IActionResult Edit(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // ── EDIT: POST /Employee/Edit/5 ───────────────────────────────
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Edit(int id, Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Update(emp);

        if (result == "success")
        {
            TempData["Success"] = "Employee updated successfully!";
            return RedirectToAction(nameof(Index));
        }

        ModelState.AddModelError("", result);
        return View(emp);
    }

    // ── DELETE: POST /Employee/Delete/5 ───────────────────────────
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Delete(int id)
    {
        string result = _bal.Delete(id);

        if (result == "success")
            TempData["Success"] = "Employee deleted.";
        else
            TempData["Error"] = result;

        return RedirectToAction(nameof(Index));
    }
}
```

---

## 🔷 Controller with Web API Actions (JSON responses)

```csharp
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    // Returns View for browser requests
    public IActionResult Index()
        => View(_bal.GetAll());

    // Returns JSON for AJAX / Kendo Grid requests
    [HttpPost]
    public IActionResult ReadJson()
        => Json(_bal.GetAll());

    // Returns JSON for a specific employee
    [HttpGet]
    public IActionResult GetJson(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null)
            return Json(new { success = false, message = "Not found" });

        return Json(new { success = true, data = emp });
    }
}
```

---

## 🔷 Controller Attributes — What You Put on the Class

```csharp
[Authorize]                          // ← require login for ALL actions
[Authorize(Roles = "Admin,Manager")] // ← require specific role
[Route("api/employees")]             // ← custom route prefix
[ResponseCache(Duration = 60)]       // ← cache all responses 60s
public class EmployeeController : Controller
{
    // All actions in this controller inherit the class-level attributes
}
```

---

## 🔷 Registering Controller Dependencies in Program.cs

```csharp
// Program.cs — register all services the Controller needs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();      // enables MVC controllers
builder.Services.AddScoped<EmployeeDAL>();        // DAL
builder.Services.AddScoped<EmployeeBAL>();        // BAL

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

app.Run();
```

---

## 🔷 Controller Naming Convention

```
File name:   EmployeeController.cs
Class name:  EmployeeController
URL prefix:  /Employee/...

File name:   DepartmentController.cs
Class name:  DepartmentController
URL prefix:  /Department/...

File name:   HomeController.cs
Class name:  HomeController
URL prefix:  /Home/...   (default route → also serves /)

The "Controller" suffix is stripped from the URL automatically.
```

---

## 🔷 Properties Available in Every Controller

```csharp
public class EmployeeController : Controller
{
    public IActionResult Demo()
    {
        // Available from the Controller base class:

        // Request info
        var url    = Request.Path;              // /Employee/Index
        var method = Request.Method;            // GET / POST
        var query  = Request.Query["search"];   // ?search=alice

        // Routing info
        var ctrl   = RouteData.Values["controller"];  // Employee
        var action = RouteData.Values["action"];      // Index

        // Pass data to View
        ViewBag.Title   = "Employee List";
        ViewData["Msg"] = "Hello from controller";

        // Pass data to NEXT request (survives redirect)
        TempData["Success"] = "Saved!";

        // User info (if authenticated)
        bool loggedIn = User.Identity.IsAuthenticated;
        string name   = User.Identity.Name;

        return View();
    }
}
```

---

## ⚠️ Common Controller Mistakes

| Mistake                                | What Happens                         | Fix                                          |
| -------------------------------------- | ------------------------------------ | -------------------------------------------- |
| Class doesn't inherit `Controller`   | Routing ignores it                   | Always `: Controller`                      |
| Class name missing "Controller" suffix | Not found by routing                 | `EmployeeController`not `Employee`       |
| SQL code in Controller                 | Layer boundary broken                | SQL only in DAL                              |
| Business rules in Controller           | Untestable, hard to maintain         | Rules only in BAL                            |
| `[HttpPost]`missing on POST action   | GET and POST both match — ambiguous | Always mark POST actions with `[HttpPost]` |

---

## ⭐ Interview Quick-Fire

| Question                                            | Answer                                                                              |
| --------------------------------------------------- | ----------------------------------------------------------------------------------- |
| What must a Controller class inherit?               | `Controller`(for MVC) or `ControllerBase`(for Web API)                          |
| What suffix must a Controller class name end with?  | `Controller`— e.g.`EmployeeController`                                         |
| How does ASP.NET Core find the right Controller?    | Routing strips "Controller" suffix and matches URL segments                         |
| How do you inject a service into a Controller?      | Constructor injection — parameter in the constructor, registered in `Program.cs` |
| What does `[HttpPost]`do?                         | Restricts the action to only handle HTTP POST requests                              |
| What does `[ValidateAntiForgeryToken]`do?         | Validates the anti-forgery token sent from the form — prevents CSRF attacks        |
| Can one Controller have both View and JSON actions? | ✅ Yes —`return View()`and `return Json()`can coexist                          |
