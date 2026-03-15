
# 01 — Introduction to Kendo UI

---

## 🎯 One-Line Definition

> **Kendo UI is a ready-made collection of professional UI widgets — grids, charts, dropdowns, date pickers — that plug into your ASP.NET Core MVC views and talk to your controllers automatically via AJAX.**

---

## 🤔 The Problem Kendo Solves

Picture what you'd need to build a data grid from scratch:

```
Building a data grid WITHOUT Kendo:
────────────────────────────────────────────────────────────
Write HTML table structure
Write AJAX to load data
Write JavaScript to render rows
Write sorting logic (click header → re-sort → re-render)
Write pagination logic (page 1, 2, 3... skip/take)
Write filtering logic (filter by column)
Write inline edit mode (click row → becomes form)
Write save/cancel buttons
Write validation display
Write delete with confirmation
Handle errors from server
Handle loading state / spinner
Make it look professional
...weeks of work
```

```
Building a data grid WITH Kendo:
────────────────────────────────────────────────────────────
<kendo-grid name="employeeGrid">
    <datasource ...>
        <read url="/Employee/Read" />
    </datasource>
    <sortable enabled="true" />
    <filterable enabled="true" />
    <pageable />
    <editable mode="popup" />
</kendo-grid>

Done. Professional. Tested. Works.
```

---

## 🧩 What Kendo UI Actually Is

Kendo UI is a **UI component library** made by  **Progress/Telerik** .

```
┌─────────────────────────────────────────────────────────────┐
│                    KENDO UI                                   │
│                                                               │
│  A collection of pre-built, professionally designed          │
│  UI components (called "widgets") that you drop into         │
│  your HTML pages.                                            │
│                                                               │
│  Each widget:                                                 │
│   • Has a visual interface (HTML + CSS)                      │
│   • Has JavaScript behaviour built in                         │
│   • Can connect to your server via AJAX                      │
│   • Works with ASP.NET Core via Tag Helpers or HTML Helpers  │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 What's Inside the Kendo UI Package

When you install Kendo UI for ASP.NET Core, you get:

| What                        | What It Contains                                 |
| --------------------------- | ------------------------------------------------ |
| **JavaScript files**  | All widget logic —`kendo.all.min.js`          |
| **CSS files**         | Themes and styling —`default-main.css`        |
| **C# Tag Helpers**    | `<kendo-grid>`,`<kendo-chart>`etc. in Razor  |
| **C# HTML Helpers**   | `Html.Kendo().Grid()`fluent syntax             |
| **Server extensions** | `ToDataSourceResult()`,`[DataSourceRequest]` |

---

## 🎛️ The Widget Library — What Kendo Gives You

Kendo has widgets for almost every UI need:

```
DATA ──────────────────────────────────────────────────────────
  Grid          → table with sort, filter, page, CRUD
  ListView      → custom-template list with paging
  TreeList      → hierarchical grid (parent-child rows)
  PivotGrid     → pivot table / cross-tab

CHARTS ────────────────────────────────────────────────────────
  Chart         → bar, line, pie, area, scatter, donut...
  Sparkline     → tiny inline charts
  StockChart    → financial OHLC candlestick charts
  Diagram       → flowcharts and org charts

FORMS ─────────────────────────────────────────────────────────
  TextBox       → styled text input
  NumericTextBox→ numbers with format (currency, percent)
  DatePicker    → calendar popup
  TimePicker    → time selector
  DropDownList  → styled select with search
  ComboBox      → dropdown + free text
  AutoComplete  → suggestions as you type
  MultiSelect   → pick multiple from a list
  Switch        → on/off toggle
  Slider        → drag to pick a value
  Upload        → file picker with drag-and-drop

NAVIGATION ────────────────────────────────────────────────────
  Menu          → horizontal/vertical navigation menu
  TreeView      → expandable tree structure
  TabStrip      → tabbed content panels
  PanelBar      → accordion-style panels

LAYOUT ────────────────────────────────────────────────────────
  Window        → floating popup panel
  Dialog        → modal confirmation / form
  Splitter      → resizable pane layout
  Notification  → toast messages (success, error, warning)
  Tooltip       → hover info box
  ProgressBar   → progress indicator

SCHEDULING ────────────────────────────────────────────────────
  Scheduler     → full calendar like Google Calendar
  Gantt         → project timeline chart
