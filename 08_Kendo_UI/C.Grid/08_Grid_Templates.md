# 08 — Grid Templates

---

## 🎯 One-Line Definition

> **Grid templates let you replace plain text in any part of the grid — cells, rows, headers, the detail row — with custom HTML, conditional styling, icons, buttons, or any other markup you need.**

---

## 🔑 Where Templates Are Used in a Grid

```
┌────────────────────────────────────────────────────────────────┐
│ TOOLBAR TEMPLATE  [+ Add]  [Custom Button]                    │ ← toolbar template
├─────────┬──────────────────┬─────────────────────────────────┤
│ # ↑↓    │ [📋 Name]  ↑↓   │  Status  ↑↓                    │ ← header template
├─────────┼──────────────────┼─────────────────────────────────┤
│ 1       │ 🟢 Alice — IT   │  ✅ Active                      │ ← column template
│ 2       │ 🔴 Bob — HR     │  ❌ Inactive                    │ ← column template
├─────────┴──────────────────┴─────────────────────────────────┤
│   ▼  [expand]  Details for Alice...                          │ ← row detail template
│       Projects: HR System, Payroll App                       │
├────────────────────────────────────────────────────────────────┤
│ No records to display                                         │ ← no-data template
└────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Template Syntax — The Four Tags

```javascript
#= expression #    → Output value (renders HTML — use for safe content)
                     Example: #= Name #  → Alice
                     Example: #= "<b>" + Name + "</b>" # → <b>Alice</b>

#: expression #    → Output value (HTML-ENCODED — safe for user input)
                     Example: #: UserComment #  → shows & as & etc.
                     USE THIS for any user-entered text (prevents XSS)

# code; #          → Run JavaScript (no output)
                     Example: # if (Salary > 80000) { #
                                  High earner!
                              # } #

#= kendo.format() #→ Format numbers and dates
                     Example: #= kendo.format('{0:C0}', Salary) #  → $75,000
                     Example: #= kendo.toString(HireDate, 'MM/dd/yyyy') #
```

---

## 🔑 Column Template — Most Used

Replace a cell's plain text with custom HTML:

```cshtml
{{!-- 1. Status badge with colour --}}
<column field="IsActive" title="Status" width="120">
    <column-template>
        # if (IsActive) { #
            <span class="badge bg-success">✅ Active</span>
        # } else { #
            <span class="badge bg-danger">❌ Inactive</span>
        # } #
    </column-template>
</column>


{{!-- 2. Salary with colour based on value --}}
<column field="Salary" title="Salary" width="140">
    <column-template>
        <span style="color: #= Salary >= 80000 ? '#27ae60' : Salary >= 50000 ? '#e67e22' : '#e74c3c' #;
                     font-weight: bold;">
            #= kendo.format('{0:C0}', Salary) #
        </span>
    </column-template>
</column>


{{!-- 3. Profile photo --}}
<column field="Name" title="Employee" width="220">
    <column-template>
        <div style="display:flex; align-items:center; gap:10px;">
            <img src="#= PhotoUrl || '/images/default-avatar.png' #"
                 style="width:36px;height:36px;border-radius:50%;object-fit:cover;" />
            <div>
                <div style="font-weight:bold;">#: Name #</div>
                <div style="font-size:12px;color:#666;">#: Email #</div>
            </div>
        </div>
    </column-template>
</column>


{{!-- 4. Progress bar for KPI score --}}
<column field="KpiScore" title="KPI" width="160">
    <column-template>
        <div style="background:#eee; border-radius:4px; overflow:hidden; height:18px;">
            <div style="width:#= KpiScore #%;
                        background: #= KpiScore >= 80 ? '#27ae60' : KpiScore >= 50 ? '#f39c12' : '#e74c3c' #;
                        height:100%;
                        line-height:18px;
                        text-align:center;
                        color:white;
                        font-size:11px;">
                #= KpiScore #%
            </div>
        </div>
    </column-template>
</column>


