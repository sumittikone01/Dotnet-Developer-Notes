
# 03 — Layout Pages

---

## 🎯 One-Line Definition

> **A Layout Page is the master template shared by all your views — it contains the common HTML shell (nav, header, footer, CSS, JS) with a `@RenderBody()` placeholder where each page's unique content is injected.**

---

## 🔷 The Problem Layout Solves

```
WITHOUT layout (HTML copied in every view):
──────────────────────────────────────────────────────────
Index.cshtml    → full HTML: <head>, navbar, content, footer
Create.cshtml   → full HTML: <head>, navbar, content, footer
Edit.cshtml     → full HTML: <head>, navbar, content, footer
Details.cshtml  → full HTML: <head>, navbar, content, footer

Problems:
  ❌ Change the navbar → edit 50+ files
  ❌ Same Bootstrap CDN link copied everywhere
  ❌ Bug in the footer → fix in every file
  ❌ Inconsistent look if one file gets out of sync

WITH layout:
──────────────────────────────────────────────────────────
_Layout.cshtml  → full HTML shell: <head>, navbar, @RenderBody(), footer
Index.cshtml    → only the unique content for this page
Create.cshtml   → only the unique content for this page

Benefits:
  ✅ Change navbar → edit _Layout.cshtml ONLY
  ✅ Bootstrap CDN in one place
  ✅ Every page automatically looks consistent
  ✅ Views are small and focused
```

---

## 🔷 How Layout and View Combine

```
_Layout.cshtml:                    Index.cshtml:
─────────────────────────────      ─────────────────────────
<!DOCTYPE html>                    @{
<html>                                 Layout = "_Layout";
  <head>                           }
    Bootstrap CSS                  @model List<Employee>
    Site.css
  </head>
  <body>                           <h2>Employee List</h2>
    <nav>navbar...</nav>           <table>...</table>
                                   ← THIS content goes into
    @RenderBody() ←─────────────── @RenderBody() of Layout

    <footer>...</footer>
    jQuery + Bootstrap JS
  </body>
</html>

Final HTML sent to browser = Layout HTML with Index content inside it
```

---

