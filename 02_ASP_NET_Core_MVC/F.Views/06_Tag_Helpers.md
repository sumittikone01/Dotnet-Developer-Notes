
# 06 — Tag Helpers

---

## 🎯 One-Line Definition

> **Tag Helpers are C#-backed HTML attributes and elements in Razor views that generate correct, context-aware HTML — replacing verbose `@Html.TextBoxFor(...)` helper calls with clean, readable HTML-like syntax while keeping server-side intelligence.**

---

## 🔷 HTML Helpers vs Tag Helpers — The Old vs New

```
OLD WAY — HTML Helpers (still works, but verbose):
──────────────────────────────────────────────────────────────
@Html.LabelFor(m => m.EmpName, "Employee Name", new { @class = "control-label" })
@Html.TextBoxFor(m => m.EmpName, new { @class = "form-control", placeholder = "Enter name" })
@Html.ValidationMessageFor(m => m.EmpName, "", new { @class = "text-danger" })
@Html.ActionLink("Edit", "Edit", new { id = emp.EmpId }, new { @class = "btn btn-primary" })

NEW WAY — Tag Helpers (looks like HTML, reads like HTML):
──────────────────────────────────────────────────────────────
<label asp-for="EmpName" class="control-label">Employee Name</label>
<input asp-for="EmpName" class="form-control" placeholder="Enter name" />
<span asp-validation-for="EmpName" class="text-danger"></span>
<a asp-action="Edit" asp-route-id="@emp.EmpId" class="btn btn-primary">Edit</a>

Same output HTML — Tag Helpers are just more readable.
```

---

## 🔷 Enabling Tag Helpers

```html
<!-- Views/_ViewImports.cshtml — add this ONCE, applies to ALL views -->
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
<!--           ↑ wildcard: include all tag helpers from this assembly -->

<!-- For your own custom tag helpers: -->
@addTagHelper *, EmployeeManagement
<!--           ↑ your project/assembly name -->

<!-- To remove a specific tag helper: -->
@removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.InputTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
```

---

## 🔷 Form Tag Helpers — Used Every Day

### `<form asp-action asp-controller>`

```html
<!-- Generates a form that POSTs to the correct action -->
<form asp-action="Create" asp-controller="Employee" method="post">
    @Html.AntiForgeryToken()
    <!-- form fields here -->
    <button type="submit">Save</button>
</form>

<!-- Generated HTML: -->
<form action="/Employee/Create" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8..." />
    <button type="submit">Save</button>
</form>

<!-- With route values: POST to /Employee/Edit/5 -->
<form asp-action="Edit" asp-controller="Employee"
      asp-route-id="@Model.EmpId" method="post">
</form>
```

### `<input asp-for>`

```html
@model Employee

<!-- Single most-used tag helper — generates input bound to model property -->
<input asp-for="EmpName" class="form-control" />

<!-- Generated HTML (reads from model + Data Annotations): -->
<input type="text"
       id="EmpName"
       name="EmpName"
       value="John Smith"
       class="form-control"
       data-val="true"
       data-val-required="Name is required" />
<!--   ↑ id, name, value, validation attrs all generated automatically -->

<!-- asp-for reads [DataType] to set input type: -->

@* [DataType(DataType.Password)] *@
<input asp-for="Password" />
<!-- → <input type="password" id="Password" name="Password" /> -->

@* [DataType(DataType.EmailAddress)] *@
<input asp-for="Email" />
<!-- → <input type="email" id="Email" name="Email" /> -->

@* [DataType(DataType.Date)] *@
<input asp-for="JoiningDate" />
<!-- → <input type="date" id="JoiningDate" name="JoiningDate" /> -->

@* decimal / int property *@
<input asp-for="Salary" />
<!-- → <input type="number" id="Salary" name="Salary" /> -->
```

### `<label asp-for>`

```html
<label asp-for="EmpName"></label>
<!-- Generated: <label for="EmpName">EmpName</label> -->

<!-- With [Display(Name = "Employee Name")] on the property: -->
<label asp-for="EmpName"></label>
<!-- Generated: <label for="EmpName">Employee Name</label> -->
<!--            ↑ reads Display attribute — no hardcoding label text -->
```

### `<span asp-validation-for>`

```html
<!-- Shows validation error message for a property -->
<span asp-validation-for="EmpName" class="text-danger"></span>

<!-- If EmpName fails [Required]: -->
<!-- Generated: <span class="text-danger">Name is required</span> -->

<!-- Requires these scripts for client-side validation: -->
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
    <!-- includes jquery.validate + jquery.validate.unobtrusive -->
}
```

### `<div asp-validation-summary>`

```html
<!-- Shows ALL validation errors at the top of the form -->
<div asp-validation-summary="ModelOnly" class="text-danger"></div>

<!--
    Values:
    "None"       → don't show anything
    "ModelOnly"  → show only model-level errors (not property errors)
    "All"        → show all errors (model + property)
-->
```

