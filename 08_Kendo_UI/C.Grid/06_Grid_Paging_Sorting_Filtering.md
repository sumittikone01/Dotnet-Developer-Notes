
# 06 — Grid Paging, Sorting & Filtering

---

## 🎯 One-Line Definition

> **Paging, sorting, and filtering let users navigate large datasets — click a header to sort, use filter icons to narrow results, and use the pager to move between pages — all handled server-side so only the needed rows are loaded.**

---

## 🔑 What Each Feature Does

```
┌─────────────────────────────────────────────────────────────────┐
│  SORTING  — click a column header to sort by that column        │
│  ──────────────────────────────────────────────────────────     │
│  Name ▲  → sorted A-Z                                          │
│  Name ▼  → sorted Z-A                                          │
│  Name    → unsorted (click 3rd time)                           │
├─────────────────────────────────────────────────────────────────┤
│  FILTERING  — narrow visible rows by column values              │
│  ──────────────────────────────────────────────────────────     │
│  Department = IT  → only IT employees shown                    │
│  Salary > 60000   → only high earners shown                    │
│  Name contains "Al" → Alice, Albert, Alicia...                 │
├─────────────────────────────────────────────────────────────────┤
│  PAGING  — split large data into pages                          │
│  ──────────────────────────────────────────────────────────     │
│  ◄  1  2  [3]  4  5  ►   10 items/page   47 total items       │
│                                                                  │
│  Shows 10 rows at a time — user navigates to load others       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Enabling All Three — The Complete Setup

```cshtml
<kendo-grid name="employeeGrid" height="500">

    <columns>
        <column field="Id"         title="#"         width="60"  />
        <column field="Name"       title="Full Name" width="220" />
        <column field="Department" title="Department" width="150" />
        <column field="Salary"     title="Salary"    width="130"
                format="{0:C0}" />
        <column field="HireDate"   title="Hire Date" width="130"
                format="{0:MM/dd/yyyy}" />
        <column field="IsActive"   title="Active"    width="80"  />
    </columns>

    {{!-- SORTING --}}
    <sortable enabled="true" />

    {{!-- FILTERING --}}
    <filterable enabled="true" />

    {{!-- PAGING --}}
    <pageable button-count="5"
              refresh="true"
              page-sizes="true" />
    {{!-- page-sizes="true" shows dropdown: 5, 10, 20, 50 per page --}}

    {{!-- Search toolbar (optional) --}}
    <toolbar>
        <toolbar-button name="search" />
    </toolbar>

    {{!-- DataSource MUST have serverPaging, serverSorting, serverFiltering --}}
    <datasource type="DataSourceTagHelperType.Ajax"
                page-size="10"
                server-paging="true"
                server-sorting="true"
                server-filtering="true">
        <transport>
            <read url="@Url.Action("Read", "Employee")" type="POST" />
        </transport>
        <schema>
            <model id="Id" />
        </schema>
    </datasource>

</kendo-grid>
```

---

## 🔑 Paging — Deep Dive

### The pager bar explained

```
◄  1  2  [3]  4  5  ►       Refresh ↺      Items per page: [10 ▼]    47 items
│  │                  │         │                               │          │
│  Page buttons        │         │                               │          │
│                      │         │                               │          └ total records
│  ◄ = go to first     │         └ refresh button                └ page size dropdown
│  ► = go to last      │
│  [3] = current page  │
└──────────────────────┘
```

### Pager configuration options

```cshtml
<pageable
    button-count="5"
    refresh="true"
    page-sizes="true"
    previous-next="true"
    numeric="true"
    info="true"
    messages-of="of"
    messages-items-per-page="items per page" />
```

| Option            | What It Does                  | Default |
| ----------------- | ----------------------------- | ------- |
| `button-count`  | How many page number buttons  | 10      |
| `refresh`       | Show refresh button           | false   |
| `page-sizes`    | Show items-per-page dropdown  | false   |
| `previous-next` | Show ◄ ► first/last buttons | true    |
| `numeric`       | Show page number buttons      | true    |
| `info`          | Show "X - Y of Z items"       | true    |

### Controlling page size from JavaScript

```javascript
var grid = $("#employeeGrid").data("kendoGrid");

grid.dataSource.pageSize(20);    // change to 20 per page
grid.dataSource.page(1);         // jump to page 1
grid.dataSource.page();          // get current page number
grid.dataSource.totalPages();    // total number of pages
grid.dataSource.total();         // total records
```

---

## 🔑 Sorting — Deep Dive

### Single-column sort (default)

```cshtml
<sortable enabled="true" />
{{!-- Click column header once → ascending ▲ --}}
{{!-- Click again             → descending ▼ --}}
{{!-- Click third time        → unsorted     --}}
```

### Multi-column sort (hold Shift + click)

```cshtml
<sortable enabled="true" mode="multiple" allow-unsort="true" />
{{!-- Shift+click second column → sorts by first column then second --}}
{{!-- Shows: Name ▲  Salary ▼ --}}
```

### Disable sort on specific columns

```cshtml
<columns>
    <column field="Name" />                    {{!-- sortable (default) --}}
    <column field="Notes" sortable="false" />  {{!-- NOT sortable --}}
    <column field="Photo" sortable="false" />  {{!-- NOT sortable --}}