{{!-- 5. Custom action buttons --}}
<column title="Actions" width="220">
    <column-template>
        <button class="k-button k-button-sm k-button-solid-primary viewBtn"
                data-id="#= Id #">
            👁 View
        </button>
        <button class="k-button k-button-sm k-button-solid-warning editBtn"
                data-id="#= Id #">
            ✏ Edit
        </button>
        <button class="k-button k-button-sm k-button-solid-error deleteBtn"
                data-id="#= Id #">
            🗑 Delete
        </button>
    </column-template>
</column>
```

---

## 🔑 Column Header Template

Customise the column header cell — add icons, tooltips, or styled text:

```cshtml
{{!-- Icon in header --}}
<column field="Salary" width="140" format="{0:C0}">
    <header-template>
        💰 Salary
    </header-template>
</column>


{{!-- Header with info tooltip --}}
<column field="KpiScore" width="120">
    <header-template>
        KPI Score
        <span class="k-icon k-i-information"
              title="Key Performance Indicator — rated 0 to 100"
              style="cursor:help; color:#3498db;">
        </span>
    </header-template>
</column>


{{!-- Styled header with background --}}
<column field="IsActive" width="100">
    <header-template>
        <span style="color:#27ae60; font-weight:bold;">● Status</span>
    </header-template>
</column>
```

---

## 🔑 Row Template — Style the Entire Row

Replace the entire `<tr>` for each data row with custom HTML.
Use when you need full control over row structure:

```cshtml
{{!-- Row template defined outside the grid --}}
<script type="text/x-kendo-template" id="rowTemplate">
    <tr data-uid="#: uid #"
        style="background: #= IsActive ? 'white' : '#fff5f5' #;">

        <td>#= Id #</td>

        <td>
            <strong>#: Name #</strong>
            <br/>
            <small style="color:#888;">#: Email #</small>
        </td>

        <td>
            <span class="dept-badge dept-#= Department.toLowerCase() #">
                #= Department #
            </span>
        </td>

        <td style="font-weight:bold; color: #= Salary > 70000 ? 'green' : 'inherit' #">
            #= kendo.format('{0:C0}', Salary) #
        </td>

        <td>
            # if (IsActive) { #
                <span class="badge bg-success">Active</span>
            # } else { #
                <span class="badge bg-secondary">Inactive</span>
            # } #
        </td>

        <td>
            <button class="k-button k-button-sm editBtn" data-id="#= Id #">Edit</button>
            <button class="k-button k-button-sm k-button-solid-error deleteBtn"
                    data-id="#= Id #">Delete</button>
        </td>
    </tr>
</script>

{{!-- Also define an alt row template for alternating row colours --}}
<script type="text/x-kendo-template" id="altRowTemplate">
    <tr data-uid="#: uid #"
        style="background: #= IsActive ? '#f9f9f9' : '#fff0f0' #;">
        {{!-- same content as rowTemplate --}}
    </tr>
</script>
```

```cshtml
{{!-- Reference the row templates in the grid --}}
<kendo-grid name="employeeGrid"
            row-template-id="rowTemplate"
            alt-row-template-id="altRowTemplate">
    <columns>
        {{!-- columns must still be defined for headers --}}
        <column field="Id"         width="60"  />
        <column field="Name"       width="220" />
        <column field="Department" width="150" />
        <column field="Salary"     width="140" />
        <column field="IsActive"   width="100" />
        <column title="Actions"    width="180" />
    </columns>
    ...
