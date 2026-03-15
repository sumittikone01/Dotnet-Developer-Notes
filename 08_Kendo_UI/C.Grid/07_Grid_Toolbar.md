
# 07 — Grid Toolbar

---

## 🎯 One-Line Definition

> **The Grid toolbar is the action bar above the table — it holds buttons for adding records, exporting data, searching, and any custom actions you need, all configurable with a few lines.**

---

## 🔑 What the Toolbar Looks Like

```
┌──────────────────────────────────────────────────────────────────┐
│ TOOLBAR                                                           │
│  [+ Add New]  [↓ Export Excel]  [📄 Export PDF]  [🔍 Search... ]│
├────────┬──────────────┬─────────────┬──────────┬─────────────────┤
│ #      │ Name         │ Department  │ Salary   │ Actions         │
├────────┼──────────────┼─────────────┼──────────┼─────────────────┤
│ 1      │ Alice        │ IT          │ $75,000  │ [Edit] [Delete] │
│ 2      │ Bob          │ HR          │ $55,000  │ [Edit] [Delete] │
└────────┴──────────────┴─────────────┴──────────┴─────────────────┘
```

---

## 🔑 Built-in Toolbar Buttons

Kendo ships with these ready-made toolbar buttons — just name them:

```cshtml
<toolbar>
    <toolbar-button name="create"        text="+ Add New"      />
    <toolbar-button name="save"          text="Save Changes"   />
    <toolbar-button name="cancel"        text="Cancel Changes" />
    <toolbar-button name="excel"         text="Export Excel"   />
    <toolbar-button name="pdf"           text="Export PDF"     />
    <toolbar-button name="search"                              />
</toolbar>
```

| Button `name` | What It Does                     | Used With               |
| --------------- | -------------------------------- | ----------------------- |
| `create`      | Opens a new empty row or popup   | All edit modes          |
| `save`        | Saves all pending changes        | InCell mode mainly      |
| `cancel`      | Cancels all pending changes      | InCell mode mainly      |
| `excel`       | Downloads grid data as `.xlsx` | Needs `<excel>`config |
| `pdf`         | Downloads grid data as `.pdf`  | Needs `<pdf>`config   |
| `search`      | Shows a search input box         | Any read grid           |

---

## 🔑 The `create` Button — Adding New Records

```cshtml
<toolbar>
    <toolbar-button name="create" text="+ Add New Employee" />
</toolbar>

{{!-- What happens depends on the edit mode: --}}
{{!--   inline  → inserts new row at top of grid          --}}
{{!--   popup   → opens the popup form with empty fields  --}}
{{!--   incell  → inserts new row at top, first cell active --}}
```

---

## 🔑 Excel Export

```cshtml
{{!-- Step 1: Add export button to toolbar --}}
<toolbar>
    <toolbar-button name="excel" text="⬇ Export to Excel" />
</toolbar>

{{!-- Step 2: Configure the Excel file --}}
<excel file-name="employees.xlsx"
       all-pages="true"
       filterable="true" />
```

| Option         | What It Does                       | Value                |
| -------------- | ---------------------------------- | -------------------- |
| `file-name`  | Downloaded file name               | `"employees.xlsx"` |
| `all-pages`  | Export all pages, not just current | `true`/`false`   |
| `filterable` | Allow filtering in the Excel file  | `true`/`false`   |

```javascript
// Trigger export programmatically from JavaScript
$("#employeeGrid").data("kendoGrid").saveAsExcel();
```

---

## 🔑 PDF Export

```cshtml
{{!-- Step 1: Add PDF button to toolbar --}}
<toolbar>
    <toolbar-button name="pdf" text="📄 Export to PDF" />
</toolbar>

{{!-- Step 2: Configure the PDF --}}
<pdf file-name="employees.pdf"
     all-pages="true"
     paper-size="A4"
     landscape="true"
     repeat-headers="true"
     scale="0.7" />
```