</columns>
```

### Set initial sort from code

```javascript
// Set the initial sort when the grid loads
var grid = $("#employeeGrid").data("kendoGrid");
grid.dataSource.sort({ field: "Name", dir: "asc" });
```

---

## 🔑 Filtering — Deep Dive

### Filter UI modes

```cshtml
{{!-- Mode 1: Filter row (always visible below column header) --}}
<filterable enabled="true" mode="row" />

{{!-- Mode 2: Filter menu (click funnel icon in header — default) --}}
<filterable enabled="true" mode="menu" />

{{!-- Mode 3: Both --}}
<filterable enabled="true" mode="row, menu" />
```

**Row mode:**

```
┌────────────────┬─────────────────┬──────────────────┐
│ Name     ▲     │ Department      │ Salary           │  ← sortable header
├────────────────┼─────────────────┼──────────────────┤
│ [Search...   ] │ [____________▼] │ [_____] [eq ▼]   │  ← filter row
├────────────────┼─────────────────┼──────────────────┤
│ Alice          │ IT              │ $75,000          │
```

**Menu mode (default):**

```
┌────────────────┬─────────────────┐
│ Name     ▲  🔻 │ Department   🔻 │  ← 🔻 = filter icon, click to open
└────────────────┴─────────────────┘
                  ┌──────────────────────────┐
                  │ Filter                   │
                  │ Show rows where:         │
                  │ Department [Is equal to] │
                  │           [IT          ] │
                  │  [And ▼]                 │
                  │ Department [Is equal to] │
                  │           [            ] │
                  │  [Filter] [Clear]        │
                  └──────────────────────────┘
```

### Per-column filter configuration

```cshtml
<columns>

    {{!-- Default filter (text: contains, startswith, etc.) --}}
    <column field="Name" />

    {{!-- Only allow specific operators --}}
    <column field="Department">
        <filterable>
            <messages operator="Filter by:" />
            <operators>
                <column-string-operators eq="Is" neq="Is not" />
            </operators>
        </filterable>
    </column>

    {{!-- Number range filter --}}
    <column field="Salary" format="{0:C0}">
        <filterable>
            <operators>
                <column-number-operators gte="Greater than" lte="Less than" />
            </operators>
        </filterable>
    </column>

    {{!-- Date range filter --}}
    <column field="HireDate" format="{0:MM/dd/yyyy}">
        <filterable>
            <operators>
                <column-date-operators gte="From" lte="To" />
            </operators>
        </filterable>
    </column>

    {{!-- Disable filter on this column --}}
    <column field="Notes" filterable="false" />

</columns>
```

### Filter operators by type

| Data Type         | Available Operators                                                                                          |
| ----------------- | ------------------------------------------------------------------------------------------------------------ |
| **String**  | Contains, Does not contain, Starts with, Ends with, Is equal to, Is not equal to, Is empty, Is not empty     |
| **Number**  | Is equal to, Is not equal to, Greater than, Less than, Greater or equal, Less or equal, Is null, Is not null |
| **Date**    | Is equal to, Is not equal to, Is after, Is after or equal to, Is before, Is before or equal to, Is null      |
| **Boolean** | Is true, Is false                                                                                            |

---

## 🔑 The Controller — Handling All Three

```csharp
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    // ToDataSourceResult applies ALL of:
    //   paging    → SKIP / TAKE
    //   sorting   → ORDER BY
    //   filtering → WHERE
    // automatically from the request object

    var result = _db.Employees
                    .AsQueryable()
                    .ToDataSourceResult(request);

    return Json(result);
}
```

**That's it. One line handles everything.**
The `request` object contains page number, page size, sort fields, filter conditions — all populated automatically by `[DataSourceRequest]`.

### Adding your own filter on top

```csharp
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    // Your mandatory filter first
    var query = _db.Employees
                   .Where(e => e.CompanyId == GetCurrentCompanyId())
                   .Where(e => !e.IsDeleted);

    // Then let the grid's paging/sorting/filtering apply on top
    return Json(query.ToDataSourceResult(request));
}
```

---

## 🔑 Setting Filters Programmatically from JavaScript

```javascript
var grid = $("#employeeGrid").data("kendoGrid");
var ds   = grid.dataSource;

