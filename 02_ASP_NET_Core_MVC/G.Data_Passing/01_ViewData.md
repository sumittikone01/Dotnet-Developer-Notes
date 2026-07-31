
# 01 — ViewData

---



## 🎯 One-Line Definition

> **ViewData is a dictionary (`ViewDataDictionary`) on the controller that passes data from a controller action to its view — data lives only for the duration of the current request and must be cast when read.**

---

## 🔷 Direction — Where ViewData Flows

```
Controller Action                        View (.cshtml)
──────────────────                       ──────────────────────
ViewData["Key"] = value;   ────────►     @ViewData["Key"]

ONE WAY. ONE REQUEST.
Data dies when the response is sent.
Does NOT survive a redirect.
```

---

## 🔷 What ViewData Actually Is

```csharp
// ViewData is a property on the Controller base class.
// Type: ViewDataDictionary
// Inherits from: Dictionary<string, object>

// Defined in ControllerBase as:
public ViewDataDictionary ViewData { get; set; }

// Because the value type is object, you MUST CAST when reading:
ViewData["PageTitle"] = "Employee List";   // stores as object
string title = (string)ViewData["PageTitle"];  // cast back to string
```

---

## 🔷 Setting ViewData in the Controller

```csharp
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    private readonly DepartmentBAL _deptBal;

    public EmployeeController(EmployeeBAL bal, DepartmentBAL deptBal)
    {
        _bal     = bal;
        _deptBal = deptBal;
    }

    public IActionResult Index()
    {
        // ── String values ──────────────────────────────────────────
        ViewData["PageTitle"]    = "Employee Management";
        ViewData["SubTitle"]     = "All Active Employees";
        ViewData["CurrentUser"]  = "John Smith";

        // ── Numeric values ─────────────────────────────────────────
        ViewData["TotalCount"]   = 47;
        ViewData["PageSize"]     = 10;

        // ── Boolean values ─────────────────────────────────────────
        ViewData["IsAdmin"]      = User.IsInRole("Admin");
        ViewData["ShowSalary"]   = User.IsInRole("Admin") || User.IsInRole("HR");

        // ── Complex objects ────────────────────────────────────────
        // Useful for secondary data (dropdowns, lookup lists, metadata)
        ViewData["DeptList"]     = _deptBal.GetAll();     // List<Department>
        ViewData["StatusList"]   = GetStatusOptions();    // SelectList

        // ── Main model data — passed via View(model), not ViewData ─
        var employees = _bal.GetActiveEmployees();
        return View(employees);
        // employees = main model → @model in view
        // ViewData  = supporting data alongside the main model
    }
}
```

---

## 🔷 Reading ViewData in the View

```html
@model List<Employee>
<!-- ↑ main model is List<Employee> — passed via return View(model) -->
<!-- ↑ ViewData carries supporting data alongside -->

<!-- ── Reading strings ────────────────────────────────────────── -->
<h1>@ViewData["PageTitle"]</h1>
<!-- No cast needed in Razor — Razor calls .ToString() automatically -->

<h2>@ViewData["SubTitle"]</h2>

<!-- ── Reading with cast ──────────────────────────────────────── -->
<!-- Cast is required when you need the actual typed value: -->
@{
    int total      = (int)ViewData["TotalCount"];
    bool isAdmin   = (bool)ViewData["IsAdmin"];
    bool showSal   = (bool)ViewData["ShowSalary"];
}

Total employees: @total

<!-- ── Conditional rendering based on ViewData ───────────────── -->
@if ((bool)ViewData["ShowSalary"])
{
    <th>Salary</th>
}

<!-- ── Using a List from ViewData in a loop ──────────────────── -->
@{
    var deptList = (List<Department>)ViewData["DeptList"];
}
@foreach (var dept in deptList)
{
    <option value="@dept.DeptId">@dept.DeptName</option>
}

<!-- ── Using SelectList for a dropdown ───────────────────────── -->
<select asp-for="DepartmentId" asp-items="ViewData["StatusList"] as SelectList">
    <option value="">-- Select --</option>
</select>
```

---

## 🔷 Null Safety — ViewData Can Be Null

```html
<!-- ViewData["Key"] returns null if key was never set -->
<!-- Always check before casting to avoid NullReferenceException -->

<!-- ❌ CRASH if key was not set: -->
<h1>@((string)ViewData["PageTitle"])</h1>

<!-- ✅ Safe with null check: -->
<h1>@(ViewData["PageTitle"]?.ToString() ?? "Default Title")</h1>

<!-- ✅ Safe in a code block: -->
@{
    var title = ViewData["PageTitle"] as string ?? "Default Title";
    int count = ViewData["TotalCount"] is int n ? n : 0;
}
```

---

## 🔷 ViewData in Layout Pages