```

---

## 🔌 How Kendo Fits in Your ASP.NET MVC Project

```
┌─────────────────────────────────────────────────────────────────┐
│                   YOUR ASP.NET CORE MVC APP                      │
│                                                                   │
│  ┌─────────────────────────┐      ┌──────────────────────────┐  │
│  │   RAZOR VIEW (.cshtml)  │      │   CONTROLLER (.cs)       │  │
│  │                         │      │                          │  │
│  │  <kendo-grid>           │      │  [HttpPost]              │  │
│  │    <datasource>   AJAX  │◄────►│  public JsonResult       │  │
│  │      <read url=   JSON  │      │  Read([DataSourceRequest]│  │
│  │        "/Emp/Read"/>    │      │  { return Json(...); }   │  │
│  │    </datasource>        │      │                          │  │
│  │  </kendo-grid>          │      └──────────┬───────────────┘  │
│  └─────────────────────────┘                 │                   │
│                                              ▼                   │
│                                    ┌─────────────────┐          │
│                                    │  SQL SERVER DB   │          │
│                                    └─────────────────┘          │
└─────────────────────────────────────────────────────────────────┘

Flow:
1. Razor renders the Kendo widget HTML + JS on page load
2. Kendo widget makes AJAX call to your Controller
3. Controller queries DB, returns JSON
4. Kendo widget renders the data
5. User interacts → Kendo sends changes → Controller saves
```

---

## ✍️ Two Syntaxes — Tag Helper vs HTML Helper

Kendo gives you two ways to write widgets in Razor views. Both produce identical output.

### Tag Helper — looks like HTML ✅ Use This

```cshtml
<kendo-grid name="employeeGrid" height="400">
    <columns>
        <column field="Name" title="Employee Name" />
        <column field="Department" />
    </columns>
    <pageable />
    <sortable enabled="true" />
</kendo-grid>
```

### HTML Helper — C# fluent chain

```cshtml
@(Html.Kendo().Grid<Employee>()
    .Name("employeeGrid")
    .Height(400)
    .Columns(c => {
        c.Bound(e => e.Name).Title("Employee Name");
        c.Bound(e => e.Department);
    })
    .Pageable()
    .Sortable()
)
```

```
┌──────────────────┬────────────────────────────────────┐
│                  │ Tag Helper      │ HTML Helper       │
├──────────────────┼─────────────────┼───────────────────┤
│ Looks like       │ HTML            │ C# code           │
│ IntelliSense     │ Good            │ Good              │
│ Readability      │ Easier          │ Harder to scan    │
│ Recommended for  │ New projects ✅  │ Legacy code       │
└──────────────────┴─────────────────┴───────────────────┘
```

> **Stick with Tag Helpers.** This entire guide uses Tag Helpers.

---

## ⚙️ Minimum Setup — 4 Things Required

Before any Kendo widget works, these 4 things must be in place:

```
1. NuGet package installed
   Telerik.UI.for.AspNet.Core

2. Program.cs configured
   builder.Services.AddKendo();

3. _Layout.cshtml has scripts in correct order
   jQuery FIRST → then Kendo JS

4. _ViewImports.cshtml has Tag Helper registered
   @addTagHelper *, Telerik.UI.for.AspNet.Core
```

Miss any one of these → widget doesn't render, or renders broken.

---

## 🧭 Where Kendo Sits in Your Daily Work

```
You write:            Kendo handles:
─────────────         ──────────────────────────────────
Controller action  →  Grid calls it automatically via AJAX
Return Json(data)  →  Grid renders rows, handles paging
Model validation   →  Grid shows errors in edit popup
IQueryable query   →  .ToDataSourceResult() does sort/page/filter
```

You write the  **data layer** . Kendo handles the  **display layer** .

---

## ❓ Interview Questions

**Q: What is Kendo UI?**

> A commercial JavaScript UI component library by Progress/Telerik that provides ready-made widgets — grids, charts, forms, navigation — which integrate with ASP.NET Core MVC via Tag Helpers and communicate with controllers using AJAX and JSON.

**Q: What are the two ways to use Kendo widgets in a Razor view?**

> Tag Helpers (`<kendo-grid>`) which look like HTML, and HTML Helpers (`Html.Kendo().Grid()`) which use C# fluent syntax. Tag Helpers are the modern recommended approach.

**Q: What does `builder.Services.AddKendo()` do?**

> It registers the Kendo UI services with ASP.NET Core's dependency injection, enabling Tag Helper rendering and server-side extensions like `ToDataSourceResult()`.

**Q: What must come before Kendo JS in the layout file?**

> jQuery. Kendo depends on jQuery — if jQuery is missing or loads after Kendo, nothing works and the browser console shows errors.
>