</kendo-grid>
```

> ⚠️ **Important:** When using row templates, `data-uid="#: uid #"` is mandatory. Kendo uses this UID to identify rows for edit, delete, and selection. Without it, editing breaks.

---

## 🔑 Detail Row Template — Expandable Rows

Add a collapsible detail section to each row — show extra info without extra columns:

```cshtml
{{!-- The expand icon appears automatically in first column --}}
<kendo-grid name="employeeGrid">
    <columns>
        <column field="Name"       width="200" />
        <column field="Department" width="150" />
        <column field="Salary"     width="130" format="{0:C0}" />
    </columns>

    <detail-template>
        <div style="padding: 15px; background: #f8f9fa;">
            <h5 style="margin:0 0 10px;">#: Name #'s Details</h5>
            <table class="table table-sm" style="width:auto;">
                <tr><td><b>Email:</b></td>     <td>#: Email #</td></tr>
                <tr><td><b>Phone:</b></td>     <td>#: Phone #</td></tr>
                <tr><td><b>Hire Date:</b></td> <td>#= kendo.toString(HireDate, 'MMM dd, yyyy') #</td></tr>
                <tr><td><b>Manager:</b></td>   <td>#: ManagerName #</td></tr>
            </table>
        </div>
    </detail-template>

    <datasource type="DataSourceTagHelperType.Ajax" page-size="10">
        <transport>
            <read url="@Url.Action("Read", "Employee")" type="POST" />
        </transport>
    </datasource>

</kendo-grid>
```

```
Result:
┌────┬───────────────┬────────────┬──────────┐
│ ▶  │ Alice         │ IT         │ $75,000  │ ← click ▶ to expand
├────┼───────────────┼────────────┼──────────┤
│ ▼  │ Bob           │ HR         │ $55,000  │ ← expanded ▼
│    │  Bob's Details                        │
│    │  Email:    bob@company.com            │
│    │  Phone:    555-1234                   │
│    │  Hire Date: Jan 15, 2020              │
│    │  Manager:   Alice Johnson             │
├────┼───────────────┼────────────┼──────────┤
│ ▶  │ Carol         │ Finance    │ $60,000  │
└────┴───────────────┴────────────┴──────────┘
```

---

## 🔑 Detail Row with AJAX — Load Data on Expand

Load detail data only when the row is expanded (lazy loading):

```javascript
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");

    grid.bind("detailExpand", function(e) {
        var empId      = this.dataItem(e.masterRow).Id;
        var detailDiv  = e.detailRow.find(".employee-detail");

        // Load projects for this employee
        $.get("/Employee/GetProjects", { employeeId: empId }, function(projects) {
            var html = "<ul>";
            projects.forEach(function(p) {
                html += "<li>" + p.title + " — " + p.status + "</li>";
            });
            html += "</ul>";
            detailDiv.html(html);
        });
    });
});
```

```cshtml
<detail-template>
    <div class="employee-detail">
        Loading...
    </div>
</detail-template>
```

---

## 🔑 No-Records Template — When Grid is Empty

```cshtml
{{!-- Custom message when no data matches --}}
<no-records template="<div style='text-align:center;padding:40px;color:#888;'>
    <p style='font-size:24px;'>📭</p>
    <p style='font-size:16px;font-weight:bold;'>No employees found</p>
    <p>Try adjusting your filters or add a new employee.</p>
    </div>" />
```

---

## 🔑 Template Functions — JavaScript for Complex Logic

For complex templates, define a JavaScript function:

```javascript
// Define template function
function salaryTemplate(dataItem) {
    var pct   = (dataItem.Salary / 200000 * 100).toFixed(0);
    var color = dataItem.Salary >= 80000 ? "#27ae60"
              : dataItem.Salary >= 50000 ? "#f39c12"
                                         : "#e74c3c";
    return '<div style="display:flex; align-items:center; gap:8px;">' +
               '<div style="width:80px;background:#eee;border-radius:3px;height:10px;">' +
                   '<div style="width:' + pct + '%;background:' + color + ';height:100%;border-radius:3px;"></div>' +
               '</div>' +
               '<span style="color:' + color + ';font-weight:bold;">' +
                   kendo.format("{0:C0}", dataItem.Salary) +
               '</span>' +
           '</div>';
}

