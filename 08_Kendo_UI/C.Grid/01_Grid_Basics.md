# 01 — Grid Basics

---

## 🎯 One-Line Definition

> **The Kendo Grid is a powerful data table widget that displays database records with built-in sorting, filtering, paging, and full CRUD — all connected to your controller via AJAX, zero page reloads.**

---

## 🤔 What the Grid Does For You

Without Kendo Grid, building a data table with all these features would take weeks:

```
What you'd build manually:          What Kendo Grid gives you in minutes:
───────────────────────────────      ───────────────────────────────────────
HTML table structure                 ✅ Professional table UI
AJAX to load data                    ✅ Automatic AJAX data loading
Render rows with JavaScript          ✅ Automatic row rendering
Sort by column (click header)        ✅ Sortable columns
Filter by column value               ✅ Filter dropdowns per column
Pagination (page 1, 2, 3...)         ✅ Pager with page size options
Inline edit form                     ✅ Inline, popup, incell editing
Save changes via AJAX                ✅ Automatic CRUD AJAX calls
Validate input before save           ✅ Validation with error display
Export to Excel                      ✅ One attribute: excel file-name="..."
Resize columns                       ✅ Drag column borders
Reorder columns                      ✅ Drag column headers
Group by column                      ✅ Drag header to group area
Freeze columns while scrolling       ✅ locked="true" on column
```

---

## 🏗️ Grid Structure — What's Inside

```
┌──────────────────────────────────────────────────────────────┐
│  TOOLBAR   [ + Add New ]  [ Export Excel ]  [ 🔍 Search ]    │
├────┬───────────┬────────────┬──────────┬─────────────────────┤
│    │  Name   ▲ │ Department │ Salary   │ Actions             │  ← HEADER
│    │  ────── │ │ ────────── │ ──────── │ ───────────────────  │
├────┼───────────┼────────────┼──────────┼─────────────────────┤
│  1 │ Alice     │ IT         │ $75,000  │ [Edit] [Delete]     │  ← ROWS
│  2 │ Bob       │ HR         │ $55,000  │ [Edit] [Delete]     │
│  3 │ Carol     │ Finance    │ $60,000  │ [Edit] [Delete]     │
├────┴───────────┴────────────┴──────────┴─────────────────────┤
│  PAGER  ◄  1  2  3  4  5  ►   [10 ▼ items per page]  47 items│
└──────────────────────────────────────────────────────────────┘

Each visible part maps to a Grid section:
  Toolbar   → <toolbar>
  Header    → <columns> (each <column> defines one header cell)
  Rows      → rendered automatically from DataSource data
  Pager     → <pageable>
```

---

## 🔑 Minimum Working Grid — Tag Helper

```cshtml
@* Views/Employee/Index.cshtml *@

<kendo-grid name="employeeGrid" height="500">

    {{!-- COLUMNS — what to show and in what order --}}
    <columns>
        <column field="Id"         title="#"              width="60"  />
        <column field="Name"       title="Employee Name"  width="200" />
        <column field="Department" title="Department"     width="150" />
        <column field="Salary"     title="Salary"
                format="{0:C0}"    width="120" />
        <column field="HireDate"   title="Hire Date"
                format="{0:MM/dd/yyyy}" width="130" />
    </columns>

    {{!-- DATASOURCE — how to get the data --}}
    <datasource type="DataSourceTagHelperType.Ajax" page-size="10">
        <transport>
            <read url="@Url.Action("Read", "Employee")" type="POST" />
        </transport>
        <schema>
            <model id="Id" />
        </schema>
    </datasource>

    {{!-- FEATURES --}}
    <sortable enabled="true" />
    <filterable enabled="true" />
    <pageable button-count="5" refresh="true" page-sizes="true" />

</kendo-grid>
```

---

## 🔑 Minimum Working Controller

```csharp
public class EmployeeController : Controller
{
    private readonly AppDbContext _db;
    public EmployeeController(AppDbContext db) => _db = db;

    // Renders the page — no data, just the View
    public IActionResult Index() => View();

    // Grid calls this via AJAX to load data
    [HttpPost]
    public JsonResult Read([DataSourceRequest] DataSourceRequest request)
    {
        return Json(_db.Employees.ToDataSourceResult(request));
    }
}
```

---

## 🔑 Grid Initialisation — What Happens on Page Load

```
Step 1: Page loads
  Razor renders the <kendo-grid> Tag Helper
  Tag Helper generates JavaScript initialisation code

Step 2: Browser runs the JavaScript
  Grid widget is created on the <div id="employeeGrid"> element
  DataSource is configured with the Read URL

Step 3: Grid calls DataSource to load data
  DataSource sends POST to /Employee/Read
  Sends: { page: 1, pageSize: 10 }

Step 4: Controller responds
  Returns: { "Data": [...], "Total": 47 }

Step 5: Grid renders
  Draws table header from column definitions
  Renders one row per item in Data array
  Draws pager showing total pages
```

