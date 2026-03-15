
# 02 — View Responsibilities

---

## 🎯 One-Line Definition

> **The View is a Razor template that receives a Model from the Controller and renders it as HTML — it contains only display logic, never business rules or database calls.**

---

## 🔷 What the View Is Responsible For

```
✅ VIEW OWNS:
  → Rendering HTML to the browser
  → Displaying Model data with Razor syntax
  → Showing validation error messages
  → Using Tag Helpers for forms, links, inputs
  → Conditional display (if/else based on data)
  → Looping through collections (foreach)
  → Including partial views and components
  → Applying CSS classes, Bootstrap layout

❌ VIEW NEVER DOES:
  → SQL queries or database calls
  → Business rule validation
  → Calling services, BAL, or DAL
  → Heavy C# logic — only display decisions
  → Creating or modifying data independently
```

---

## 🔷 How a View is Connected to a Controller

```
CONVENTION: Views/{ControllerName}/{ActionName}.cshtml

EmployeeController.Index()   →  Views/Employee/Index.cshtml
EmployeeController.Create()  →  Views/Employee/Create.cshtml
EmployeeController.Edit()    →  Views/Employee/Edit.cshtml
HomeController.Index()       →  Views/Home/Index.cshtml

This is automatic — return View() finds the right file.
return View("CustomName") overrides to a different file.
```

---

## 🔷 Razor — What It Is

```
Razor is the view engine in ASP.NET Core MVC.
It mixes C# code with HTML using the @ symbol.

  @  →  start a C# expression or statement
  @{} →  C# code block (multiple statements)
  @Model.Name  →  render a property value
  @foreach(...){}  →  loop over a collection
```

---

## 🔷 The `@model` Directive — Typing the View

```cshtml
@* Tell the view what type of data it receives *@

@model Employee                   ← single Employee object
@model List<Employee>             ← list of employees
@model EmployeeFormViewModel      ← ViewModel

@* Access it with capital M: *@
@Model.Name
@Model.Salary.ToString("C0")

@* Without @model, you have no IntelliSense and no type safety *@
@* Always declare @model at the top of every view *@
```

---

## 🔷 Read-Only View — Display Data

```cshtml
@* Views/Employee/Details.cshtml *@
@model Employee

<div class="container mt-4">
    <h2>Employee Details</h2>

    <dl class="row">
        <dt class="col-sm-3">
            @Html.DisplayNameFor(m => m.Name)
        </dt>
        <dd class="col-sm-9">
            @Model.Name
        </dd>

        <dt class="col-sm-3">
            @Html.DisplayNameFor(m => m.Salary)
        </dt>
        <dd class="col-sm-9">
            @Model.Salary.ToString("C0")
        </dd>

        <dt class="col-sm-3">Hire Date</dt>
        <dd class="col-sm-9">
            @Model.HireDate.ToString("dd MMM yyyy")
        </dd>

        <dt class="col-sm-3">Status</dt>
        <dd class="col-sm-9">
            @if (Model.IsActive)
            {
                <span class="badge bg-success">Active</span>
            }
            else
            {
                <span class="badge bg-secondary">Inactive</span>
            }
        </dd>
    </dl>

    <a asp-action="Edit" asp-route-id="@Model.Id"
       class="btn btn-warning">Edit</a>
    <a asp-action="Index" class="btn btn-secondary">Back to List</a>
</div>
```

---

## 🔷 List View — Loop and Display Collection

```cshtml
@* Views/Employee/Index.cshtml *@
@model List<Employee>

<div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2>Employees (@Model.Count)</h2>
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
                    <th>Name</th>
                    <th>Role</th>
                    <th>Email</th>
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
                        <td>@emp.Email</td>
                        <td>@emp.Salary.ToString("C0")</td>
                        <td>
                            <a asp-action="Edit"
                               asp-route-id="@emp.Id"
                               class="btn btn-sm btn-warning">Edit</a>

                            <a asp-action="Details"
                               asp-route-id="@emp.Id"
                               class="btn btn-sm btn-info">View</a>

                            <form asp-action="Delete"
                                  asp-route-id="@emp.Id"
                                  method="post"
                                  style="display:inline"
                                  onsubmit="return confirm('Delete @emp.Name?')">
                                @Html.AntiForgeryToken()
                                <button type="submit"
                                        class="btn btn-sm btn-danger">Delete</button>
                            </form>
                        </td>
                    </tr>
                }
            </tbody>
        </table>
    }
</div>
```

---

## 🔷 Form View — Create / Edit

```cshtml
@* Views/Employee/Create.cshtml *@
@model Employee

<div class="container mt-4">
    <h2>Add New Employee</h2>

    @* asp-action + asp-controller = generates action URL *@
    <form asp-action="Create" asp-controller="Employee" method="post">

        @Html.AntiForgeryToken()   @* CSRF protection *@

        <div class="mb-3">
            @* asp-for generates id, name, value attributes AND links to validation *@
            <label asp-for="Name" class="form-label"></label>
            <input asp-for="Name" class="form-control" placeholder="Enter full name" />
            <span asp-validation-for="Name" class="text-danger"></span>
            @*       ↑ shows error message if Name validation fails *@
        </div>

        <div class="mb-3">
            <label asp-for="Role" class="form-label"></label>
            <input asp-for="Role" class="form-control" />
            <span asp-validation-for="Role" class="text-danger"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Email" class="form-label"></label>
            <input asp-for="Email" type="email" class="form-control" />
            <span asp-validation-for="Email" class="text-danger"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Salary" class="form-label"></label>
            <input asp-for="Salary" type="number" class="form-control" />
            <span asp-validation-for="Salary" class="text-danger"></span>
        </div>

        <div class="mb-3">
            <label asp-for="HireDate" class="form-label"></label>
            <input asp-for="HireDate" type="date" class="form-control" />
            <span asp-validation-for="HireDate" class="text-danger"></span>
        </div>

        @* Summary of ALL validation errors at once *@
        <div asp-validation-summary="ModelOnly" class="text-danger mb-3"></div>

        <button type="submit" class="btn btn-success">Save Employee</button>
        <a asp-action="Index" class="btn btn-secondary">Cancel</a>
    </form>
</div>

@* Include validation scripts at the bottom *@
@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScripts");}
}
```