---

## 🔷 Full Form Example — The Real Pattern

```html
<!-- Views/Employee/Create.cshtml -->
@model Employee

<h2>Add New Employee</h2>

<form asp-action="Create" asp-controller="Employee" method="post">

    <!-- Summary of model-level errors -->
    <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>

    <div class="form-group">
        <label asp-for="EmpName" class="control-label"></label>
        <input asp-for="EmpName" class="form-control" placeholder="Full name" />
        <span asp-validation-for="EmpName" class="text-danger small"></span>
    </div>

    <div class="form-group">
        <label asp-for="Department" class="control-label"></label>
        <select asp-for="Department"
                asp-items="ViewBag.DepartmentList"
                class="form-control">
            <option value="">-- Select Department --</option>
        </select>
        <span asp-validation-for="Department" class="text-danger small"></span>
    </div>

    <div class="form-group">
        <label asp-for="Salary" class="control-label"></label>
        <input asp-for="Salary" class="form-control" />
        <span asp-validation-for="Salary" class="text-danger small"></span>
    </div>

    <div class="form-group">
        <label asp-for="JoiningDate" class="control-label"></label>
        <input asp-for="JoiningDate" class="form-control" />
        <span asp-validation-for="JoiningDate" class="text-danger small"></span>
    </div>

    <div class="form-check">
        <input asp-for="IsActive" class="form-check-input" />
        <label asp-for="IsActive" class="form-check-label"></label>
    </div>

    <button type="submit" class="btn btn-primary">Save Employee</button>
    <a asp-action="Index" class="btn btn-secondary">Cancel</a>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

```csharp
// Controller providing ViewBag.DepartmentList for the dropdown:
public IActionResult Create()
{
    ViewBag.DepartmentList = new SelectList(
        _deptBal.GetAll(), "DeptId", "DeptName");
    return View();
}
```

---

## 🔷 `<select asp-for asp-items>`

```html
<!-- Dropdown bound to model + populated from SelectList -->

@* In controller: *@
@* ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "DeptId", "DeptName"); *@

<select asp-for="DepartmentId"
        asp-items="ViewBag.DeptList"
        class="form-control">
    <option value="">-- Select --</option>
</select>
<!--
    Generated:
    <select id="DepartmentId" name="DepartmentId">
        <option value="">-- Select --</option>
        <option value="1">HR</option>
        <option value="2" selected>IT</option>   ← current value selected
        <option value="3">Finance</option>
    </select>
    ↑ If Model.DepartmentId = 2, "IT" is auto-selected
-->

<!-- From an enum: -->
<select asp-for="Status" asp-items="Html.GetEnumSelectList<EmployeeStatus>()">
    <option value="">-- Select Status --</option>
</select>

<!-- From hardcoded list: -->
<select asp-for="Gender" class="form-control">
    <option value="">-- Select --</option>
    <option value="M">Male</option>
    <option value="F">Female</option>
</select>
```

---

## 🔷 Anchor Tag Helpers — Navigation Links

```html
<!-- <a asp-action asp-controller asp-route-*> -->

<!-- Link to an action in the same controller: -->
<a asp-action="Index">Back to List</a>
<!-- → <a href="/Employee/Index">Back to List</a> -->

<!-- Link to action in different controller: -->
<a asp-action="Index" asp-controller="Home">Home</a>
<!-- → <a href="/Home/Index">Home</a> -->

<!-- Link with route parameter: -->
<a asp-action="Edit" asp-route-id="@emp.EmpId">Edit</a>
<!-- → <a href="/Employee/Edit/5">Edit</a> -->

<!-- Link with multiple route values: -->
<a asp-action="Report"
   asp-route-dept="HR"
   asp-route-year="2024">HR Report 2024</a>
<!-- → <a href="/Employee/Report?dept=HR&year=2024">HR Report 2024</a> -->

<!-- Link to area: -->
<a asp-area="Admin" asp-controller="Users" asp-action="Index">Admin</a>
<!-- → <a href="/Admin/Users/Index">Admin</a> -->
```

---

## 🔷 Environment Tag Helper

```html
<!-- Render content only in specific environments -->

<!-- Development only: -->
<environment include="Development">
    <script src="~/lib/jquery/jquery.js"></script>
    <!-- unminified for debugging -->
</environment>

<!-- Production only: -->
<environment exclude="Development">
    <script src="https://cdn.jquery.com/jquery.min.js"
            asp-fallback-src="~/lib/jquery/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
    <!-- minified CDN with local fallback -->
</environment>
```

---

## 🔷 Cache Tag Helper

```html
<!-- Cache rendered HTML output for a duration -->
<cache expires-after="@TimeSpan.FromMinutes(10)">
    <!-- This expensive HTML block is rendered once, cached for 10 min -->
    @await Component.InvokeAsync("DepartmentStats")
</cache>

