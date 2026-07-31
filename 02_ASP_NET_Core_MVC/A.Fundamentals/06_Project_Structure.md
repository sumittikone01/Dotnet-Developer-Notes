# 06 — Project Structure

## 📌 What is it?

The **default folder/file layout** of an ASP.NET Core MVC project — knowing what lives where is essential for navigating any project quickly, including ones you didn't build yourself.

## 🖼 Default MVC project layout

```
MyApp/
│
├── Controllers/              ← C# classes handling requests, one per resource
│   └── HomeController.cs
│
├── Models/                   ← Data classes, ViewModels, entities
│   └── Product.cs
│
├── Views/                    ← Razor (.cshtml) templates, mirrors Controller names
│   ├── Home/
│   │   ├── Index.cshtml
│   │   └── Privacy.cshtml
│   ├── Shared/                ← Views/partials shared across controllers
│   │   ├── _Layout.cshtml     ← master page template
│   │   └── _ValidationScriptsPartial.cshtml
│   └── _ViewStart.cshtml      ← runs before every view (sets default layout)
│
├── wwwroot/                  ← STATIC files only (CSS, JS, images, libs)
│   ├── css/
│   ├── js/
│   ├── lib/                   ← client libraries (jQuery, Bootstrap, Kendo, etc.)
│   └── images/
│
├── Properties/
│   └── launchSettings.json    ← local dev run/debug configuration
│
├── appsettings.json           ← main configuration file
├── appsettings.Development.json ← environment-specific overrides
├── Program.cs                 ← app entry point (see topic 05)
└── MyApp.csproj                ← project file (dependencies, SDK version)
```

## 🤔 Why do we need to know this?

Convention-based structure means ASP.NET Core can **automatically find things** without you configuring paths manually:

- A request to `/Home/Index` automatically maps to `HomeController.Index()` → looks for `Views/Home/Index.cshtml`
- Anything in `wwwroot/` is automatically servable as a static file (e.g., `wwwroot/css/site.css` → `https://yoursite.com/css/site.css`)

This "convention over configuration" approach is central to how MVC minimizes boilerplate.

## 🧠 Intuition

Think of it like a **well-organized office building**:

- `Controllers/` = the reception desks (one per department, handles incoming requests)
- `Models/` = the filing cabinets (where data structure lives)
- `Views/` = the presentation materials handed to visitors
- `wwwroot/` = the public lobby — anyone (any client) can grab things from here directly, no processing needed

## 📊 Key folders explained

| Folder/File                                    | Purpose                                                                                    |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `Controllers/`                               | Handles incoming requests, orchestrates Model + View                                       |
| `Models/`                                    | Entity classes, ViewModels, DTOs                                                           |
| `Views/{ControllerName}/{ActionName}.cshtml` | Razor view matching a specific action —**naming convention is critical**            |
| `Views/Shared/`                              | Views reused across multiple controllers (layouts, partials, error pages)                  |
| `Views/_ViewStart.cshtml`                    | Executes before each view renders — typically sets the default`_Layout.cshtml`          |
| `Views/_ViewImports.cshtml`                  | Central place for`@using` statements and Tag Helper registration, shared by all views    |
| `wwwroot/`                                   | The ONLY folder servable as static content by default — CSS, JS, images, client libraries |
| `appsettings.json`                           | Configuration (connection strings, app settings, logging levels)                           |
| `Program.cs`                                 | Bootstraps everything                                                                      |

## 📊 Comparison: Web Forms structure vs MVC structure

| Web Forms                                                      | MVC                                                              |
| -------------------------------------------------------------- | ---------------------------------------------------------------- |
| `.aspx` + `.aspx.cs` code-behind pairs, scattered anywhere | `Controllers/` (logic) + `Views/` (markup) cleanly separated |
| No enforced folder convention                                  | Strong naming convention:`Views/{Controller}/{Action}.cshtml`  |
| Static files mixed anywhere in project                         | All static content isolated in`wwwroot/`                       |