```html
<!-- Views/Shared/_Layout.cshtml — sets the page <title> -->
<title>
    @(ViewData["PageTitle"]?.ToString() ?? "My App")
    <!-- Every view can set ViewData["PageTitle"] to control <title> tag -->
</title>

<!-- In Views/Employee/Index.cshtml: -->
@{
    ViewData["PageTitle"] = "Employee List | My App";
    //       ↑ This is read by _Layout.cshtml above
}
```

```csharp
// Views flow data UP to Layout via ViewData:
// View sets ViewData["PageTitle"]  →  Layout reads ViewData["PageTitle"]
// This is one of ViewData's most common real-world uses
```

---

## 🔷 ViewData vs ViewBag vs TempData — At a Glance

```
┌──────────────────┬───────────────────────────────────────────────┐
│                  │  ViewData                                     │
├──────────────────┼───────────────────────────────────────────────┤
│  What it is      │  Dictionary<string, object>                   │
│  Syntax (set)    │  ViewData["Key"] = value;                     │
│  Syntax (get)    │  (Type)ViewData["Key"]   ← cast required      │
│  Type safety     │  ❌ No — stores as object, must cast          │
│  IntelliSense    │  ❌ No — string key, no autocomplete          │
│  Null safe       │  ❌ No — throws if null and you cast it       │
│  Survives redirect│ ❌ No — current request only                 │
│  Where declared  │  Controller base class property               │
│  Best for        │  Page title, secondary dropdowns,             │
│                  │  boolean flags, count values                  │
└──────────────────┴───────────────────────────────────────────────┘
```

---

## 🔷 When to Use ViewData — and When Not To

```
USE ViewData for:
──────────────────────────────────────────────────────────────
✅ Setting the <title> tag from views (ViewData["PageTitle"])
✅ Small supporting data: booleans, strings, counts
✅ Dropdown lists alongside the main model
✅ Passing data between a view and its layout

DO NOT use ViewData for:
──────────────────────────────────────────────────────────────
❌ Your main page data — use a Strongly Typed Model instead
   (return View(employeeList) not ViewData["Employees"] = list)

❌ Data that must survive a redirect — use TempData instead

❌ Large or complex objects where you need IntelliSense
   → use a ViewModel (Strongly Typed View) for those

REAL PATTERN — ViewData for supporting, Model for main:
──────────────────────────────────────────────────────────────
public IActionResult Create()
{
    // Supporting data → ViewData (dropdown lists, flags)
    ViewData["DeptList"] = new SelectList(_deptBal.GetAll(), "Id", "Name");
    ViewData["PageTitle"] = "Add Employee";

    // Main data → model (return View(model))
    return View(new Employee());  // or return View() for empty form
}
```

---

## 🔷 ViewData in Partial Views

```csharp
// Parent view's ViewData flows into partial views automatically
// when you use Html.PartialAsync with ViewData:

// Index.cshtml:
ViewData["ShowSalary"] = User.IsInRole("Admin");

// Pass ViewData down to partial:
@await Html.PartialAsync("_EmployeeRow", emp,
    new ViewDataDictionary(ViewData))
// ↑ Without new ViewDataDictionary(ViewData), partial gets a fresh empty one
```

```html
<!-- _EmployeeRow.cshtml — reads from ViewData passed by parent -->
@model Employee
<tr>
    <td>@Model.EmpName</td>
    @if ((bool)(ViewData["ShowSalary"] ?? false))
    {
        <td>@Model.Salary.ToString("C")</td>
    }
</tr>
```

---

## ⭐ Interview Quick-Fire

| Question                                                             | Answer                                                                                                                                                   |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What is ViewData?                                                    | A `Dictionary<string, object>`property on the controller used to pass data from an action to its view                                                  |
| What type is ViewData?                                               | `ViewDataDictionary`— inherits from `Dictionary<string, object>`                                                                                    |
| Does ViewData require casting?                                       | ✅ Yes — values are stored as `object`, must be cast when reading                                                                                     |
| Does ViewData survive a redirect?                                    | ❌ No — lives only for the current request                                                                                                              |
| What is the most common real-world use of `ViewData["PageTitle"]`? | Setting the `<title>`tag in `_Layout.cshtml`from individual views                                                                                    |
| What is the difference between ViewData and ViewBag?                 | ViewData is a dictionary (`ViewData["Key"]`). ViewBag is a dynamic wrapper around the same dictionary (`ViewBag.Key`) — same data, different syntax |
| Should you use ViewData for your main page data?                     | ❌ No — use a Strongly Typed Model. ViewData is for small supporting data alongside the main model                                                      |
| Can a partial view access the parent view's ViewData?                | ✅ Yes — if you pass `new ViewDataDictionary(ViewData)`when invoking the partial                                                                      |