| Option             | What It Does                                      |
| ------------------ | ------------------------------------------------- |
| `file-name`      | Downloaded file name                              |
| `all-pages`      | Include all pages                                 |
| `paper-size`     | `"A4"`,`"Letter"`,`"A3"`                    |
| `landscape`      | Horizontal orientation                            |
| `repeat-headers` | Column headers on every page                      |
| `scale`          | Shrink factor (0.7 = 70% size, fits more columns) |

```javascript
// Trigger PDF export programmatically
$("#employeeGrid").data("kendoGrid").saveAsPDF();
```

---

## 🔑 Search Box

```cshtml
<toolbar>
    <toolbar-button name="search" />
</toolbar>
{{!-- Searches across ALL string columns by default --}}
```

```
[🔍 Type to search...                    ]
     ↓ user types "ali"
Grid filters: Name contains "ali"  OR  Department contains "ali"
(searches all string fields simultaneously)
```

**Limit which columns are searched:**

```cshtml
<toolbar>
    <toolbar-button name="search" />
</toolbar>

{{!-- Add this outside toolbar to control search fields --}}
<search fields="Name, Email, Department" />
```

---

## 🔑 Custom Toolbar Buttons

Add your own buttons beyond the built-in ones:

```cshtml
{{!-- Method 1: Template string inline --}}
<toolbar>
    <toolbar-button name="create" text="+ Add New" />
    <toolbar-button template="<button class='k-button k-button-solid-primary' onclick='exportReport()'>📊 Monthly Report</button>" />
    <toolbar-button template="<button class='k-button' onclick='sendReminders()'>📧 Send Reminders</button>" />
</toolbar>
```

```cshtml
{{!-- Method 2: External template (cleaner for complex buttons) --}}
<toolbar>
    <toolbar-button name="create" text="+ Add New" />
    <toolbar-button template-id="customToolbarTemplate" />
</toolbar>

<script type="text/x-kendo-template" id="customToolbarTemplate">
    <div class="toolbar-custom">
        <button class="k-button k-button-solid-primary"
                onclick="bulkActivate()">
            ✅ Activate Selected
        </button>
        <button class="k-button k-button-solid-error"
                onclick="bulkDeactivate()">
            ❌ Deactivate Selected
        </button>
        <select id="deptFilter" class="k-select"
                onchange="filterByDept(this.value)">
            <option value="">All Departments</option>
            <option value="IT">IT</option>
            <option value="HR">HR</option>
            <option value="Finance">Finance</option>
        </select>
    </div>
</script>
```

```javascript
function bulkActivate() {
    var grid    = $("#employeeGrid").data("kendoGrid");
    var checked = grid.tbody.find("tr input[type='checkbox']:checked");
    var ids     = [];
    checked.each(function() {
        ids.push($(this).closest("tr").data("uid"));
    });
    // send ids to server...
}

function filterByDept(dept) {
    var grid = $("#employeeGrid").data("kendoGrid");
    if (dept) {
        grid.dataSource.filter({ field: "Department", operator: "eq", value: dept });
    } else {
        grid.dataSource.filter({});  // clear filter
    }
}
```

---

## 🔑 Toolbar with Department Filter Dropdown — Real Pattern

A very common real-world pattern: a department dropdown in the toolbar that filters the grid:

```cshtml
<toolbar>
    <toolbar-button name="create" text="+ Add New" />
    <toolbar-button template-id="deptFilterTemplate" />
    <toolbar-button name="excel" text="Export" />
    <toolbar-button name="search" />
</toolbar>

<script type="text/x-kendo-template" id="deptFilterTemplate">
    <span style="margin-left:10px; font-weight:bold;">Department:</span>
    <select id="deptToolbarFilter"
            class="k-select"
            style="width:160px; margin-left:5px;">
        <option value="">All</option>
    </select>
</script>
```

```javascript
$(function() {
    // Populate the dropdown from server
    $.get("/Department/GetAll", function(depts) {
        var select = $("#deptToolbarFilter");
        depts.forEach(function(d) {
            select.append($("<option>").val(d.name).text(d.name));
        });
    });

    // Filter grid when selection changes
    $(document).on("change", "#deptToolbarFilter", function() {
        var val  = $(this).val();
        var grid = $("#employeeGrid").data("kendoGrid");

        if (val) {
            grid.dataSource.filter({
                field: "Department", operator: "eq", value: val
            });
        } else {
            grid.dataSource.filter({});
        }

        grid.dataSource.page(1);  // reset to page 1 after filter
    });
});
```