## 🚨 Common mistakes

- Putting static files (CSS/JS/images) **outside** `wwwroot/` and expecting them to be servable — by default, only `wwwroot/` is exposed to HTTP requests.
- Naming a View file incorrectly (e.g., `views/home/index.cshtml` lowercase mismatch on case-sensitive Linux deployments) — works fine on Windows dev machine, breaks in Linux production/Docker.
- Forgetting `_ViewStart.cshtml` — views render without the expected layout/master page.

## 💡 Best practices

- Keep `wwwroot/` organized into subfolders (`css/`, `js/`, `lib/`, `images/`) rather than dumping everything at the root.
- Use `Views/Shared/` for any partial view or layout used by more than one Controller.
- Match folder/file casing consistently — even though Windows is case-insensitive, always assume Linux-style case sensitivity for production safety.

## 🎤 Interview questions

1. How does ASP.NET Core know which `.cshtml` file to render for a given Controller action?
2. Why is `wwwroot/` special, and what happens if you put a JS file outside of it?
3. What's the purpose of `_ViewStart.cshtml` and `_ViewImports.cshtml`?
4. How would the default project structure need to change for a Web API-only project (no Views)?

## 📝 30-second revision cheat sheet

- `Controllers/` → logic, `Views/` → UI templates, `Models/` → data shapes, `wwwroot/` → static assets.
- View naming convention: `Views/{ControllerName}/{ActionName}.cshtml`.
- Only `wwwroot/` is servable as static content by default.
- `_ViewStart.cshtml` sets default layout; `_ViewImports.cshtml` centralizes `@using`/Tag Helpers

# 06 — Project Structure

---

## 🎯 One-Line Definition

> **An ASP.NET Core MVC project has a specific folder layout where Controllers handle requests, Models hold data, Views render HTML, and `wwwroot` holds static files — understanding what lives where tells you exactly where to look and where to put things.**

---

## 🔷 The Complete Folder Structure — Your Real Project

```
EmployeeManagement/                  ← Solution root
│
├── EmployeeManagement.csproj        ← Project file (NuGet packages, .NET version)
├── Program.cs                       ← Entry point (DI + middleware pipeline)
├── appsettings.json                 ← Config (connection strings, settings)
├── appsettings.Development.json     ← Dev overrides (local DB, verbose logging)
│
├── Controllers/                     ← Handle HTTP requests, call BAL, return views/JSON
│   ├── HomeController.cs            ← Routes: /Home/Index, /Home/About
│   ├── EmployeeController.cs        ← Routes: /Employee/Index, /Employee/Create
│   └── API/                         ← Sub-folder for Web API controllers
│       ├── EmployeeApiController.cs ← Routes: /api/employee (returns JSON)
│       └── DepartmentApiController.cs
│
├── Models/                          ← C# classes that represent your data
│   ├── Employee.cs                  ← Maps to Employees table in SQL
│   ├── Department.cs
│   └── ViewModels/                  ← Data shaped specifically for Views
│       ├── EmployeeListViewModel.cs ← What the Index view needs
│       └── EmployeeFormViewModel.cs ← What the Create/Edit form needs
│
├── BAL/                             ← Business Access Layer — business rules
│   ├── EmployeeBAL.cs
│   └── DepartmentBAL.cs
│
├── DAL/                             ← Data Access Layer — all ADO.NET code here
│   ├── EmployeeDAL.cs
│   └── DepartmentDAL.cs
│
├── Views/                           ← Razor (.cshtml) files — HTML templates
│   ├── _ViewStart.cshtml            ← Sets Layout = "_Layout" for all views
│   ├── _ViewImports.cshtml          ← Shared using statements for Razor
│   ├── Shared/                      ← Views used across the whole app
│   │   ├── _Layout.cshtml           ← Master page (header, footer, nav)
│   │   ├── _LoginPartial.cshtml
│   │   └── Error.cshtml
│   ├── Home/
│   │   └── Index.cshtml
│   └── Employee/
│       ├── Index.cshtml             ← List page (has Kendo Grid)
│       ├── Create.cshtml
│       └── Edit.cshtml
│
├── wwwroot/                         ← Static files served directly to browser
│   ├── css/
│   │   └── site.css
│   ├── js/
│   │   └── site.js
│   ├── lib/
│   │   ├── jquery/
│   │   │   └── jquery.min.js
│   │   ├── bootstrap/
│   │   └── kendo/                   ← Kendo UI CSS + JS files go here
│   │       ├── kendo.all.min.js
│   │       └── kendo.default.min.css
│   └── images/
│
├── Filters/                         ← Custom action/exception filters
│   └── ApiLoggingFilter.cs
│
├── Middleware/                      ← Custom middleware classes
│   └── RequestLoggingMiddleware.cs
│
└── Properties/
    └── launchSettings.json          ← Dev server config (ports, environment)
```

