
# 04 — Partial Views

---

## 🎯 One-Line Definition

> **A Partial View is a reusable Razor file that renders a fragment of HTML — not a full page — used to break large views into smaller pieces, avoid duplicate markup, and update sections of a page via AJAX without a full reload.**

---

## 🔷 Full View vs Partial View

```
Full View (Index.cshtml):                Partial View (_EmployeeRow.cshtml):
──────────────────────────────────       ──────────────────────────────────
Returns complete HTML page               Returns HTML fragment ONLY

Includes _Layout.cshtml                  NO layout
Has <html>, <head>, <body>               Just the fragment markup

Called by: return View(model)            Called by: return PartialView(...)
                                         OR: <partial name="..." />
                                         OR: @Html.Partial(...)
                                         OR: AJAX → innerHTML update

Response: full page for browser          Response: fragment for injection
```

---

## 🔷 Naming Convention

```
Partial view files start with underscore:  _FileName.cshtml
Full view files have no underscore:        Index.cshtml

_EmployeeRow.cshtml      ← partial
_EmployeeCard.cshtml     ← partial
_SearchFilter.cshtml     ← partial
Index.cshtml             ← full view
Create.cshtml            ← full view

WHY the underscore?
  Convention only — tells developers "this is a partial, not a full page"
  Underscore files are also excluded from direct URL routing
```

---

## 🔷 Where Partial Views Live

```
Views/
├── Employee/
│   ├── Index.cshtml               ← full view
│   ├── _EmployeeRow.cshtml        ← partial used only in Employee views
│   └── _SearchFilter.cshtml       ← partial used only in Employee views
│
└── Shared/
    ├── _Layout.cshtml             ← layout (special partial)
    ├── _LoginPartial.cshtml       ← used across the whole app
    ├── _EmployeeCard.cshtml       ← reused in multiple views
    └── _Pagination.cshtml         ← reused in multiple views
```

```
Search order when you call <partial name="_EmployeeCard" />:
──────────────────────────────────────────────────────────────
1. Views/{CurrentController}/_EmployeeCard.cshtml
2. Views/Shared/_EmployeeCard.cshtml
3. Error thrown: partial not found
```

---

## 🔷 Three Ways to Render a Partial View

### Way 1: Tag Helper `<partial>` — Recommended

```html
<!-- In Index.cshtml — synchronous, clean syntax -->

<!-- Simplest — no model passed -->
<partial name="_SearchFilter" />

<!-- Pass the current page's model to the partial -->
<partial name="_EmployeeCard" model="Model" />

<!-- Pass a specific property of the model -->
<partial name="_EmployeeCard" model="Model.Employee" />

<!-- Pass a completely different object -->
<partial name="_EmployeeCard" model="new Employee { EmpName = 'Test' }" />

<!-- Loop — render partial for each item -->
@foreach (var emp in Model.Employees)
{
    <partial name="_EmployeeRow" model="emp" />
}
```

### Way 2: `@await Html.PartialAsync()` — Async Version

```html
<!-- Async partial rendering (preferred over synchronous Html.Partial) -->

@await Html.PartialAsync("_SearchFilter")

@await Html.PartialAsync("_EmployeeCard", Model.Employee)

<!-- With ViewData (extra data alongside the model) -->
@await Html.PartialAsync("_EmployeeCard", Model.Employee,
    new ViewDataDictionary(ViewData) { { "ShowActions", true } })
```

### Way 3: `return PartialView()` From Controller — For AJAX

```csharp
// Controller action that returns a partial for AJAX calls
public IActionResult GetEmployeeRow(int id)
{
    var emp = _bal.GetById(id);
    return PartialView("_EmployeeRow", emp);
    // Returns ONLY the HTML fragment — no layout, no full page
}

public IActionResult GetFilteredTable(string dept)
{
    var employees = _bal.GetByDepartment(dept);
    return PartialView("_EmployeeTable", employees);
}
```

```javascript
// AJAX call that receives partial HTML and injects it into the page
$.ajax({
    url:  '/Employee/GetEmployeeRow',
    type: 'GET',
    data: { id: 5 },
    success: function(html) {
        // html = the rendered HTML fragment from _EmployeeRow.cshtml
        $('#employee-container').html(html);
        // OR append to a list:
        $('#employee-list').append(html);
    }
});

// Department filter — reload just the table, not the full page
$('#ddlDepartment').change(function() {
    $.ajax({
        url:  '/Employee/GetFilteredTable',
        data: { dept: $(this).val() },
        success: function(html) {
            $('#tableContainer').html(html);   // inject rendered HTML
        }
    });
});
```

---

## 🔷 Partial View With and Without a Model

```html
<!-- ── Without a model — uses parent view's ViewData ─────────────── -->

<!-- _SearchFilter.cshtml — no @model directive -->
<div class="search-bar">
    <input type="text" id="txtSearch" placeholder="Search employees..." />
    <button onclick="searchEmployees()">Search</button>
</div>

<!-- Called from Index.cshtml: no model needed -->
<partial name="_SearchFilter" />


<!-- ── With a strongly typed model ──────────────────────────────── -->

<!-- _EmployeeCard.cshtml -->
@model Employee
<div class="employee-card">
    <h3>@Model.EmpName</h3>
    <p>@Model.Department</p>
    <p>Salary: @Model.Salary.ToString("C")</p>
    <button onclick="editEmployee(@Model.EmpId)">Edit</button>
</div>

<!-- Called from Index.cshtml — pass each employee: -->
@foreach (var emp in Model.Employees)
{
    <partial name="_EmployeeCard" model="emp" />
}


<!-- ── With a List model ─────────────────────────────────────────── -->

<!-- _EmployeeTable.cshtml -->
@model List<Employee>
<table class="table">
    <thead>
        <tr><th>Name</th><th>Department</th><th>Salary</th></tr>
    </thead>
    <tbody>
        @foreach (var emp in Model)
        {
            <tr>
                <td>@emp.EmpName</td>
                <td>@emp.Department</td>
                <td>@emp.Salary.ToString("C")</td>
            </tr>
        }
    </tbody>
</table>
```