---

## 🔑 Toolbar Position — Top and Bottom

```cshtml
{{!-- Default: toolbar at TOP --}}
<kendo-grid name="employeeGrid">
    <toolbar>
        <toolbar-button name="create" />
    </toolbar>
    ...
</kendo-grid>

{{!-- Toolbar at BOTTOM --}}
<kendo-grid name="employeeGrid" toolbar-client-template-id="bottomToolbar">
...
</kendo-grid>
{{!-- Note: for true bottom placement use JavaScript initialization --}}
```

```javascript
// Position toolbar at bottom using JavaScript init
$("#employeeGrid").kendoGrid({
    toolbar: ["create", "excel"],
    ...
});
// Then move it:
var grid    = $("#employeeGrid").data("kendoGrid");
var toolbar = grid.element.find(".k-toolbar");
grid.element.append(toolbar);   // moves toolbar to after the grid body
```

---

## 🔑 Styling Toolbar Buttons

```css
/* Make the Add New button green */
.k-grid-add {
    background-color: #27ae60 !important;
    color: white !important;
    border-color: #219a52 !important;
}

/* Make Export Excel button blue */
.k-grid-excel {
    background-color: #2980b9 !important;
    color: white !important;
}

/* Custom toolbar button styling */
.toolbar-custom .k-button {
    margin-right: 8px;
}
```

---

## 📊 Toolbar Quick Reference

| Scenario             | What to Add                                                             |
| -------------------- | ----------------------------------------------------------------------- |
| Allow adding records | `<toolbar-button name="create" />`                                    |
| Excel download       | `<toolbar-button name="excel" />`+`<excel file-name="..." />`       |
| PDF download         | `<toolbar-button name="pdf" />`+`<pdf file-name="..." />`           |
| Quick search         | `<toolbar-button name="search" />`                                    |
| InCell save all      | `<toolbar-button name="save" />`+`<toolbar-button name="cancel" />` |
| Custom action        | `<toolbar-button template="<button...>...</button>" />`               |
| Complex toolbar      | `<toolbar-button template-id="myTemplate" />`+ separate `<script>`  |

---

## ⚠️ Common Mistakes

| Mistake                                         | Symptom                                | Fix                                                                                                 |
| ----------------------------------------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `excel`button but no `<excel>`config        | Button appears but nothing downloads   | Add `<excel file-name="file.xlsx" />`to the grid                                                  |
| `save`/`cancel`buttons on popup/inline grid | Buttons appear but behave unexpectedly | `save`/`cancel`are for InCell mode — remove for other modes                                    |
| Custom template button not found on load        | JS error: element doesn't exist        | Template renders after grid init — use `$(document).on("click", ...)`not `$("#myBtn").on(...)` |
| Export shows only current page                  | User gets 10 rows when expecting all   | Set `all-pages="true"`on `<excel>`                                                              |

---

## ❓ Interview Questions

**Q: What is the Kendo Grid toolbar?**

> The action bar rendered above the grid columns. It holds built-in buttons like create, excel, pdf, and search, plus any custom buttons or controls you define with a template.

**Q: How do you add a custom button to the toolbar?**

> Use `<toolbar-button template="...">` with an inline HTML string, or `<toolbar-button template-id="...">` pointing to a `<script type="text/x-kendo-template">` block for more complex content.

**Q: How do you export all pages to Excel, not just the current page?**

> Add `all-pages="true"` to the `<excel>` configuration: `<excel file-name="data.xlsx" all-pages="true" />`. Without it, only the currently visible page is exported.

**Q: Which toolbar buttons are specific to InCell editing mode?**

> `save` and `cancel` — they save or discard all pending cell changes at once. In inline and popup mode, saving is per-row, so these toolbar buttons aren't needed.
>
