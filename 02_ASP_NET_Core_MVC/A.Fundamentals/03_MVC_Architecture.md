
# 03 — MVC Architecture

---

## 🎯 One-Line Definition

> **MVC (Model-View-Controller) is a design pattern that separates your application into three parts: Model (data), View (UI), and Controller (logic) — each with one clear job, so they can change independently without breaking each other.**

---

## 🔷 The Problem MVC Solves

```
WITHOUT MVC (everything in one place — like old ASP.NET Web Forms):
──────────────────────────────────────────────────────────────
EmployeeList.aspx — contains:
  HTML structure
  CSS styling
  C# database queries
  Business validation
  Response formatting

Problems:
  ❌ Designer can't change HTML without risking C# logic
  ❌ Can't unit-test business logic — it's tangled with HTML
  ❌ Can't reuse the data query for an API — it's in the page
  ❌ Changing DB schema = rewriting the whole page

WITH MVC:
──────────────────────────────────────────────────────────────
Model       → only data + validation rules
View        → only HTML + Razor
Controller  → only receives request, calls model, returns view

✅ Designer edits View without touching C#
✅ Business logic in Controller/Model is unit-testable
✅ Same Controller can return View OR JSON
✅ One change in Model updates all views automatically
```

---

## 🔷 The Three Parts — What Each Does

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MODEL                                                          │
│  ─────                                                          │
│  Represents data and business rules                             │
│  e.g.: Employee.cs with Id, Name, Salary, [Required] etc.      │
│  Knows nothing about UI or HTTP                                 │
│                                                                 │
│  VIEW                                                           │
│  ─────                                                          │
│  HTML + Razor template — generates what the user sees          │
│  e.g.: Index.cshtml renders a table of employees               │
│  Knows nothing about where the data came from                   │
│                                                                 │
│  CONTROLLER                                                     │
│  ──────────                                                     │
│  Handles the HTTP request — the traffic director               │
│  Gets data from Model/services, passes to View                  │
│  e.g.: EmployeeController.Index() → gets list → returns view   │
│  Knows about both Model and View — bridges them                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 How a Request Flows Through MVC

```
Browser: GET /Employee/Index
         │
         ▼
ROUTING: matches → EmployeeController.Index()
         │
         ▼
CONTROLLER: EmployeeController.Index()
  → calls _bal.GetAll()
  → receives List<Employee>
  → return View(employees)
         │
         ▼
VIEW ENGINE: Index.cshtml
  → receives List<Employee> as @model
  → renders HTML table of employees
         │
         ▼
Browser receives HTML → renders the page

Browser: POST /Employee/Create  (form submitted)
         │
         ▼
ROUTING: matches → EmployeeController.Create(emp)
         │
         ▼
CONTROLLER: Create([HttpPost])
  → model binding fills Employee emp from form
  → ModelState.IsValid? → call _bal.Insert(emp)
  → success → RedirectToAction("Index")
  → fail    → return View(emp) with errors
```

---

## 🔷 MVC in Your Project — Concrete Example

### Model

```csharp
// Models/Employee.cs
public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    public string Name { get; set; }

    [Required]
    public string Role { get; set; }

    [Range(10000, 1000000, ErrorMessage = "Salary between ₹10,000–₹10,00,000")]
    public decimal Salary { get; set; }
}
// Model knows: what an Employee IS and what's valid
// Model does NOT know: how it's displayed or how it was requested
```

### Controller

```csharp
// Controllers/EmployeeController.cs
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    // GET /Employee  → Show list
    public IActionResult Index()
    {
        List<Employee> employees = _bal.GetAll();
        return View(employees);                   // passes model to view
    }

    // GET /Employee/Create  → Show empty form
    public IActionResult Create()
    {
        return View(new Employee());
    }

    // POST /Employee/Create  → Save new employee
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)    // model binding populates emp
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Insert(emp);
        if (result == "success") return RedirectToAction(nameof(Index));

        ModelState.AddModelError("", result);
        return View(emp);
    }
}
// Controller knows: HTTP verbs, routing, model state, which view to return
// Controller does NOT know: SQL queries, HTML rendering
```

### View