---

## 🔷 Key Tag Helpers in Views

| Tag Helper                    | What It Generates                     | Example                                      |
| ----------------------------- | ------------------------------------- | -------------------------------------------- |
| `asp-for="Name"`            | `id="Name" name="Name" value="..."` | On `<input>`,`<label>`,`<span>`        |
| `asp-validation-for="Name"` | Validation error message span         | `<span asp-validation-for="Name">`         |
| `asp-validation-summary`    | All errors at once                    | `<div asp-validation-summary="ModelOnly">` |
| `asp-action="Create"`       | `/Employee/Create`URL               | On `<form>`or `<a>`                      |
| `asp-controller="Employee"` | Target controller                     | On `<form>`or `<a>`                      |
| `asp-route-id="@emp.Id"`    | `/Employee/Edit/5`                  | On `<a>`                                   |
| `asp-items="Model.Depts"`   | Dropdown options                      | On `<select>`                              |

---

## 🔷 Displaying Success and Error Messages from TempData

```cshtml
@* In _Layout.cshtml or at top of a view *@

@if (TempData["Success"] != null)
{
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        @TempData["Success"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

@if (TempData["Error"] != null)
{
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        @TempData["Error"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}
```

---

## 🔷 View with ViewModel — Dropdowns

```cshtml
@* Views/Employee/Create.cshtml *@
@model EmployeeFormViewModel

<form asp-action="Create" method="post">
    @Html.AntiForgeryToken()

    <div class="mb-3">
        <label asp-for="Employee.Name" class="form-label"></label>
        <input asp-for="Employee.Name" class="form-control" />
        <span asp-validation-for="Employee.Name" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Employee.DeptId" class="form-label"></label>
        <select asp-for="Employee.DeptId"
                asp-items="Model.Departments"
                class="form-select">
            <option value="">-- Select Department --</option>
        </select>
        <span asp-validation-for="Employee.DeptId" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-success">Save</button>
</form>
```

---

## 🔷 `@Html` vs Tag Helpers — Which to Use

```cshtml
@* OLD WAY — HTML Helpers *@
@Html.LabelFor(m => m.Name)
@Html.TextBoxFor(m => m.Name, new { @class = "form-control" })
@Html.ValidationMessageFor(m => m.Name)
@Html.ActionLink("Edit", "Edit", new { id = emp.Id })

@* MODERN WAY — Tag Helpers (preferred in ASP.NET Core) *@
<label asp-for="Name" class="form-label"></label>
<input asp-for="Name" class="form-control" />
<span asp-validation-for="Name" class="text-danger"></span>
<a asp-action="Edit" asp-route-id="@emp.Id">Edit</a>

Tag Helpers are preferred because:
  ✅ Look like standard HTML — designers can read them
  ✅ Full IntelliSense support
  ✅ Easier to add CSS classes
  ✅ Less @{ } C# noise
```

---

## 🔷 `_ViewImports.cshtml` — Global Imports for All Views

```cshtml
@* Views/_ViewImports.cshtml *@
@* This file applies to ALL views in the folder *@

@using EmployeeApp.Models          @* Makes Employee, Department etc. available *@
@using EmployeeApp.Models.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers   @* enables all Tag Helpers *@
@addTagHelper *, Telerik.UI.for.AspNet.Core            @* enables Kendo Tag Helpers *@
```

---

## ⚠️ Common View Mistakes

| Mistake                                       | What Happens                             | Fix                                                      |
| --------------------------------------------- | ---------------------------------------- | -------------------------------------------------------- |
| No `@model`declaration                      | No IntelliSense,`@Model`is `dynamic` | Always declare `@model Type`at top                     |
| Putting heavy logic in View                   | View becomes hard to read and test       | Move logic to Controller or BAL                          |
| Using `@Html.TextBoxFor`in new projects     | Old verbose syntax                       | Use Tag Helpers (`asp-for`)                            |
| Forgetting `@Html.AntiForgeryToken()`       | POST requests rejected (400)             | Add to every form, or use `[ValidateAntiForgeryToken]` |
| Not adding `@section Scripts`for validation | Client-side validation doesn't work      | Include `_ValidationScripts`partial in Scripts section |

---

## ⭐ Interview Quick-Fire

| Question                                        | Answer                                                              |
| ----------------------------------------------- | ------------------------------------------------------------------- |
| What is a View responsible for?                 | Rendering HTML — displaying Model data to the user                 |
| What file extension do Razor Views use?         | `.cshtml`— C# + HTML                                             |
| How is a View connected to a Controller action? | Convention:`Views/{ControllerName}/{ActionName}.cshtml`           |
| What does `@model Employee`do?                | Declares the type of data the View receives — enables IntelliSense |
| What does `asp-for="Name"`generate?           | `id="Name" name="Name" value="..."`— bound to the model property |
| What does `asp-validation-for`show?           | The validation error message for that specific field                |
| What is `_ViewImports.cshtml`?                | Global file — applies `@using`and `@addTagHelper`to all views  |
| Tag Helpers vs HTML Helpers?                    | Tag Helpers are preferred — look like HTML, better IntelliSense    |