function deptBadgeTemplate(dataItem) {
    var colors = {
        "IT":      "#3498db",
        "HR":      "#e74c3c",
        "Finance": "#27ae60",
        "Sales":   "#f39c12"
    };
    var color = colors[dataItem.Department] || "#95a5a6";
    return '<span style="background:' + color + ';color:white;padding:2px 8px;' +
               'border-radius:12px;font-size:11px;">' +
               dataItem.Department +
           '</span>';
}
```

```cshtml
{{!-- Reference function name as the template --}}
<column field="Salary"     title="Salary"     template="#= salaryTemplate(data) #"    />
<column field="Department" title="Department" template="#= deptBadgeTemplate(data) #" />
```

---

## 🔑 Handling Button Clicks in Templates

Buttons inside column templates need event delegation:

```javascript
$(function() {

    // Use $(document).on() — NOT $(".viewBtn").on()
    // because template buttons are created dynamically after grid loads

    $(document).on("click", ".viewBtn", function() {
        var empId = $(this).data("id");
        openViewWindow(empId);
    });

    $(document).on("click", ".editBtn", function() {
        var empId = $(this).data("id");
        // Find the grid row and trigger inline/popup edit
        var grid = $("#employeeGrid").data("kendoGrid");
        var row  = $(this).closest("tr");
        grid.editRow(row);
    });

    $(document).on("click", ".deleteBtn", function() {
        var empId = $(this).data("id");
        kendo.confirm("Delete this employee?")
            .then(function() {
                $.post("/Employee/Delete", { id: empId }, function() {
                    $("#employeeGrid").data("kendoGrid").dataSource.read();
                });
            });
    });

});
```

---

## 📊 Template Types Quick Reference

| Template            | Where It Appears              | Set With                               |
| ------------------- | ----------------------------- | -------------------------------------- |
| Column template     | Inside each data cell         | `<column-template>`                  |
| Header template     | Inside the column header      | `<header-template>`                  |
| Row template        | Replaces the entire `<tr>`  | `row-template-id="..."`              |
| Alt row template    | Alternating rows              | `alt-row-template-id="..."`          |
| Detail template     | Expandable row below each row | `<detail-template>`                  |
| No-records template | When grid has no data         | `<no-records template="...">`        |
| Toolbar template    | Inside the toolbar bar        | `<toolbar-button template-id="...">` |

---

## ⚠️ Common Mistakes

| Mistake                                  | Symptom                                                    | Fix                                                   |
| ---------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| Using `#= #`for user content           | XSS vulnerability — user can inject HTML/scripts          | Use `#: #`for any user-entered data                 |
| Row template missing `data-uid`        | Edit/delete breaks, grid can't identify rows               | Always include `data-uid="#: uid #"`on the `<tr>` |
| Direct click handler on template buttons | Buttons don't respond (elements didn't exist at bind time) | Use `$(document).on("click", ".btnClass", fn)`      |
| JavaScript error inside template         | Grid rows show empty or broken                             | Wrap risky expressions: `#= val                       |
| Template accesses non-existent field     | Undefined error, row breaks                                | Match field names exactly to JSON property names      |

---

## ❓ Interview Questions

**Q: What is the difference between `#= #` and `#: #` in a Kendo template?**

> `#= #` outputs the value directly and renders HTML tags — use it for values you control (formatted numbers, conditional HTML). `#: #` HTML-encodes the output — use it for any user-provided data to prevent XSS attacks (a user entering `<script>alert(1)</script>` would display as text, not execute).

**Q: Why must a row template include `data-uid="#: uid #"`?**

> Kendo uses the UID internally to identify rows for edit, delete, selection, and other operations. Without it, the grid can't map user actions back to data items — editing and deleting stop working.

**Q: How do you handle click events on buttons inside column templates?**

> Use event delegation — `$(document).on("click", ".buttonClass", handler)`. Direct binding like `$(".buttonClass").on("click", ...)` won't work because the buttons are created after the event is bound (they're rendered dynamically by the grid).

**Q: What is the detail row template used for?**

> To show expandable additional information for each row without adding more columns. A `▶` icon appears in the first column — clicking it expands a detail section below the row where you can show extra fields, related data, or even a nested grid.
>