<!-- Cache with vary-by (different cache per user): -->
<cache vary-by-user="true" expires-after="@TimeSpan.FromMinutes(5)">
    <vc:notification-bell></vc:notification-bell>
</cache>
```

---

## 🔷 Image Tag Helper — Cache Busting

```html
<!-- asp-append-version adds a hash to the URL for cache busting -->
<img src="~/images/logo.png" asp-append-version="true" alt="Logo" />

<!-- Generated: -->
<img src="/images/logo.png?v=GRA6f..." alt="Logo" />
<!--                        ↑ hash changes when file changes
         Browser re-downloads when file is updated — no stale cache -->
```

---

## 🔷 Link and Script Tag Helpers — CDN with Fallback

```html
<!-- <link asp-append-version> — CSS with cache busting -->
<link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
<!-- → /css/site.css?v=abc123 -->

<!-- CDN with local fallback: -->
<link rel="stylesheet"
      href="https://cdn.example.com/bootstrap.min.css"
      asp-fallback-href="~/lib/bootstrap/bootstrap.min.css"
      asp-fallback-test-class="sr-only"
      asp-fallback-test-property="position"
      asp-fallback-test-value="absolute"
      crossorigin="anonymous" />

<!-- Script with CDN fallback: -->
<script src="https://cdn.example.com/jquery.min.js"
        asp-fallback-src="~/lib/jquery/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
<!--
    Logic:
    1. Try loading from CDN
    2. Test if it loaded (window.jQuery exists?)
    3. If CDN failed → load from local wwwroot
-->
```

---

## 🔷 Custom Tag Helper — Building Your Own

```csharp
// TagHelpers/AlertTagHelper.cs
using Microsoft.AspNetCore.Razor.TagHelpers;

// HtmlTargetElement = which HTML element this applies to
[HtmlTargetElement("alert")]
public class AlertTagHelper : TagHelper
{
    // HTML attributes become properties (PascalCase ↔ kebab-case)
    public string Type    { get; set; } = "info";    // alert-type="warning"
    public string Message { get; set; }               // message="..."

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";   // replace <alert> with <div>
        output.Attributes.SetAttribute("class", $"alert alert-{Type}");
        output.Content.SetContent(Message);
    }
}
```

```html
<!-- Usage in view (after @addTagHelper *, EmployeeManagement): -->
<alert type="success" message="Employee created successfully!"></alert>

<!-- Generated: -->
<div class="alert alert-success">Employee created successfully!</div>
```

---

## 🔷 Most Used Tag Helpers — Quick Reference

```
TAG HELPER                          WHAT IT GENERATES
──────────────────────────────────────────────────────────────────────
<form asp-action asp-controller>    <form action="/Ctrl/Action" method="post">
<input asp-for="Prop">              <input id type name value data-val-* ...>
<label asp-for="Prop">              <label for="Prop">Display Name</label>
<span asp-validation-for="Prop">    Validation error message
<div asp-validation-summary>        All validation errors summary
<select asp-for asp-items>          <select> with options + auto-selected
<a asp-action asp-route-id>         <a href="/Controller/Action/5">
<environment include="Dev">         Conditionally renders in named environment
<cache expires-after>               Caches the rendered HTML
<img asp-append-version>            <img src="/img/logo.png?v=hash">
<link asp-append-version>           <link href="/css/site.css?v=hash">
<partial name="_PartialName">       Renders a partial view
<vc:component-name>                 Invokes a View Component
```

---

## ⭐ Interview Quick-Fire

| Question                                                   | Answer                                                                                                                 |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| What is a Tag Helper?                                      | A C#-backed attribute or element in Razor that generates correct HTML — cleaner alternative to `@Html.TextBoxFor()` |
| How do you enable Tag Helpers in a project?                | `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`in `_ViewImports.cshtml`                                     |
| What does `asp-for="EmpName"`do on an `<input>`?       | Generates `id`,`name`,`value`, and validation data attributes from the model property and its Data Annotations   |
| What does `asp-validation-for`do?                        | Renders the validation error message for that model property when `ModelState`has an error                           |
| What is `asp-items`used for?                             | Populates a `<select>`dropdown with options from a `SelectList`or `IEnumerable<SelectListItem>`                  |
| What does `asp-append-version="true"`do?                 | Appends a hash of the file content to the URL — forces browser to re-download when file changes                       |
| What is the Tag Helper equivalent of `@Html.ActionLink`? | `<a asp-action="Edit" asp-route-id="@emp.EmpId">Edit</a>`                                                            |
| What is the `<environment>`tag helper used for?          | Conditionally renders content based on the current environment (Development/Production)                                |
| What is the `<cache>`tag helper?                         | Caches the rendered HTML output for a specified duration — avoids re-rendering expensive content                      |
| Where must `@addTagHelper`be placed to affect all views? | In `Views/_ViewImports.cshtml`— applies to the whole project                                                        |