```cshtml
@* Views/Employee/Index.cshtml *@
@model List<Employee>

<h2>Employee List</h2>

<a asp-action="Create" class="btn btn-primary">+ Add New</a>

<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Role</th>
            <th>Salary</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var emp in Model)
        {
            <tr>
                <td>@emp.Name</td>
                <td>@emp.Role</td>
                <td>@emp.Salary.ToString("C0")</td>
                <td>
                    <a asp-action="Edit" asp-route-id="@emp.Id">Edit</a>
                </td>
            </tr>
        }
    </tbody>
</table>
@* View knows: how to display Employee data as HTML
   View does NOT know: where the data came from, any business rules *@
```

---

## 🔷 Separation of Concerns — The Key Principle

```
CONCERN             LIVES IN        NEVER IN
──────────────────────────────────────────────────────
Data + rules        Model           View, Controller
HTML display        View            Model, Controller
HTTP handling       Controller      Model, View
SQL queries         DAL             Controller, View
Business rules      BAL             Controller, View
```

---

## 🔷 What Each Part Knows About

```
MODEL knows about:
  ✅ Its own properties (Id, Name, Salary)
  ✅ Its own validation rules ([Required], [Range])
  ❌ Views — never
  ❌ Controllers — never
  ❌ HTTP — never
  ❌ Databases — never (in MVC Pattern; DAL handles that)

VIEW knows about:
  ✅ The model it receives (@model Employee)
  ✅ HTML rendering
  ✅ Razor syntax
  ❌ Where the model came from
  ❌ Any database or business logic

CONTROLLER knows about:
  ✅ HTTP methods (GET, POST)
  ✅ How to call services/BAL
  ✅ Which view to return
  ✅ ModelState
  ❌ SQL queries
  ❌ Raw HTML construction
```

---

## 🔷 MVC vs Other Patterns

|                  | MVC                  | MVVM                   | MVP               |
| ---------------- | -------------------- | ---------------------- | ----------------- |
| Used in          | Web (ASP.NET Core)   | Desktop (WPF), Blazor  | Android (old)     |
| View updates     | Controller → View   | Two-way binding        | Presenter → View |
| View knows about | Model (directly)     | ViewModel              | Interface only    |
| Best for         | Web request-response | Data binding heavy UIs | Desktop apps      |

---

## 🔷 MVC Folder Structure in Your Project

```
MyApp/
│
├── Controllers/               ← C files (EmployeeController.cs)
│   └── EmployeeController.cs
│
├── Models/                    ← M files (Employee.cs, Department.cs)
│   ├── Employee.cs
│   └── Department.cs
│
├── Views/                     ← V files (.cshtml templates)
│   ├── Employee/
│   │   ├── Index.cshtml       ← for EmployeeController.Index()
│   │   ├── Create.cshtml      ← for EmployeeController.Create()
│   │   └── Edit.cshtml
│   ├── Shared/
│   │   ├── _Layout.cshtml     ← master layout (header, footer)
│   │   └── _ValidationScripts.cshtml
│   └── _ViewImports.cshtml    ← global Razor imports
│
├── BAL/                       ← Business logic (your addition)
├── DAL/                       ← Data access (your addition)
└── appsettings.json
```

---

## 🔷 Convention Over Configuration

ASP.NET Core MVC uses conventions — follow the naming pattern and things work automatically:

```
EmployeeController.cs    → controller name
  ↓
Index() action method    → action name
  ↓
Views/Employee/Index.cshtml  ← view automatically found here

Rule: Views/{ControllerName}/{ActionName}.cshtml
  EmployeeController + Index  →  Views/Employee/Index.cshtml
  EmployeeController + Create →  Views/Employee/Create.cshtml
  HomeController     + About  →  Views/Home/About.cshtml
```

---

## ⭐ Interview Quick-Fire

| Question                                         | Answer                                                                 |
| ------------------------------------------------ | ---------------------------------------------------------------------- |
| What does MVC stand for?                         | Model, View, Controller                                                |
| What is the Model responsible for?               | Represents data and defines validation rules                           |
| What is the View responsible for?                | Renders HTML — the UI the user sees                                   |
| What is the Controller responsible for?          | Handles HTTP requests, calls services, returns views                   |
| What design principle does MVC follow?           | Separation of Concerns — each part has one job                        |
| Where does View find its template automatically? | `Views/{ControllerName}/{ActionName}.cshtml`                         |
| Can a Controller return both View and JSON?      | ✅ Yes —`return View()`or `return Json()`from the same controller |
| What passes data from Controller to View?        | `return View(model)`— model binding                                 |
