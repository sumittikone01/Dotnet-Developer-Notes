
# 01 — Razor View Engine

---

## 🎯 One-Line Definition

> **Razor is ASP.NET Core's view engine that lets you write HTML and C# in the same `.cshtml` file — the `@` symbol is the switch between HTML mode and C# mode, and the server processes the file and sends pure HTML to the browser.**

---

## 🔷 What Razor Does

```
YOU WRITE (.cshtml file on the server):
──────────────────────────────────────────────────────────────
<h2>Employee: @Model.Name</h2>
<p>Salary: @Model.Salary.ToString("C0")</p>

@if (Model.IsActive)
{
    <span class="badge bg-success">Active</span>
}

BROWSER RECEIVES (pure HTML — no C# visible):
──────────────────────────────────────────────────────────────
<h2>Employee: Alice Johnson</h2>
<p>Salary: ₹75,000</p>

<span class="badge bg-success">Active</span>

Razor runs on the SERVER — the browser never sees C# code.
```

---

## 🔷 The `.cshtml` File — What It Is

```
.cshtml = C# + HTML in one file

Extension breakdown:
  .cs   → contains C# code
  html  → contains HTML markup
  .cshtml → both together

Razor parser reads the file:
  @ symbol → switch to C# mode
  < symbol → switch to HTML mode

Example:
  <p>               ← HTML mode
  @Model.Name       ← C# mode (evaluates and inserts the value)
  </p>              ← HTML mode
```

---

## 🔷 Where Views Live — The Convention

```
Views/
├── Employee/                  ← folder = controller name
│   ├── Index.cshtml           ← action name
│   ├── Create.cshtml
│   ├── Edit.cshtml
│   └── Details.cshtml
├── Home/
│   ├── Index.cshtml
│   └── About.cshtml
└── Shared/                    ← shared across all controllers
    ├── _Layout.cshtml         ← master layout
    ├── _ValidationScripts.cshtml
    └── Error.cshtml

Convention: Views/{ControllerName}/{ActionName}.cshtml
When you write: return View();
Razor looks for: Views/[current controller]/[current action].cshtml
```

---

## 🔷 The Razor Processing Pipeline

```
REQUEST ARRIVES
       │
       ▼
Controller runs:
  var employees = _bal.GetAll();
  return View(employees);
       │
       │  passes List<Employee> to Razor
       ▼
Razor View Engine:
  1. Reads Views/Employee/Index.cshtml
  2. Compiles it to a C# class (once — cached)
  3. Executes it with the model data
  4. Produces pure HTML output
       │
       ▼
HTML SENT TO BROWSER
(No .cshtml, no C#, no @Model — just clean HTML)
```

---

## 🔷 Key Files in the Views Folder

### `_ViewImports.cshtml` — Global Imports

```cshtml
@* Views/_ViewImports.cshtml *@
@* Applied to ALL views in the folder and subfolders *@

@using EmployeeApp.Models               @* all model classes available *@
@using EmployeeApp.Models.ViewModels    @* viewmodels available *@
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers    @* core tag helpers *@
@addTagHelper *, Telerik.UI.for.AspNet.Core             @* Kendo tag helpers *@

@* Without this file, every view needs @using at the top *@
```

### `_ViewStart.cshtml` — Default Layout

```cshtml
@* Views/_ViewStart.cshtml *@
@* Runs BEFORE every view — sets the default layout *@

@{
    Layout = "_Layout";
    @* Every view uses _Layout.cshtml by default *@
    @* Override in a specific view: Layout = null; *@
}
```

---

## 🔷 Razor File Structure — What Goes Where

```cshtml
@* 1. MODEL DECLARATION — always first *@
@model List<Employee>

@* 2. VIEWDATA / PAGE TITLE *@
@{
    ViewData["Title"] = "Employee List";
}

@* 3. PAGE HTML *@
<div class="container">
    <h2>Employees</h2>

    @* 4. C# LOGIC — inline with HTML *@
    @foreach (var emp in Model)
    {
        <div class="card">
            <h5>@emp.Name</h5>
            <p>@emp.Role — @emp.Salary.ToString("C0")</p>
        </div>
    }
</div>

@* 5. SECTION SCRIPTS — appended to layout's @RenderSection("Scripts") *@
@section Scripts {
    <script>
        // page-specific JavaScript here
    </script>
}
```

---

## 🔷 How Razor Compiles Views

```
First request to a view:
  Razor reads the .cshtml file
  Compiles it to a C# class (GeneratedViews/...)
  Executes the class
  Returns HTML

Every subsequent request:
  Compiled class already in memory
  Executes directly — no re-compilation
  Very fast

In Production (published app):
  Views compiled at publish time (not at runtime)
  Even faster — zero compile cost on first request
  Set: <RazorCompileOnPublish>true</RazorCompileOnPublish>
```

---

## 🔷 Razor vs Other View Engines

|               | Razor              | Mustache       | Blade (Laravel) |
| ------------- | ------------------ | -------------- | --------------- |
| Language      | C#                 | JavaScript     | PHP             |
| Switch symbol | `@`              | `{{ }}`      | `{{ }}`/`@` |
| Compiled      | ✅ Yes — fast     | ❌ Interpreted | ❌ Interpreted  |
| IntelliSense  | ✅ Full            | ❌ Limited     | ❌ Limited      |
| Type safe     | ✅ With `@model` | ❌             | ❌              |
| Used in       | ASP.NET Core       | Node.js apps   | Laravel         |

---

## 🔷 Razor Output Encoding — Security Built In

```cshtml
@* Razor automatically HTML-encodes output to prevent XSS: *@

@* If Model.Name = "<script>alert('hack')</script>" *@

@Model.Name
@* Renders as: <script>alert('hack')</script> *@
@* Browser shows as text — script does NOT execute ✅ *@

@* To render raw HTML (only when YOU control the content): *@
@Html.Raw(Model.HtmlContent)
@* Renders as actual HTML — use with caution! *@
```

---

## ⭐ Interview Quick-Fire

| Question                                        | Answer                                                                          |
| ----------------------------------------------- | ------------------------------------------------------------------------------- |
| What is Razor?                                  | ASP.NET Core's view engine — mixes C# and HTML in `.cshtml`files using `@` |
| What extension do Razor files use?              | `.cshtml`                                                                     |
| Does the browser see Razor/C# code?             | ❌ No — server processes it, browser receives pure HTML                        |
| What does `@`symbol do in Razor?              | Switches from HTML mode to C# mode                                              |
| What is `_ViewImports.cshtml`?                | Applies `@using`and `@addTagHelper`to all views globally                    |
| What is `_ViewStart.cshtml`?                  | Runs before every view — sets the default `_Layout.cshtml`                   |
| How does Razor prevent XSS attacks?             | Automatically HTML-encodes all `@expression`output                            |
| Where does Razor look for a View by convention? | `Views/{ControllerName}/{ActionName}.cshtml`                                  |
