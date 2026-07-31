
# 04 — Strongly Typed Views

---

## 🎯 One-Line Definition

> **A Strongly Typed View uses `@model TypeName` to declare exactly what data it expects — giving you IntelliSense, compile-time error checking, and Tag Helper support, instead of blind casting from a loosely typed dictionary.**

---

## 🔷 Weakly Typed vs Strongly Typed — The Core Problem

```
WEAKLY TYPED (using ViewBag/ViewData):
──────────────────────────────────────────────────────────────
Controller:
  ViewBag.Employee = _bal.GetById(id);
  return View();

View:
  var emp = ViewBag.Employee;
  <p>@emp.Name</p>      ← no IntelliSense
  <p>@emp.Sallary</p>   ← TYPO — no compile error → crashes at runtime!

Problems:
  ❌ No IntelliSense — you type blindly
  ❌ Typos crash at runtime — not caught by compiler
  ❌ Refactoring breaks silently — rename a property = runtime error
  ❌ No Tag Helper support (asp-for doesn't work)
  ❌ Reviewers can't see what data the view needs

STRONGLY TYPED (@model):
──────────────────────────────────────────────────────────────
Controller:
  var emp = _bal.GetById(id);
  return View(emp);

View:
  @model Employee
  <p>@Model.Name</p>      ← full IntelliSense ✅
  <p>@Model.Sallary</p>   ← COMPILE ERROR — caught immediately ✅

Benefits:
  ✅ Full IntelliSense on Model properties
  ✅ Compile-time errors — typos caught before deployment
  ✅ Rename-safe — refactoring works correctly
  ✅ Tag Helpers work: asp-for="Name", asp-validation-for="Name"
  ✅ Self-documenting — anyone reading the view knows what data it uses
```

---

## 🔷 The `@model` Directive

```cshtml
@* Declares the type of data this view receives *@
@* Must be the first meaningful line in the view *@

@model Employee                    ← single object
@model List<Employee>              ← list of objects
@model EmployeeFormViewModel       ← ViewModel (recommended for complex forms)
@model IEnumerable<Employee>       ← any enumerable
@model Dictionary<string, int>     ← dictionary (rare)

@* Then access with capital M: *@
@Model.Name            ← property of the Employee
@Model.Count           ← Count of the List<Employee>
@Model.Employee.Name   ← nested ViewModel property
```

---

## 🔷 How Data Flows: Controller → View

```csharp
// Controller — passes the model:
public IActionResult Details(int id)
{
    Employee emp = _bal.GetById(id);      // ← get the data
    if (emp == null) return NotFound();
    return View(emp);                      // ← pass to View
}

// The emp object is passed as the Model argument
// View receives it as @Model
```

```cshtml
@* View — declares and uses the model *@
@model Employee        @* ← must match what controller passes *@

<h2>@Model.Name</h2>
<p>Role: @Model.Role</p>
<p>Salary: @Model.Salary.ToString("C0")</p>
```

---

## 🔷 Strongly Typed — Single Object (Details / Edit)

```cshtml
@* Views/Employee/Details.cshtml *@
@model Employee

@{
    ViewData["Title"] = $"Details — {Model.Name}";
}

<div class="container mt-4">
    <h2>@Model.Name</h2>

    <dl class="row">
        <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Name)</dt>
        <dd class="col-sm-9">@Html.DisplayFor(m => m.Name)</dd>

        <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Role)</dt>
        <dd class="col-sm-9">@Model.Role</dd>

        <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Salary)</dt>
        <dd class="col-sm-9">@Model.Salary.ToString("C0")</dd>

        <dt class="col-sm-3">@Html.DisplayNameFor(m => m.HireDate)</dt>
        <dd class="col-sm-9">@Model.HireDate.ToString("dd MMM yyyy")</dd>

        <dt class="col-sm-3">Status</dt>
        <dd class="col-sm-9">
            <span class="badge @(Model.IsActive ? "bg-success" : "bg-secondary")">
                @(Model.IsActive ? "Active" : "Inactive")
            </span>
        </dd>
    </dl>

    <a asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-warning">Edit</a>
    <a asp-action="Index" class="btn btn-secondary">Back to List</a>
</div>
```

---

## 🔷 Strongly Typed — List (Index)

