# 01 — Controller / BAL / DAL Overview

---

## 🎯 One-Line Definition

> **Three-Layer Architecture splits your app into Controller (handles HTTP), BAL (handles business rules), and DAL (handles database) — each layer has one job and never does another layer's job.**

---

## 🔷 Why Three Layers?

```
WITHOUT layers (everything in one place):
──────────────────────────────────────────────────────────────
EmployeeController.cs — 800 lines
  ├── Handles HTTP request
  ├── Validates input
  ├── Checks if email is unique (business rule)
  ├── Calculates tax (business logic)
  ├── Opens SqlConnection
  ├── Executes INSERT SQL
  ├── Closes connection
  └── Returns JSON

Problems:
  ❌ Hard to read — all concerns mixed together
  ❌ Hard to test — can't test business logic without HTTP
  ❌ Hard to change — change DB = rewrite controller
  ❌ Hard to reuse — same logic copied in 5 controllers

WITH three layers:
──────────────────────────────────────────────────────────────
EmployeeController.cs   → "I received a request. Let me ask the BAL."
EmployeeBAL.cs          → "I'll validate and apply business rules. Then ask the DAL."
EmployeeDAL.cs          → "I'll talk to the database and return the data."

Benefits:
  ✅ Each file has one clear responsibility
  ✅ Easy to test each layer independently
  ✅ Change DAL (e.g. SQL Server → Oracle) = doesn't touch Controller
  ✅ Change business rules = doesn't touch Controller or DAL
```

---

## 🔷 The Three Layers — What Each One Does

```
┌─────────────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER                                             │
│  Controller (ASP.NET Core MVC)                                  │
│                                                                 │
│  ✅ Receives HTTP requests                                      │
│  ✅ Reads form data / route values / query strings             │
│  ✅ Calls BAL methods                                          │
│  ✅ Returns View / JSON / IActionResult                        │
│  ❌ No SQL, no business rules, no direct DB access             │
├─────────────────────────────────────────────────────────────────┤
│  BUSINESS LOGIC LAYER (BAL)                                     │
│  Service / Business class                                       │
│                                                                 │
│  ✅ Validates input (email format, salary range)               │
│  ✅ Applies business rules (no duplicate email, salary cap)    │
│  ✅ Orchestrates multiple DAL calls                            │
│  ✅ Transforms / calculates data                               │
│  ❌ No HTTP, no SqlConnection, no SQL queries                  │
├─────────────────────────────────────────────────────────────────┤
│  DATA ACCESS LAYER (DAL)                                        │
│  Repository / Data class                                        │
│                                                                 │
│  ✅ Opens SqlConnection                                         │
│  ✅ Executes SQL queries and stored procedures                  │
│  ✅ Returns data as models / DataTable / List<T>               │
│  ❌ No business logic, no HTTP concerns, no validation         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 The Request Flow

```
Browser sends HTTP request
        │
        ▼
┌───────────────────┐
│   CONTROLLER      │  Receives request, extracts data
│                   │  Calls BAL.Insert(emp)
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│       BAL         │  Validates: is email valid? is salary in range?
│                   │  Applies rules: is email unique?
│                   │  Calls DAL.Insert(emp)
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│       DAL         │  Opens SqlConnection
│                   │  Executes: INSERT INTO Employee...
│                   │  Returns rows affected
└────────┬──────────┘
         │
         ▼ result flows back up
┌───────────────────┐
│       BAL         │  Processes result, returns to Controller
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│   CONTROLLER      │  Returns View/JSON to browser
└───────────────────┘
```

---

## 🔷 Project Folder Structure

```
EmployeeApp/
│
├── Controllers/
│   └── EmployeeController.cs        ← Layer 1
│
├── BAL/  (or Services/)
│   └── EmployeeBAL.cs               ← Layer 2
│
├── DAL/  (or Repositories/)
│   └── EmployeeDAL.cs               ← Layer 3
│
├── Models/
│   └── Employee.cs                  ← Shared across all layers
│
├── Views/
│   └── Employee/
│       ├── Index.cshtml
│       └── Create.cshtml
│
└── appsettings.json                 ← Connection string here
```

---

## 🔷 What Each Layer Knows About

```
CONTROLLER knows about:  BAL only
                         (never imports DAL or SqlClient)

BAL knows about:         Controller inputs + DAL
                         (knows about models, never SqlConnection)

DAL knows about:         Database only
                         (SqlConnection, SqlCommand, SQL Server)

Models are shared:       All three layers use Employee, Department etc.
```

---

## 🔷 Dependency Registration — Program.cs

```csharp
// Register all three layers via Dependency Injection
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();

// Controller is registered automatically by AddControllersWithViews()
builder.Services.AddControllersWithViews();
```

---

## 🔷 Quick Code Sketch — All Three Layers

```csharp
// ── DAL — only talks to database ──────────────────────────────────
public class EmployeeDAL
{
    private readonly string _cs;
    public EmployeeDAL(IConfiguration config)
        => _cs = config.GetConnectionString("DefaultConnection");

    public int Insert(Employee emp)
    {
        // SqlConnection, SqlCommand, ExecuteNonQuery
        // Returns rows affected
    }

    public List<Employee> GetAll()
    {
        // SqlConnection, SqlCommand, ExecuteReader
        // Returns List<Employee>
    }
}


// ── BAL — business rules, calls DAL ──────────────────────────────
public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;
    public EmployeeBAL(EmployeeDAL dal) => _dal = dal;

    public string Insert(Employee emp)
    {
        // Validate salary range
        if (emp.Salary < 10000) return "Salary too low.";

        // Call DAL to check email uniqueness
        // (business rule — belongs in BAL)

        int rows = _dal.Insert(emp);
        return rows > 0 ? "success" : "insert failed";
    }
}


// ── Controller — handles HTTP, calls BAL ─────────────────────────
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Insert(emp);
        if (result == "success") return RedirectToAction("Index");

        ModelState.AddModelError("", result);
        return View(emp);
    }
}
```

---

## 🔷 Layers Responsibility Matrix

| Task                               | Controller      | BAL | DAL               |
| ---------------------------------- | --------------- | --- | ----------------- |
| Read form data from HTTP request   | ✅              | ❌  | ❌                |
| Validate email format              | ✅ (ModelState) | ✅  | ❌                |
| Check business rule (email unique) | ❌              | ✅  | ❌ (via DAL call) |
| Calculate salary after tax         | ❌              | ✅  | ❌                |
| Open SqlConnection                 | ❌              | ❌  | ✅                |
| Execute SQL query                  | ❌              | ❌  | ✅                |
| Return JSON / View                 | ✅              | ❌  | ❌                |
| Read appsettings.json              | ❌              | ❌  | ✅                |

---

## ⭐ Interview Quick-Fire

| Question                          | Answer                                                   |
| --------------------------------- | -------------------------------------------------------- |
| What is three-layer architecture? | Controller (HTTP), BAL (business rules), DAL (database)  |
| What does DAL stand for?          | Data Access Layer                                        |
| What does BAL stand for?          | Business Access Layer / Business Logic Layer             |
| Can Controller have SQL code?     | ❌ Never — SQL only in DAL                              |
| Can DAL have business rules?      | ❌ Never — business rules only in BAL                   |
| Who calls who?                    | Controller calls BAL, BAL calls DAL — never skip layers |
| What is shared across all layers? | Model classes (Employee, Department etc.)                |
| Where is connection string read?  | DAL — via IConfiguration injection                      |