---

## 🔷 Each Folder — Purpose and Rules

### Controllers/

```csharp
// EmployeeController.cs — MVC controller (returns Views)
public class EmployeeController : Controller
//                                ↑ Controller = MVC (has View(), ViewBag etc.)
{
    // ← URL: GET /Employee/Index
    public IActionResult Index()
    {
        return View();  // → looks for Views/Employee/Index.cshtml
    }
}

// API/EmployeeApiController.cs — API controller (returns JSON)
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
//                                    ↑ ControllerBase = API only (no views)
{
    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(data);  // → returns JSON
    }
}
```

```
Naming rule: class EmployeeController → URL prefix /Employee
             class HomeController     → URL prefix /Home
             class API/EmployeeApiController + [Route("api/employee")] → /api/employee
```

---

### Models/

```csharp
// Models/Employee.cs — maps to the Employees table in SQL Server
public class Employee
{
    public int    EmpId      { get; set; }
    public string EmpName    { get; set; }
    public string Department { get; set; }
    public decimal Salary    { get; set; }
    public bool   IsActive   { get; set; }
}

// Models/ViewModels/EmployeeListViewModel.cs — shaped for the Index view
// Different from the DB model — has exactly what the view needs, no more
public class EmployeeListViewModel
{
    public List<Employee>    Employees    { get; set; }
    public List<Department>  Departments  { get; set; }  // for filter dropdown
    public int               TotalCount   { get; set; }
    public int               CurrentPage  { get; set; }
}
```

```
Models/         → mirrors DB tables (used in DAL)
ViewModels/     → shaped for specific views (used in Controller → View)

Rule: never pass a raw DB model to a view if it has sensitive fields.
      Use a ViewModel instead — only include what the view needs.
```

---

### BAL/ and DAL/ — Your Company's Standard Pattern

```
Request Flow in Your App:
──────────────────────────────────────────────────────────────
Browser
  ↓ HTTP Request
Controller   ← receives request, validates, calls BAL
  ↓
BAL          ← business rules: validate salary, check duplicates
  ↓
DAL          ← pure database code: SqlConnection, SqlCommand
  ↓
SQL Server
```

```csharp
// DAL/EmployeeDAL.cs — ONLY database code here, no business logic
public class EmployeeDAL
{
    private readonly string _conn;

    public EmployeeDAL(IConfiguration config)
        => _conn = config.GetConnectionString("DefaultConnection");

    public List<Employee> GetAll()
    {
        var list = new List<Employee>();
        using var con = new SqlConnection(_conn);
        using var cmd = new SqlCommand("SELECT * FROM Employees", con);
        con.Open();
        using var rdr = cmd.ExecuteReader();
        while (rdr.Read())
            list.Add(new Employee { /* map columns */ });
        return list;
    }
}

// BAL/EmployeeBAL.cs — business rules, calls DAL
public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;

    public EmployeeBAL(EmployeeDAL dal) => _dal = dal;

    public List<Employee> GetActiveEmployees()
    {
        var all = _dal.GetAll();
        return all.Where(e => e.IsActive).ToList(); // business rule: active only
    }

    public bool AddEmployee(Employee emp)
    {
        if (emp.Salary < 0) return false;           // business rule: no negative salary
        return _dal.Insert(emp);
    }
}

// Controllers/EmployeeController.cs — calls BAL, returns View
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    public IActionResult Index()
    {
        var employees = _bal.GetActiveEmployees();
        return View(employees);
    }
}
```