## 🔷 Complete `_Layout.cshtml`

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    @* Page-specific title from ViewData *@
    <title>@ViewData["Title"] — EmployeeApp</title>

    @* Bootstrap CSS *@
    <link rel="stylesheet"
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" />

    @* Kendo UI CSS (if using Kendo) *@
    <link rel="stylesheet"
          href="https://kendo.cdn.telerik.com/themes/7.0.0/default/default-main.css" />

    @* Site-specific CSS *@
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>

    @* ── Navigation Bar ──────────────────────────────────────────── *@
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" asp-controller="Home" asp-action="Index">
                EmployeeApp
            </a>
            <div class="navbar-nav ms-auto">
                <a class="nav-link" asp-controller="Employee" asp-action="Index">
                    Employees
                </a>
                <a class="nav-link" asp-controller="Department" asp-action="Index">
                    Departments
                </a>
            </div>
        </div>
    </nav>

    @* ── Flash Messages ───────────────────────────────────────────── *@
    <div class="container mt-2">
        @if (TempData["Success"] != null)
        {
            <div class="alert alert-success alert-dismissible fade show">
                @TempData["Success"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }
        @if (TempData["Error"] != null)
        {
            <div class="alert alert-danger alert-dismissible fade show">
                @TempData["Error"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }
    </div>

    @* ── PAGE CONTENT — each view's body goes here ───────────────── *@
    <main class="container mt-4 mb-5">
        @RenderBody()
    </main>

    @* ── Footer ──────────────────────────────────────────────────── *@
    <footer class="bg-dark text-white text-center py-3 mt-5">
        <p class="mb-0">© @DateTime.Now.Year EmployeeApp. All rights reserved.</p>
    </footer>

    @* ── Scripts — jQuery + Bootstrap (always loaded) ────────────── *@
    <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>

    @* Kendo JS (if using Kendo) *@
    <script src="https://kendo.cdn.telerik.com/2024.1.130/js/kendo.all.min.js"></script>
    <script src="https://kendo.cdn.telerik.com/2024.1.130/js/kendo.aspnetmvc.min.js"></script>

    @* ── Page-specific scripts section ──────────────────────────── *@
    @RenderSection("Scripts", required: false)
    @*                                ↑ required:false = pages without scripts work fine *@
</body>
</html>
```

---

## 🔷 View That Uses the Layout

```cshtml
@* Views/Employee/Index.cshtml *@
@model List<Employee>

@{
    ViewData["Title"] = "Employee List";
    @* This sets the <title> tag in _Layout.cshtml *@
}

@* ONLY the unique content — layout provides everything else *@
<div class="d-flex justify-content-between mb-3">
    <h2>Employees (@Model.Count)</h2>
    <a asp-action="Create" class="btn btn-primary">+ Add Employee</a>
</div>

<table class="table table-striped">
    <thead class="table-dark">
        <tr>
            <th>Name</th><th>Role</th><th>Salary</th><th>Actions</th>
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
                    <a asp-action="Edit" asp-route-id="@emp.Id"
                       class="btn btn-sm btn-warning">Edit</a>
                </td>
            </tr>
        }
    </tbody>
</table>

@* Page-specific JS injected into layout's @RenderSection("Scripts") *@
@section Scripts {
    <script>
        $(function() {
            console.log("Employee list page loaded");
        });
    </script>
}
```

---

## 🔷 `@RenderBody()` — The Core Placeholder

```cshtml
@* In _Layout.cshtml — marks where each view's content goes *@
<main class="container mt-4">
    @RenderBody()
</main>

@* MUST appear exactly ONCE in a layout file *@
@* Every view's content replaces this call *@
```

---

## 🔷 `@RenderSection()` — Optional Content Areas

Sections let a view inject content into specific named spots in the layout:

```cshtml
@* In _Layout.cshtml — define a named section *@
<head>
    @* Always loaded CSS *@
    <link rel="stylesheet" href="bootstrap.css" />

    @* Optional per-page CSS *@
    @RenderSection("Styles", required: false)
</head>
<body>
    @RenderBody()

    @* Always loaded scripts *@
    <script src="jquery.min.js"></script>

    @* Optional per-page scripts *@
    @RenderSection("Scripts", required: false)
</body>
```

```cshtml
@* In a View — fill the sections *@
@section Styles {
    <link rel="stylesheet" href="~/css/employee.css" />
    @* Only loaded on this page *@
}

@section Scripts {
    <script src="~/js/employee-grid.js"></script>
    <script>
        $(function() {
            initGrid();
        });
    </script>
}

@* required: false → view without @section Scripts still works fine *@
@* required: true  → every view MUST define this section or error *@
```

---

## 🔷 Setting Layout — Three Ways

### Way 1 — `_ViewStart.cshtml` (automatic for all views — recommended)

```cshtml
@* Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
@* Every view in this folder uses _Layout.cshtml — nothing to set per view *@
```

### Way 2 — Per-view override

```cshtml
@* In a specific view — override the default layout *@
@{
    Layout = "_AdminLayout";   @* use a different layout *@
}
```

### Way 3 — Disable layout completely

```cshtml
@* In a specific view — no layout at all *@
@{
    Layout = null;   @* renders just this view's content — no HTML shell *@
}
@* Useful for: AJAX partial responses, email templates, print pages *@
```

---

## 🔷 Multiple Layouts — Different Areas

```
Common pattern for apps with different sections:
  _Layout.cshtml       → main site layout (navbar + footer)
  _AdminLayout.cshtml  → admin panel layout (sidebar + different nav)
  _PrintLayout.cshtml  → print layout (no nav, no footer)
  _EmailLayout.cshtml  → email template layout

Views/Admin/Dashboard.cshtml:
@{
    Layout = "_AdminLayout";  ← uses admin layout
}

Views/Employee/Print.cshtml:
@{
    Layout = "_PrintLayout";  ← uses print layout
}
```

---

## 🔷 Passing Data to the Layout — `ViewData`

```cshtml
@* In the view: set values *@
@{
    ViewData["Title"]     = "Employee Details";
    ViewData["Breadcrumb"] = "Home > Employees > Details";
}

@* In _Layout.cshtml: use those values *@
<title>@ViewData["Title"] — EmployeeApp</title>

<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li>@ViewData["Breadcrumb"]</li>
    </ol>
</nav>
```

---

## 🔷 Layout Structure Diagram

```
_Layout.cshtml (the shell)
┌─────────────────────────────────────────────────────┐
│  <!DOCTYPE html>                                    │
│  <head>                                             │
│    <title>@ViewData["Title"]</title>                │
│    Bootstrap CSS                                    │
│    @RenderSection("Styles", required: false) ←──┐  │
│  </head>                                        │  │
│  <body>                                         │  │
│    <nav>navbar</nav>                            │  │
│    Flash messages                               │  │
│                                                 │  │
│    ┌─────────────────────────────────────┐      │  │
│    │ @RenderBody()  ←─── view content   │      │  │
│    └─────────────────────────────────────┘      │  │
│                                                 │  │
│    <footer></footer>                            │  │
│    jQuery + Bootstrap JS                        │  │
│    @RenderSection("Scripts", false) ←────────┐ │  │
│  </body>                                      │ │  │
└───────────────────────────────────────────────┼─┼──┘
                                                │ │
Index.cshtml injects:                           │ │
  body content → @RenderBody()                 │ │
  @section Scripts { ... } ─────────────────────┘ │
  @section Styles  { ... } ───────────────────────┘
```

---

## ⚠️ Common Layout Mistakes

| Mistake                                          | What Happens                                      | Fix                                                                   |
| ------------------------------------------------ | ------------------------------------------------- | --------------------------------------------------------------------- |
| Missing `@RenderBody()`in layout               | View content never renders — blank page body     | Always include `@RenderBody()`in layout                             |
| `required: true`on `@RenderSection`          | Views without that section throw exception        | Use `required: false`for optional sections                          |
| Setting `Layout = null`in `_ViewStart`       | All views render without layout                   | Leave `_ViewStart`with `"_Layout"`, override per view when needed |
| `@RenderBody()`called twice                    | Exception — can only render body once            | Call it exactly once                                                  |
| Scripts in the view outside `@section Scripts` | Scripts above jQuery — not loaded in right order | Always put page JS in `@section Scripts`                            |

---

## ⭐ Interview Quick-Fire

| Question                                          | Answer                                                                           |
| ------------------------------------------------- | -------------------------------------------------------------------------------- |
| What is a Layout Page?                            | Master template with common HTML shell — navbar, CSS, JS — shared by all views |
| What does `@RenderBody()`do?                    | Placeholder where each view's unique content is injected                         |
| Where is the default layout set?                  | `Views/_ViewStart.cshtml`— sets `Layout = "_Layout"`                        |
| How to disable layout for one view?               | Set `@{ Layout = null; }`at top of the view                                    |
| What is `@RenderSection()`?                     | Named placeholder — views can inject content into specific spots in the layout  |
| What is `required: false`on `@RenderSection`? | Makes the section optional — views that don't define it still work              |
| How do views pass data to the layout?             | Through `ViewData`— e.g.,`ViewData["Title"]`set in view, read in layout     |
| How many times can you call `@RenderBody()`?    | Exactly once — calling twice throws an exception                                |