```cshtml
@* Views/Employee/Index.cshtml *@
@model List<Employee>

@{
    ViewData["Title"] = "Employee List";
}

<div class="d-flex justify-content-between align-items-center mb-3">
    <h2>Employees <span class="badge bg-secondary">@Model.Count</span></h2>
    <a asp-action="Create" class="btn btn-primary">+ Add Employee</a>
</div>

@if (!Model.Any())
{
    <div class="alert alert-info">No employees found.</div>
}
else
{
    <table class="table table-striped table-hover">
        <thead class="table-dark">
            <tr>
                @* DisplayNameFor reads [Display(Name="...")] annotation *@
                <th>@Html.DisplayNameFor(m => m[0].Name)</th>
                <th>@Html.DisplayNameFor(m => m[0].Role)</th>
                <th>@Html.DisplayNameFor(m => m[0].Salary)</th>
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
                        <a asp-action="Details" asp-route-id="@emp.Id"
                           class="btn btn-sm btn-info">View</a>
                        <a asp-action="Edit" asp-route-id="@emp.Id"
                           class="btn btn-sm btn-warning">Edit</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}
```

---

## 🔷 Strongly Typed — Form (Create / Edit) with Tag Helpers

```cshtml
@* Views/Employee/Create.cshtml *@
@model Employee

@{
    ViewData["Title"] = "Add New Employee";
}

<div class="container mt-4">
    <h2>Add New Employee</h2>

    <form asp-action="Create" asp-controller="Employee" method="post">
        @Html.AntiForgeryToken()

        @* asp-for="Name" generates: id="Name", name="Name", value="@Model.Name" *@
        @* It also reads [Required], [StringLength] etc. for client validation *@
        @* Label reads [Display(Name="...")] annotation automatically *@

        <div class="mb-3">
            <label asp-for="Name" class="form-label"></label>
            <input asp-for="Name" class="form-control" />
            <span asp-validation-for="Name" class="text-danger small"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Role" class="form-label"></label>
            <input asp-for="Role" class="form-control" />
            <span asp-validation-for="Role" class="text-danger small"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Email" class="form-label"></label>
            <input asp-for="Email" type="email" class="form-control" />
            <span asp-validation-for="Email" class="text-danger small"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Salary" class="form-label"></label>
            <input asp-for="Salary" type="number" class="form-control" />
            <span asp-validation-for="Salary" class="text-danger small"></span>
        </div>

        <div class="mb-3">
            <label asp-for="HireDate" class="form-label"></label>
            <input asp-for="HireDate" type="date" class="form-control" />
        </div>

        <div class="mb-3 form-check">
            <input asp-for="IsActive" class="form-check-input" type="checkbox" />
            <label asp-for="IsActive" class="form-check-label"></label>
        </div>

        @* Summary of all errors *@
        <div asp-validation-summary="ModelOnly" class="text-danger mb-3"></div>

        <button type="submit" class="btn btn-success">Save Employee</button>
        <a asp-action="Index" class="btn btn-secondary ms-2">Cancel</a>
    </form>
</div>

@section Scripts {
    @{ await Html.RenderPartialAsync("_ValidationScripts"); }
}
```

---

## 🔷 ViewModel — Strongly Typed with Multiple Models

When a View needs data from more than one source:

```csharp
// Models/ViewModels/EmployeeFormViewModel.cs
public class EmployeeFormViewModel
{
    public Employee Employee { get; set; } = new();

    // Dropdown data for the form
    public List<SelectListItem> Departments { get; set; } = new();
    public List<SelectListItem> Roles       { get; set; } = new();
}
```

```csharp
// Controller — builds and returns ViewModel
public IActionResult Create()
{
    var vm = new EmployeeFormViewModel
    {
        Employee    = new Employee { IsActive = true },

        Departments = _bal.GetDepartments()
                          .Select(d => new SelectListItem
                          {
                              Value = d.Id.ToString(),
                              Text  = d.Name
                          }).ToList(),

        Roles = new List<SelectListItem>
        {
            new SelectListItem("Developer",   "Developer"),
            new SelectListItem("QA Engineer", "QA Engineer"),
            new SelectListItem("Manager",     "Manager"),
            new SelectListItem("DevOps",      "DevOps")
        }
    };

    return View(vm);
}

[HttpPost, ValidateAntiForgeryToken]
public IActionResult Create(EmployeeFormViewModel vm)
{
    if (!ModelState.IsValid)
    {
        // Re-populate dropdowns before returning
        vm.Departments = GetDeptSelectList();
        vm.Roles       = GetRolesSelectList();
        return View(vm);
    }

    string result = _bal.Insert(vm.Employee);

    if (result == "success")
    {
        TempData["Success"] = "Employee added!";
        return RedirectToAction(nameof(Index));
    }

    ModelState.AddModelError("", result);
    vm.Departments = GetDeptSelectList();
    return View(vm);
}
```