---

## 🔷 Passing Extra Data — ViewData in Partials

```csharp
// Sometimes the partial needs data beyond just the model
// Use ViewData for small extras

// Controller:
public IActionResult Index()
{
    var employees = _bal.GetAll();
    ViewData["ShowSalary"]    = User.IsInRole("Admin"); // admin sees salary
    ViewData["CurrentUserId"] = GetCurrentUserId();
    return View(employees);
}
```

```html
<!-- Index.cshtml passes ViewData through to partial: -->
@foreach (var emp in Model)
{
    @await Html.PartialAsync("_EmployeeRow", emp,
        new ViewDataDictionary(ViewData))
    <!-- ViewData["ShowSalary"] is now available inside _EmployeeRow -->
}

<!-- _EmployeeRow.cshtml -->
@model Employee
<tr>
    <td>@Model.EmpName</td>
    <td>@Model.Department</td>
    @if ((bool)ViewData["ShowSalary"] == true)
    {
        <td>@Model.Salary.ToString("C")</td>   <!-- only admins see this -->
    }
</tr>
```

---

## 🔷 Real Pattern — AJAX Table Refresh

```csharp
// Full Index action — loads the page with the filter form
public IActionResult Index()
{
    ViewBag.Departments = _deptBal.GetAll();
    var employees = _bal.GetAll();
    return View(employees);
}

// Partial action — called by AJAX when filter changes
public IActionResult FilterTable(string dept, string search)
{
    var employees = _bal.GetFiltered(dept, search);
    return PartialView("_EmployeeTable", employees);
    // Returns ONLY the table HTML — no nav bar, no filter form
}
```

```html
<!-- Views/Employee/Index.cshtml — full page -->
@model List<Employee>
<h1>Employees</h1>

<!-- Filter controls -->
<partial name="_SearchFilter" />

<!-- Table area — this div gets replaced by AJAX -->
<div id="tableWrapper">
    <partial name="_EmployeeTable" model="Model" />
</div>

<script>
function applyFilter() {
    $.ajax({
        url:  '/Employee/FilterTable',
        data: {
            dept:   $('#ddlDept').val(),
            search: $('#txtSearch').val()
        },
        success: function(html) {
            $('#tableWrapper').html(html);  // replace table, keep rest of page
        }
    });
}
</script>
```

```html
<!-- Views/Employee/_EmployeeTable.cshtml — the partial -->
@model List<Employee>
<table class="k-grid">
    <thead>
        <tr>
            <th>Name</th><th>Department</th><th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var emp in Model)
        {
            <tr>
                <td>@emp.EmpName</td>
                <td>@emp.Department</td>
                <td>
                    <button onclick="editEmployee(@emp.EmpId)">Edit</button>
                    <button onclick="deleteEmployee(@emp.EmpId)">Delete</button>
                </td>
            </tr>
        }
    </tbody>
</table>
<p>Showing @Model.Count employees</p>
```

---

## 🔷 Partial Views vs Other Options

```
┌──────────────────┬───────────────────────────────────────────────┐
│  Option          │  Best For                                     │
├──────────────────┼───────────────────────────────────────────────┤
│  Partial View    │  Reusable HTML fragments, AJAX HTML injection  │
│                  │  Simple — just HTML + model, no logic          │
├──────────────────┼───────────────────────────────────────────────┤
│  View Component  │  Self-contained widget that needs its OWN      │
│  (next chapter)  │  data from DB — not passed from parent view    │
├──────────────────┼───────────────────────────────────────────────┤
│  Layout          │  Shared chrome (header, footer, nav)           │
│  _Layout.cshtml  │  Wraps every full page                        │
├──────────────────┼───────────────────────────────────────────────┤
│  Editor Template │  Custom input field renderers                  │
│                  │  Used in forms with Html.EditorFor()           │
└──────────────────┴───────────────────────────────────────────────┘

USE PARTIAL VIEW when:
  ✅ You want to reuse a chunk of HTML across pages
  ✅ You want to AJAX-update part of a page with HTML
  ✅ The data is already available in the parent view's model
  ✅ No separate database call needed

USE VIEW COMPONENT instead when:
  ✅ The fragment needs its OWN data from the database
  ✅ The logic for getting data is complex
  ✅ You want a self-contained, testable widget
```

---

## ⭐ Interview Quick-Fire

| Question                                                       | Answer                                                                                                          |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| What is a partial view?                                        | A reusable Razor file that renders an HTML fragment — no layout, no full page                                  |
| What is the naming convention for partial views?               | Prefix with underscore:`_EmployeeCard.cshtml`                                                                 |
| Where do shared partials live?                                 | `Views/Shared/`— available to all controllers                                                                |
| What tag helper renders a partial?                             | `<partial name="_FileName" model="..." />`                                                                    |
| How do you return a partial from a controller for AJAX?        | `return PartialView("_PartialName", model)`— returns HTML fragment                                           |
| What is the difference between a partial and a View Component? | Partial = simple HTML reuse, data from parent. View Component = self-contained widget with its own data from DB |
| What does `@await Html.PartialAsync(...)`do?                 | Asynchronously renders a partial view and injects the HTML into the current view                                |
| Can a partial view have its own `@model`directive?           | ✅ Yes — strongly typed partial:`@model Employee`                                                            |