// Filter to show only IT department
ds.filter({
    field:    "Department",
    operator: "eq",
    value:    "IT"
});

// Filter with multiple conditions (AND)
ds.filter({
    logic: "and",
    filters: [
        { field: "Department", operator: "eq",  value: "IT"    },
        { field: "Salary",     operator: "gte", value: 60000   }
    ]
});

// Filter with multiple conditions (OR)
ds.filter({
    logic: "or",
    filters: [
        { field: "Department", operator: "eq", value: "IT" },
        { field: "Department", operator: "eq", value: "HR" }
    ]
});

// Clear all filters
ds.filter({});

// Read current active filters
var currentFilters = ds.filter();
console.log(currentFilters);
```

---

## 🔑 The Toolbar Search Box

The built-in search box filters across all string columns automatically:

```cshtml
<toolbar>
    <toolbar-button name="search" />
</toolbar>
```

```
[🔍 Search employees...              ]
           ↓ as user types
Grid filters by Name, Department, Email — all string fields
```

Customise which columns the search covers:

```cshtml
<toolbar>
    <toolbar-button name="search" />
</toolbar>

<search fields="Name, Email" />
{{!-- Only searches in Name and Email columns --}}
```

---

## 🔑 Grouping — Bonus Feature

Drag a column header into the group area to group rows:

```cshtml
<groupable enabled="true" />
```

```
Drag "Department" header to group bar:

  Department: Finance ▼
  ┌────┬───────┬─────────┬──────────┐
  │ 3  │ Carol │ Finance │ $60,000  │
  └────┴───────┴─────────┴──────────┘
  Department: HR ▼
  ┌────┬───────┬─────────┬──────────┐
  │ 2  │ Bob   │ HR      │ $55,000  │
  └────┴───────┴─────────┴──────────┘
  Department: IT ▼
  ┌────┬───────┬─────────┬──────────┐
  │ 1  │ Alice │ IT      │ $75,000  │
  └────┴───────┴─────────┴──────────┘
```

---

## 📊 Feature Configuration Quick Reference

```
Feature          Tag Helper                              JS to control
───────────────  ──────────────────────────────────────  ────────────────────────────
Paging           <pageable />                            ds.page(2), ds.pageSize(20)
Sorting          <sortable enabled="true" />             ds.sort({field,dir})
Multi-sort       <sortable mode="multiple" />            ds.sort([{...},{...}])
Filtering        <filterable enabled="true" />           ds.filter({field,op,val})
Filter row       <filterable mode="row" />               ds.filter({})  ← clear
Grouping         <groupable enabled="true" />            ds.group({field:"Dept"})
Search box       <toolbar-button name="search" />        — built-in
Refresh button   <pageable refresh="true" />             ds.read()
Page sizes       <pageable page-sizes="true" />          ds.pageSize(n)
```

---

## ⚠️ Common Mistakes

| Mistake                                                           | Symptom                                                      | Fix                                                             |
| ----------------------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------- |
| `server-paging="true"`but no `schema.total`                   | Pager shows only 1 page                                      | Add `schema: { data:"Data", total:"Total" }`                  |
| `ToList()`before `ToDataSourceResult`                         | All rows loaded into memory, paging doesn't work server-side | Pass `IQueryable`, not `List<T>`                            |
| `server-paging="false"`on large data                            | Slow load, browser memory issue                              | Enable `server-paging="true"`for > 500 rows                   |
| Filter clears on page change                                      | User applies filter, goes to page 2, filter is gone          | Normal — filters persist. If not,`serverFiltering`may be off |
| Column marked `filterable="false"`but search box still finds it | Search box searches all string fields                        | Use `<search fields="Name, Email" />`to limit                 |

---

## ❓ Interview Questions

**Q: What does `server-paging: true` do and why is it important?**

> It tells the DataSource to request only one page of records from the server at a time. The server applies SKIP/TAKE. Without it, all records are loaded at once into the browser — fine for 50 rows, catastrophic for 50,000.

**Q: How does `ToDataSourceResult(request)` handle sorting, filtering, and paging?**

> The `[DataSourceRequest]` attribute populates the `request` object with whatever the grid sent (page number, sort fields, filter conditions). `ToDataSourceResult` translates these into LINQ/SQL operations on your `IQueryable` and returns `{ Data, Total }`.

**Q: How do you filter a Kendo Grid programmatically from a button outside the grid?**

> Get the DataSource — `grid.dataSource` — and call `.filter({ field: "x", operator: "eq", value: "y" })`. Call `.filter({})` to clear all filters.

**Q: What is the difference between filter mode "row" and "menu"?**

> Row mode shows a permanent filter input row below the column headers — always visible. Menu mode shows a funnel icon in each header that opens a filter dropdown when clicked. Row is faster for frequent filtering; menu is less visually intrusive.
>
