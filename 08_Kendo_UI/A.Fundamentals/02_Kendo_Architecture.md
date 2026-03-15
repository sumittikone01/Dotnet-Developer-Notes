
# 02 — Kendo Architecture

---

## 🎯 One-Line Definition

> **Kendo's architecture is: a Widget on screen, powered by a DataSource for data, all styled by a Theme — and on the server side, Tag Helpers generate the HTML and `ToDataSourceResult` handles the data.**

---

## 🏛️ The Big Picture

Before going deep, see how all the pieces fit together:

```
┌──────────────────────────────────────────────────────────────────┐
│                        BROWSER                                    │
│                                                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   KENDO WIDGET                           │   │
│   │   (Grid, Chart, DropDownList, DatePicker...)             │   │
│   │                                                          │   │
│   │   Handles: rendering, user interaction, edit mode       │   │
│   │                                                          │   │
│   │              ▲ reads/writes data                        │   │
│   │              │                                          │   │
│   │   ┌──────────────────────────────┐                      │   │
│   │   │       KENDO DATASOURCE       │                      │   │
│   │   │  Handles: AJAX, paging,      │                      │   │
│   │   │  sorting, change tracking    │ ◄──► SERVER (AJAX)   │   │
│   │   └──────────────────────────────┘                      │   │
│   │                                                          │   │
│   │   Styled by: KENDO THEME (CSS)                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
                              │ HTTP + JSON
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                        SERVER                                     │
│                                                                   │
│   ASP.NET Core MVC Controller                                    │
│   [DataSourceRequest] ← reads paging/sort/filter from body      │
│   .ToDataSourceResult() ← queries DB, returns {Data,Total}      │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🧱 The 3 Client-Side Building Blocks

### Block 1 — The Widget

The widget is what the user  **sees and interacts with** .

```
Grid widget = the visible table on screen
  ┌────┬──────────┬────────────┬──────────┐
  │ ID │ Name     │ Department │ Salary   │
  ├────┼──────────┼────────────┼──────────┤
  │  1 │ Alice    │ IT         │ $75,000  │
  │  2 │ Bob      │ HR         │ $55,000  │
  └────┴──────────┴────────────┴──────────┘
  [ Add New ]              Page 1 of 5 ►

The widget handles:
  → What columns to show
  → Edit mode (inline / popup / incell)
  → Which buttons appear
  → How rows are rendered
  → User clicks, selections, scrolling
```

The widget does NOT fetch its own data. It delegates that to the DataSource.

---

### Block 2 — The DataSource

The DataSource is the widget's  **data engine** . It runs invisibly.

```
DataSource handles:
  → AJAX calls (read, create, update, destroy)
  → Tracking which rows are new / changed / deleted
  → Paging state (current page, total pages)
  → Sorting state (which column, which direction)
  → Filter state (active filters)
  → Local data cache (the rows currently loaded)
```

**One DataSource can power multiple widgets:**

```javascript
var sharedSource = new kendo.data.DataSource({
    transport: { read: { url: "/Employee/Read", type: "POST" } }
});

// Same data, two different widgets
$("#employeeGrid").kendoGrid({ dataSource: sharedSource });
$("#employeeChart").kendoChart({ dataSource: sharedSource });

// When sharedSource.read() is called → both Grid and Chart update
```

---

### Block 3 — The Theme (CSS)

The theme controls all visual styling — colours, borders, fonts, button styles.

```
Kendo ships with multiple built-in themes:

  Default     → clean blue/grey professional look
  Bootstrap   → matches Bootstrap 5 styling
  Material    → Google Material Design
  Fluent      → Microsoft Fluent / Office style
  Nova        → dark-friendly modern look
  Ocean Blue  → blue-toned professional
  ...and more
```

A theme is just one CSS file. Swap the CSS file → entire app re-skins.

```html
<!-- Default theme -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css"/>

<!-- Change to Bootstrap theme — swap the href, done -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/bootstrap/bootstrap-main.css"/>
```

---

## 🧱 The 2 Server-Side Building Blocks

### Block 4 — Tag Helpers (Razor → HTML)

Tag Helpers are **C# classes** that translate your Razor markup into the correct HTML + JavaScript that Kendo needs.

```
What you write in Razor:
────────────────────────────────────────────────────────────
<kendo-grid name="employeeGrid">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="/Employee/Read" type="POST"/>
        </transport>
    </datasource>
    <sortable enabled="true"/>
</kendo-grid>


What Tag Helpers generate in the browser (HTML output):
────────────────────────────────────────────────────────────
<div id="employeeGrid"></div>
<script>
  $("#employeeGrid").kendoGrid({
    dataSource: {
      transport: {
        read: { url: "/Employee/Read", type: "POST" }
      }
    },
    sortable: true
  });
</script>
```

You write clean Razor. Tag Helpers write the complex JavaScript for you.

---

### Block 5 — Server Extensions (`ToDataSourceResult`)

The server-side extension methods bridge your C# code with what Kendo expects.

```
[DataSourceRequest] DataSourceRequest request
    ↑
    Reads this from the POST body:
    page=1, pageSize=10, sort[0][field]=Name, sort[0][dir]=asc
    filter[filters][0][field]=Dept, operator=eq, value=IT


_db.Employees.AsQueryable().ToDataSourceResult(request)
    ↑
    Translates those params into efficient SQL:
    SELECT * FROM Employees
    WHERE Department = 'IT'
    ORDER BY Name ASC
    OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY

    Also runs: SELECT COUNT(*) WHERE Department = 'IT'  → Total = 12


Returns:
    { "Data": [ ...10 rows... ], "Total": 12, "Errors": null }
    ↑
    Exactly the JSON structure Kendo's DataSource expects
