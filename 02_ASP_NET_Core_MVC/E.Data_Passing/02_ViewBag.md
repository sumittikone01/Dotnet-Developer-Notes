
# 02 — ViewBag

---

## 🎯 One-Line Definition

> **ViewBag is a `dynamic` property on the controller that wraps ViewData — it passes data from controller to view using dot-notation instead of dictionary syntax, but it's the exact same underlying dictionary with no type safety and no IntelliSense.**

---

## 🔷 Direction — Where ViewBag Flows

```
Controller Action                        View (.cshtml)
──────────────────                       ──────────────────────
ViewBag.PageTitle = "List";  ────────►   @ViewBag.PageTitle

ONE WAY. ONE REQUEST.
Data dies when the response is sent.
Does NOT survive a redirect.
```

---

## 🔷 ViewBag IS ViewData — Same Dictionary, Different Syntax

```csharp
// This is the definition of ViewBag in ASP.NET Core:
public dynamic ViewBag
{
    get { return new DynamicViewData(() => ViewData); }
    //                               ↑ wraps ViewData
}

// PROOF — these two lines are IDENTICAL:
ViewData["PageTitle"] = "Employee List";
ViewBag.PageTitle     = "Employee List";
// → BOTH write to the same dictionary entry

// Reading either way also works interchangeably:
var a = (string)ViewData["PageTitle"];  // "Employee List"
var b = ViewBag.PageTitle;              // "Employee List"

// Set with ViewData, read with ViewBag:
ViewData["DeptList"] = deptList;
var list = ViewBag.DeptList;    // works — reads from same dictionary
```

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   ViewBag.SomeKey = value                                 │
│                 │                                         │
│                 ▼                                         │
│   SAME underlying ViewDataDictionary["SomeKey"] = value   │
│                 ▲                                         │
│                 │                                         │
│   ViewData["SomeKey"] = value                             │
│                                                           │
│   Both access the SAME dictionary.                        │
│   ViewBag = syntactic sugar over ViewData.                │
└───────────────────────────────────────────────────────────┘
```

---

## 🔷 Setting ViewBag in the Controller

```csharp
public class EmployeeController : Controller
{
    private readonly EmployeeBAL    _bal;
    private readonly DepartmentBAL  _deptBal;

    public EmployeeController(EmployeeBAL bal, DepartmentBAL deptBal)
    {
        _bal     = bal;
        _deptBal = deptBal;
    }

    public IActionResult Index()
    {
        // ── String values ──────────────────────────────────────────
        ViewBag.PageTitle   = "Employee Management";
        ViewBag.SubTitle    = "All Active Employees";
        ViewBag.CurrentUser = HttpContext.User.Identity.Name;

        // ── Numeric ────────────────────────────────────────────────
        ViewBag.TotalCount  = _bal.GetCount();
        ViewBag.PageSize    = 10;

        // ── Boolean flags ──────────────────────────────────────────
        ViewBag.IsAdmin     = User.IsInRole("Admin");
        ViewBag.ShowSalary  = User.IsInRole("Admin") || User.IsInRole("HR");

        // ── SelectList for dropdowns ───────────────────────────────
        ViewBag.DeptList    = new SelectList(
            _deptBal.GetAll(), "DeptId", "DeptName");

        // ── Main page data goes via model, not ViewBag ─────────────
        var employees = _bal.GetActiveEmployees();
        return View(employees);
    }

    public IActionResult Create()
    {
        // Populate dropdowns for the create form:
        ViewBag.DeptList   = new SelectList(_deptBal.GetAll(), "DeptId", "DeptName");
        ViewBag.StatusList = new SelectList(new[]
        {
            new { Value = "Active",   Text = "Active" },
            new { Value = "Inactive", Text = "Inactive" }
        }, "Value", "Text");

        return View(new Employee());
    }

    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid)
        {
            // Re-populate dropdowns when returning to form with errors
            // (ViewBag is empty on POST — you must refill it)
            ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "DeptId", "DeptName");
            return View(emp);
        }
        _bal.Add(emp);
        return RedirectToAction("Index");
    }
}
```

---

## 🔷 Reading ViewBag in the View

```html
@model List<Employee>