---

## 🔑 Getting a Reference to the Grid in JavaScript

After the grid is rendered, access it like this:

```javascript
// Standard pattern — used constantly
var grid = $("#employeeGrid").data("kendoGrid");

// Then call methods on it:
grid.dataSource.read();             // refresh all data
grid.dataSource.page(2);            // jump to page 2
grid.dataSource.total();            // total record count
grid.addRow();                      // open add-new row
grid.saveChanges();                 // save pending edits
grid.cancelChanges();               // discard pending edits
grid.editRow($("tr:eq(0)"));        // programmatically edit first row
grid.removeRow($("tr:eq(0)"));      // programmatically delete first row
```

---

## 🔑 Grid Configuration Options — Overview

```cshtml
<kendo-grid name="employeeGrid"
            height="500"
            resizable="true"
            reorderable="true"
            mobile="true">

    <columns> ... </columns>

    <toolbar>
        <toolbar-button name="create" />
        <toolbar-button name="excel"  />
        <toolbar-button name="search" />
    </toolbar>

    <datasource type="DataSourceTagHelperType.Ajax" page-size="10">
        ...
    </datasource>

    <editable mode="popup" />

    <sortable      enabled="true"  />
    <filterable    enabled="true"  />
    <groupable     enabled="true"  />
    <pageable      button-count="5" refresh="true" page-sizes="true" />
    <scrollable    enabled="true"  />
    <resizable     enabled="true"  />
    <reorderable   enabled="true"  />
    <column-menu   enabled="true"  />
    <selectable    mode="single"   />

    <excel file-name="employees.xlsx" all-pages="true" />
    <pdf   file-name="employees.pdf"                   />

</kendo-grid>
```

---

## 📊 Key Grid Attributes Quick Reference

| Attribute       | What It Does                  | Example                             |
| --------------- | ----------------------------- | ----------------------------------- |
| `name`        | Element ID — required        | `name="employeeGrid"`             |
| `height`      | Fixed height in px            | `height="500"`                    |
| `page-size`   | Rows per page (on datasource) | `page-size="10"`                  |
| `sortable`    | Click headers to sort         | `<sortable enabled="true" />`     |
| `filterable`  | Filter icon in columns        | `<filterable enabled="true" />`   |
| `pageable`    | Page navigation bar           | `<pageable />`                    |
| `groupable`   | Drag headers to group         | `<groupable enabled="true" />`    |
| `scrollable`  | Scrollable body               | `<scrollable enabled="true" />`   |
| `resizable`   | Drag column edges             | `<resizable enabled="true" />`    |
| `reorderable` | Drag columns sideways         | `<reorderable enabled="true" />`  |
| `editable`    | Edit mode                     | `<editable mode="popup" />`       |
| `selectable`  | Row selection                 | `<selectable mode="single" />`    |
| `excel`       | Excel export button           | `<excel file-name="data.xlsx" />` |

---

## ⚠️ Common Mistakes at Setup

| Mistake                                      | Symptom                          | Fix                                |
| -------------------------------------------- | -------------------------------- | ---------------------------------- |
| No `<schema><model id="Id" /></schema>`    | Edit/delete target wrong row     | Always set the model's primary key |
| Missing `using Kendo.Mvc.Extensions`       | `ToDataSourceResult`not found  | Add the using statement            |
| Missing `using Kendo.Mvc.UI`               | `[DataSourceRequest]`not found | Add the using statement            |
| No `builder.Services.AddKendo()`           | Tag helpers don't render         | Register Kendo in `Program.cs`   |
| Missing `@addTagHelper`in `_ViewImports` | `<kendo-grid>`is plain HTML    | Add the Kendo tag helper line      |
| Grid height not set                          | Grid expands infinitely          | Set `height="500"`or use CSS     |

---

## ❓ Interview Questions

**Q: What is the Kendo Grid?**

> A JavaScript widget that renders a data table with built-in sorting, filtering, paging, grouping, and CRUD editing. It uses a DataSource to communicate with an ASP.NET controller via AJAX — all without page reloads.

**Q: What is the minimum you need for a working read-only Kendo Grid?**

> A `<kendo-grid>` tag with `<columns>` defining what to show, a `<datasource>` with a `<read>` transport URL pointing to a controller action, and that controller action returning `Json(data.ToDataSourceResult(request))`.

**Q: How do you get a reference to a Kendo Grid in JavaScript?**

> `var grid = $("#elementId").data("kendoGrid")` — then call methods like `grid.dataSource.read()` to refresh data.

**Q: What does `<schema><model id="Id" /></schema>` do?**

> It tells the DataSource which field is the primary key. Without it, Kendo can't correctly identify which row to update or delete during CRUD operations.
>