```

---

## 🔄 How All 5 Blocks Work Together — Full Flow

Take the moment a user opens the Employee Grid page and it loads data:

```
Step 1 — Page Load
  Browser requests /Employee/Index
  ASP.NET renders the Razor view
  Tag Helpers generate the Grid HTML + initialization JS
  Browser renders the empty grid shell

Step 2 — Grid Initialization
  Kendo JS reads the grid config
  Creates a Widget + a DataSource internally
  DataSource is told: "read from /Employee/Read via POST"

Step 3 — Initial Data Load
  DataSource sends AJAX POST to /Employee/Read
  Body: { page: 1, pageSize: 10 }

Step 4 — Server Processing
  [DataSourceRequest] reads page=1, pageSize=10
  .ToDataSourceResult() runs: SELECT TOP 10... COUNT(*)...
  Returns JSON: { Data: [...], Total: 47 }

Step 5 — Widget Renders
  DataSource receives JSON, stores 10 rows locally
  DataSource notifies Widget: "new data ready"
  Widget renders 10 rows in the table
  Widget renders pager: "Page 1 of 5"
  Theme CSS styles everything

Step 6 — User Interaction (e.g., clicks page 2)
  Widget tells DataSource: "user wants page 2"
  DataSource sends: POST /Employee/Read { page: 2, pageSize: 10 }
  Steps 4-5 repeat
  Widget shows page 2 rows
```

---

## 📦 How Kendo's JavaScript is Organized

Kendo JS comes in two forms:

```
┌──────────────────────────────────────────────────────────┐
│  kendo.all.min.js                                        │
│  → Contains ALL widgets                                  │
│  → Large file (~1MB)                                     │
│  → Simple: one script tag, everything available          │
│  → Use for development or when using many widgets        │
├──────────────────────────────────────────────────────────┤
│  Individual widget files                                 │
│  kendo.grid.min.js                                       │
│  kendo.datepicker.min.js                                 │
│  → Only what you need                                    │
│  → Smaller total download                               │
│  → Complex: need to include dependencies correctly       │
│  → Use for production optimization                       │
└──────────────────────────────────────────────────────────┘
```

For most ASP.NET MVC projects:  **use `kendo.all.min.js`** . Simple, reliable.

---

## 📜 Script Load Order — Strict Rule

```
CORRECT order — must be exactly this:
──────────────────────────────────────────────────────────
1.  jQuery                    (Kendo depends on jQuery)
2.  kendo.all.min.js          (all Kendo widgets)
3.  kendo.aspnetmvc.min.js    (ASP.NET MVC integration)

WRONG — Kendo before jQuery:
──────────────────────────────────────────────────────────
1.  kendo.all.min.js          ← tries to use jQuery, fails
2.  jQuery                    ← too late
Result: "$ is not defined" error in console, nothing works
```

```html
<!-- In _Layout.cshtml — this order is mandatory -->
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
<script src="https://kendo.cdn.telerik.com/.../kendo.all.min.js"></script>
<script src="https://kendo.cdn.telerik.com/.../kendo.aspnetmvc.min.js"></script>
```

---

## 🔍 Getting a Widget Reference in JavaScript

After Kendo renders a widget, you access it like this:

```javascript
// Pattern: $("#elementId").data("kendo[WidgetName]")

var grid   = $("#employeeGrid").data("kendoGrid");
var chart  = $("#salaryChart").data("kendoChart");
var ddl    = $("#deptDropDown").data("kendoDropDownList");
var window = $("#editWindow").data("kendoWindow");

// Then call methods on it
grid.dataSource.read();       // refresh data
chart.dataSource.read();      // refresh chart
ddl.value("IT");              // set selected value
window.open().center();       // show the window
```

This is the standard pattern for controlling any Kendo widget from JavaScript.

---

## 🗺️ Architecture Summary Table

| Layer  | Name                                   | Lives In    | Responsibility                       |
| ------ | -------------------------------------- | ----------- | ------------------------------------ |
| Client | Widget                                 | Browser JS  | Display, user interaction            |
| Client | DataSource                             | Browser JS  | AJAX, paging, change tracking        |
| Client | Theme                                  | Browser CSS | All visual styling                   |
| Server | Tag Helpers                            | C# / Razor  | Generate widget JS from Razor syntax |
| Server | DataSourceRequest + ToDataSourceResult | C#          | Translate paging/sort/filter ↔ SQL  |

---

## ❓ Interview Questions

**Q: What are the main architectural components of Kendo UI?**

> On the client: Widget (display and interaction), DataSource (data and AJAX), and Theme (styling). On the server: Tag Helpers (convert Razor to JavaScript) and `ToDataSourceResult` / `[DataSourceRequest]` (handle data queries).

**Q: What is the role of the DataSource in Kendo architecture?**

> It's the data layer that sits between the widget and the server. It handles all AJAX communication, tracks which rows are new/changed/deleted, manages paging and sorting state, and caches the current page of data. The widget never makes AJAX calls directly.

**Q: Why must jQuery load before Kendo JS?**

> Kendo is built on top of jQuery — it uses jQuery for DOM manipulation, event handling, and AJAX internally. Loading Kendo before jQuery causes a "$ is not defined" error and no widgets initialise.

**Q: What does `kendo.aspnetmvc.min.js` do?**

> It adds the ASP.NET MVC-specific integration — primarily the serialization format that makes Kendo's DataSource work correctly with `[DataSourceRequest]` and `ToDataSourceResult` on the server side.

**Q: Can one DataSource power multiple widgets?**

> Yes. You can create a DataSource instance and pass it to a Grid and a Chart simultaneously. When the DataSource refreshes, both widgets update with the same data.
>