```cshtml
@* Views/Employee/Create.cshtml *@
@model EmployeeFormViewModel

<form asp-action="Create" method="post">
    @Html.AntiForgeryToken()

    @* Access employee properties via ViewModel *@
    <div class="mb-3">
        <label asp-for="Employee.Name" class="form-label"></label>
        <input asp-for="Employee.Name" class="form-control" />
        <span asp-validation-for="Employee.Name" class="text-danger small"></span>
    </div>

    @* Dropdown — populated from ViewModel.Departments *@
    <div class="mb-3">
        <label asp-for="Employee.DeptId" class="form-label"></label>
        <select asp-for="Employee.DeptId"
                asp-items="Model.Departments"
                class="form-select">
            <option value="">-- Select Department --</option>
        </select>
    </div>

    <div class="mb-3">
        <label asp-for="Employee.Role" class="form-label"></label>
        <select asp-for="Employee.Role"
                asp-items="Model.Roles"
                class="form-select">
            <option value="">-- Select Role --</option>
        </select>
    </div>

    <div class="mb-3">
        <label asp-for="Employee.Salary" class="form-label"></label>
        <input asp-for="Employee.Salary" type="number" class="form-control" />
        <span asp-validation-for="Employee.Salary" class="text-danger small"></span>
    </div>

    <button type="submit" class="btn btn-success">Save</button>
    <a asp-action="Index" class="btn btn-secondary">Cancel</a>
</form>

@section Scripts {
    @{ await Html.RenderPartialAsync("_ValidationScripts"); }
}
```

---

## 🔷 `Html.DisplayNameFor` and `Html.DisplayFor`

Strongly typed helpers that read your model's annotations:

```cshtml
@model List<Employee>

@* DisplayNameFor reads [Display(Name="Full Name")] from the model *@
<th>@Html.DisplayNameFor(m => m[0].Name)</th>
@* Renders: <th>Full Name</th>  (from [Display(Name="Full Name")]) *@

@* DisplayFor formats the value using [DisplayFormat] annotation *@
<td>@Html.DisplayFor(m => m.Salary)</td>
@* Renders formatted value per [DisplayFormat(DataFormatString="{0:C0}")] *@

@* Without annotations, renders the raw value *@
@* With [Display(Name="...")], renders that name *@
```

---

## 🔷 All Four Data-Passing Methods — Final Comparison

|                           | ViewData            | ViewBag         | TempData            | Strongly Typed               |
| ------------------------- | ------------------- | --------------- | ------------------- | ---------------------------- |
| Syntax                    | `ViewData["key"]` | `ViewBag.key` | `TempData["key"]` | `@model T`/`@Model.Prop` |
| Type safe                 | ❌                  | ❌              | ❌                  | ✅                           |
| IntelliSense              | ❌                  | ❌              | ❌                  | ✅                           |
| Compile error on typo     | ❌                  | ❌              | ❌                  | ✅                           |
| Tag Helpers (`asp-for`) | ❌                  | ❌              | ❌                  | ✅                           |
| Survives redirect         | ❌                  | ❌              | ✅                  | ❌                           |
| Best for                  | Layout title        | Quick extras    | Flash messages      | **Main page data**     |

> 📌  **Rule** : Always use strongly typed views for your main page data. Use ViewData for layout metadata (page title). Use TempData for flash messages after redirects.

---

## ⚠️ Common Strongly Typed View Mistakes

| Mistake                                              | What Happens                                                    | Fix                                                                         |
| ---------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `@model`type doesn't match what Controller passes  | `InvalidCastException`at runtime                              | Ensure `return View(empList)`matches `@model List<Employee>`            |
| Forgetting to re-populate dropdowns on POST failure  | ViewModel returned to View with null lists → crash             | Re-populate all dropdown lists before `return View(vm)`                   |
| Using `@model`without capital M in `@Model`      | Property reads fail                                             | `@model`(lowercase) declares type.`@Model`(uppercase) accesses the data |
| Using ViewBag for main content instead of `@model` | No IntelliSense, no compile-time safety, Tag Helpers don't work | Use strongly typed views for all main content                               |

---



## ⭐ Interview Quick-Fire

| Question                                               | Answer                                                                                                       |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| What is a strongly typed view?                         | A view with `@model TypeName`— gives IntelliSense, compile-time checking, and Tag Helper support          |
| What does `@model Employee`do?                       | Declares the view expects an `Employee`object from the Controller                                          |
| How is the model accessed in the view?                 | `@Model`(capital M) —`@Model.Name`,`@Model.Salary`etc.                                                |
| What advantage does `@model`give over `ViewBag`?   | Type safety, IntelliSense, compile errors on typos, Tag Helpers (`asp-for`) work                           |
| What is a ViewModel and when do you use it?            | A class combining multiple models or adding display-only data — used when a View needs more than one entity |
| What does `asp-for="Name"`require to work?           | `@model`must be declared — Tag Helpers need the model type                                                |
| What does `Html.DisplayNameFor(m => m.Name)`render?  | The `[Display(Name="...")]`annotation value — or the property name if no annotation                       |
| Which method is always recommended for main page data? | Strongly typed views —`@model`+`return View(model)`                                                     |