<!-- ── Reading simple values ──────────────────────────────────── -->
<title>@ViewBag.PageTitle</title>
<h1>@ViewBag.PageTitle</h1>
<p>@ViewBag.SubTitle</p>
<small>Total: @ViewBag.TotalCount employees</small>

<!-- ── No cast needed for display in Razor ───────────────────── -->
<!-- Razor calls .ToString() on dynamic values automatically -->

<!-- ── Cast required for logic/conditions ────────────────────── -->
@if (ViewBag.IsAdmin)
{
    <!-- Works: dynamic resolves to bool, C# evaluates it -->
    <button>Delete All</button>
}

@if ((bool)ViewBag.ShowSalary)
{
    <th>Salary</th>
}

<!-- ── Using ViewBag SelectList in dropdown ──────────────────── -->
<!-- With Tag Helper: -->
<select asp-for="DepartmentId"
        asp-items="@ViewBag.DeptList"
        class="form-control">
    <option value="">-- Select Department --</option>
</select>

<!-- With Html Helper (old way): -->
@Html.DropDownListFor(m => m.DepartmentId,
    ViewBag.DeptList as SelectList,
    "-- Select Department --",
    new { @class = "form-control" })

<!-- ── Looping over a list from ViewBag ──────────────────────── -->
@foreach (var dept in ViewBag.DeptList as SelectList ?? Enumerable.Empty<SelectListItem>())
{
    <li>@dept.Text</li>
}
```

---

## 🔷 ViewBag in Layout Pages

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <!-- Any view can set ViewBag.PageTitle — Layout reads it -->
    <title>@(ViewBag.PageTitle ?? "My App")</title>
</head>
<body>
    <!-- Render notification count set by controller: -->
    @if (ViewBag.UnreadCount != null && ViewBag.UnreadCount > 0)
    {
        <span class="badge">@ViewBag.UnreadCount</span>
    }

    @RenderBody()
</body>
</html>
```

```html
<!-- Views/Employee/Index.cshtml — sets ViewBag for Layout to read -->
@{
    ViewBag.PageTitle    = "Employee List | My App";
    ViewBag.UnreadCount  = 3;  // or loaded in a base controller
}
```

---

## 🔷 The Common Gotcha — Repopulating on POST

```csharp
// ❌ COMMON BUG:
[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
        return View(emp);
    // ↑ BUG: ViewBag.DeptList was set in the GET action, NOT here
    //   On POST, ViewBag starts empty. The dropdown is NULL in the view.
    //   → NullReferenceException or empty dropdown
}

// ✅ CORRECT — always repopulate ViewBag before returning View on POST:
[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
    {
        // Repopulate everything the view needs:
        ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "DeptId", "DeptName");
        return View(emp);
    }
    _bal.Add(emp);
    return RedirectToAction("Index");
}

// BETTER PATTERN — extract to a private method to avoid duplication:
private void PopulateCreateFormData()
{
    ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "DeptId", "DeptName");
    ViewBag.StatusList = new SelectList(GetStatusOptions(), "Value", "Text");
}

[HttpGet]
public IActionResult Create()
{
    PopulateCreateFormData();
    return View(new Employee());
}

[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
    {
        PopulateCreateFormData();   // ← same method
        return View(emp);
    }
    _bal.Add(emp);
    return RedirectToAction("Index");
}
```

---

## 🔷 ViewBag Limitations — Why Strongly Typed Views Are Better

```csharp
// ── No IntelliSense ────────────────────────────────────────────────
ViewBag.EmpoyeeList = employees;   // ← typo: "Empoyee" instead of "Employee"
// No error at compile time. Crashes at runtime.

// In view:
@foreach (var emp in ViewBag.EmployeeList)  // ← correct spelling
// → NullReferenceException at runtime because "EmployeeList" is null
//   "EmpoyeeList" was set, "EmployeeList" was read — silent bug

// ── No type checking ──────────────────────────────────────────────
ViewBag.Count = "not a number";  // stores a string
// In view: @ViewBag.Count + 1   → "not a number1"  (string concat)
// No warning, no error

// ── No compile-time validation ────────────────────────────────────
ViewBag.PageTitle = "Employee List";
// Controller compiles fine. View compiles fine.
// If you rename property in controller but not view → runtime error only.
```