---

### Views/

```
Views/
├── _ViewStart.cshtml      ← runs before every view, sets Layout
├── _ViewImports.cshtml    ← shared directives (using, tag helpers)
├── Shared/
│   ├── _Layout.cshtml     ← master page (nav bar, footer, scripts)
│   └── Error.cshtml
├── Home/
│   └── Index.cshtml       ← returned by HomeController.Index()
└── Employee/
    └── Index.cshtml       ← returned by EmployeeController.Index()
```

```html
<!-- Views/_ViewStart.cshtml — sets default layout for all views -->
@{
    Layout = "_Layout";   // all views use Shared/_Layout.cshtml by default
}

<!-- Views/_ViewImports.cshtml — available in all views without @using -->
@using EmployeeManagement.Models
@using EmployeeManagement.Models.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

```html
<!-- Views/Employee/Index.cshtml — strongly typed to a ViewModel -->
@model EmployeeListViewModel
<!-- ↑ tells Razor: Model is EmployeeListViewModel, enable IntelliSense -->

<h1>Employees (@Model.TotalCount)</h1>
<div id="employeeGrid"></div>

@section Scripts {
    <!-- Scripts section — injected into _Layout.cshtml's @RenderSection("Scripts") -->
    <script>
        $('#employeeGrid').kendoGrid({ /* ... */ });
    </script>
}
```

---

### wwwroot/

```
wwwroot/ — ALL static files served DIRECTLY by UseStaticFiles()
           No controller involved. No auth. Just files.

URL → File mapping:
──────────────────────────────────────────────────────────────
/css/site.css         → wwwroot/css/site.css
/js/site.js           → wwwroot/js/site.js
/lib/jquery/jquery.min.js  → wwwroot/lib/jquery/jquery.min.js
/lib/kendo/kendo.all.min.js → wwwroot/lib/kendo/kendo.all.min.js
/images/logo.png      → wwwroot/images/logo.png
```

```html
<!-- Referencing wwwroot files in Razor views: -->

<!-- Method 1: Using Tag Helper (recommended — validates file exists) -->
<link rel="stylesheet" href="~/lib/kendo/kendo.default.min.css" />
<script src="~/lib/kendo/kendo.all.min.js"></script>
<!--                   ↑ ~ = wwwroot root -->

<!-- Method 2: Hardcoded path -->
<script src="/lib/jquery/jquery.min.js"></script>
```

---

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB;Integrated Security=True;TrustServerCertificate=True"
  },
  "AppSettings": {
    "PageSize": 10,
    "AppName": "Employee Management System",
    "MaxLoginAttempts": 5
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

```json
// appsettings.Development.json — overrides for your dev machine only
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB_Dev;Integrated Security=True;TrustServerCertificate=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

---

### launchSettings.json

```json
// Properties/launchSettings.json — dev-only, NOT deployed to production
{
  "profiles": {
    "EmployeeManagement": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7001;http://localhost:5001",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
        // ↑ This is why IsDevelopment() returns true locally
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

---

### .csproj — The Project File

```xml
<!-- EmployeeManagement.csproj — SDK, .NET version, NuGet packages -->
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>   <!-- which .NET version -->
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>     <!-- global using statements -->
  </PropertyGroup>

  <ItemGroup>
    <!-- NuGet packages your project uses: -->
    <PackageReference Include="Microsoft.Data.SqlClient" Version="5.1.5" />
    <!-- ↑ ADO.NET SQL Server provider -->
  </ItemGroup>

</Project>
```

---

## 🔷 How a Request Flows Through the Structure

```
User visits: https://localhost:7001/Employee/Index
───────────────────────────────────────────────────────────────────
1. Kestrel receives HTTP request

2. Program.cs middleware pipeline runs:
   UseStaticFiles  → not a static file, pass through
   UseRouting      → matches: controller=Employee, action=Index
   UseAuthorization → passed (or redirects to login)

3. EmployeeController.Index() is called

4. Index() calls: _bal.GetActiveEmployees()
   BAL calls:     _dal.GetAll()
   DAL opens:     SqlConnection → SqlCommand → ExecuteReader
   Returns:       List<Employee>

5. Controller calls: return View(employees);

6. Razor Engine finds: Views/Employee/Index.cshtml
   Renders HTML using the List<Employee> as @Model

7. HTML response sent back to browser
───────────────────────────────────────────────────────────────────
```

```
User's Kendo Grid loads data: GET /api/employee?skip=0&take=10
───────────────────────────────────────────────────────────────────
1. Same middleware pipeline

2. UseRouting → matches: API/EmployeeApiController.GetAll()

3. EmployeeApiController.GetAll() is called
   Calls: _bal.GetAll(skip, take)
   Returns: List<Employee>

4. return Ok(list) → ASP.NET Core serializes to JSON

5. JSON response: [{"empId":1,"empName":"John",...},...]

6. Kendo DataSource receives JSON → Grid renders rows
───────────────────────────────────────────────────────────────────
```

---

## 🔷 Where Does Each File Type Live — Quick Reference

| What                 | Where                           | Why                           |
| -------------------- | ------------------------------- | ----------------------------- |
| Connection string    | `appsettings.json`            | Config, not hardcoded         |
| C# class (entity)    | `Models/`                     | Data shape                    |
| ADO.NET code         | `DAL/`                        | All DB access in one place    |
| Business rules       | `BAL/`                        | Logic separated from data     |
| HTTP request handler | `Controllers/`                | Entry point for each URL      |
| HTML template        | `Views/ControllerName/`       | Razor view                    |
| Master layout        | `Views/Shared/_Layout.cshtml` | Shared across all pages       |
| CSS file             | `wwwroot/css/`                | Static, served directly       |
| JavaScript file      | `wwwroot/js/`                 | Static, served directly       |
| Kendo files          | `wwwroot/lib/kendo/`          | Static, served directly       |
| Custom middleware    | `Middleware/`                 | Request pipeline component    |
| DI registrations     | `Program.cs`                  | Service container setup       |
| Route configuration  | `Program.cs`                  | Where URLs map to controllers |

---

## ⭐ Interview Quick-Fire

| Question                                                            | Answer                                                                                        |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| What is the purpose of the`wwwroot`folder?                        | Stores static files (CSS, JS, images) served directly to the browser by`UseStaticFiles()`   |
| What does`~`mean in a Razor view path?                            | Root of`wwwroot`—`~/css/site.css`→`wwwroot/css/site.css`                              |
| What is`_Layout.cshtml`?                                          | The master page (shared header, footer, nav) that all views use by default                    |
| What is`_ViewStart.cshtml`?                                       | Runs before every view — typically sets`Layout = "_Layout"`                                |
| What is`_ViewImports.cshtml`?                                     | Adds shared`@using`and `@addTagHelper`directives to all views                             |
| What is a ViewModel?                                                | A C# class shaped for a specific view — not a DB entity, but exactly what the view needs     |
| What is`launchSettings.json`?                                     | Dev-only config file for local server ports and environment variables — NOT deployed         |
| What is`.csproj`for?                                              | Defines the .NET version and NuGet package references for the project                         |
| What is the difference between`Controller`and `ControllerBase`? | `Controller`= MVC (has `View()`,`ViewBag`)`ControllerBase`= API only (JSON responses) |
| Where do you register your DAL in ASP.NET Core?                     | `Program.cs`—`builder.Services.AddScoped<EmployeeDAL>()`                                 |