---

## 🔷 ViewBag vs ViewData — One Final Comparison

```
┌──────────────────────┬────────────────────────┬──────────────────────────┐
│  Feature             │  ViewData              │  ViewBag                 │
├──────────────────────┼────────────────────────┼──────────────────────────┤
│  Underlying type     │  Dictionary<string,obj>│  Same dictionary         │
│  Set syntax          │  ViewData["Key"] = val │  ViewBag.Key = val       │
│  Get syntax          │  (Type)ViewData["Key"] │  ViewBag.Key             │
│  Cast required?      │  ✅ Yes                │  ❌ No (dynamic)         │
│  IntelliSense        │  ❌ No                 │  ❌ No                   │
│  Type safety         │  ❌ No                 │  ❌ No (worse — dynamic) │
│  Null safety         │  ❌ Check before cast  │  ❌ Null if not set      │
│  Survives redirect   │  ❌ No                 │  ❌ No                   │
│  Performance         │  Slightly faster       │  Slightly slower         │
│                      │                        │  (dynamic resolution)    │
│  When same key set   │  Read either way       │  Read either way         │
│  with both           │                        │  (SAME dictionary)       │
│  Best used for       │  Layout page title     │  Small supporting data   │
│                      │  dropdowns, flags      │  dropdowns, flags        │
└──────────────────────┴────────────────────────┴──────────────────────────┘

BOTTOM LINE:
ViewBag = cleaner syntax for setting, no cast needed for reading.
ViewData = slightly more explicit, same result.
Both = same limitations. Neither gives IntelliSense or type safety.
For anything complex → use a ViewModel (Strongly Typed View) instead.
```

---

## 🔷 When to Use ViewBag — Real Guidelines

```
USE ViewBag for:
──────────────────────────────────────────────────────────────
✅ Page title passed to _Layout:   ViewBag.PageTitle = "..."
✅ Dropdown SelectLists:           ViewBag.DeptList = new SelectList(...)
✅ Boolean display flags:          ViewBag.ShowSalary = isAdmin
✅ Small scalar values:            ViewBag.TotalCount = 47
✅ When you prefer dot syntax over ViewData["Key"] syntax

DO NOT use ViewBag for:
──────────────────────────────────────────────────────────────
❌ Main page data (employee list, order details)
   → Use a Strongly Typed Model

❌ Data that must survive a redirect (success/error messages)
   → Use TempData

❌ Large, complex objects where you need IntelliSense
   → Use a ViewModel

❌ Data shared between multiple controllers
   → Use Session or a service
```

---

## ⭐ Interview Quick-Fire

| Question                                                       | Answer                                                                                                                                |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| What is ViewBag?                                               | A `dynamic`property on the controller that wraps ViewData, allowing dot-notation access instead of dictionary syntax                |
| What is the underlying type of ViewBag?                        | `DynamicViewData`— wraps the same `ViewDataDictionary`that ViewData uses                                                         |
| Are ViewBag and ViewData the same data?                        | ✅ Yes —`ViewBag.Key = value`and `ViewData["Key"] = value`write to the exact same dictionary                                     |
| Does ViewBag require casting when reading?                     | ❌ No — it's `dynamic`, so no explicit cast is needed. But type errors only surface at runtime                                     |
| Does ViewBag survive a redirect?                               | ❌ No — lives only for the current request. Use TempData for redirect survival                                                       |
| What is the most common ViewBag bug?                           | Forgetting to repopulate dropdowns (`ViewBag.DeptList`) in the `[HttpPost]`action before returning the view on validation failure |
| Why is ViewBag considered worse than a Strongly Typed View?    | No IntelliSense, no compile-time type checking — typos in property names cause silent runtime NullReferenceExceptions                |
| What should you use instead of ViewBag for the main page data? | A Strongly Typed View —`return View(employeeList)`with `@model List<Employee>`                                                   |
| What is `ViewBag.PageTitle`commonly used for?                | Setting the `<title>`tag in `_Layout.cshtml`from individual views                                                                 |
