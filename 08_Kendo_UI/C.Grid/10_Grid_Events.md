
# 10 — Grid Events

---

## 🎯 One-Line Definition

> **Grid events are hooks that fire at specific moments — when data loads, when a row is edited, when a save completes, when a row is selected — letting you run your own code at exactly the right time.**

---

## 🔑 Why Events Exist

The Grid works automatically — data loads, rows render, saves happen. But you often need to react to what the Grid is doing:

```
Without events:                  With events:
────────────────────────────     ──────────────────────────────────────
Grid saves a row                 Grid saves a row
  → nothing else happens            → you show a success notification
                                    → you refresh a chart
                                    → you update a counter
                                    → you log the action

Grid row is selected             Grid row is selected
  → nothing else happens            → you load detail data below
                                    → you enable an Edit button
                                    → you highlight related rows

Grid has a server error          Grid has a server error
  → nothing else happens            → you show user-friendly message
                                    → you reset the form
```

---

## 🔑 All Grid Events — Overview

```
DATA EVENTS (DataSource level)
────────────────────────────────────────────────────────────────
  requestStart   → before any AJAX request (read/create/update/destroy)
  requestEnd     → after any AJAX request
  change         → DataSource data changed (new data loaded, item modified)
  error          → AJAX request failed

RENDERING EVENTS (Grid widget level)
────────────────────────────────────────────────────────────────
  dataBound      → grid has finished rendering rows (most useful)

EDITING EVENTS
────────────────────────────────────────────────────────────────
  edit           → a row has entered edit mode (inline/popup opened)
  save           → user clicked Update/Save (before server call)
  saveChanges    → user clicked Save Changes toolbar button (InCell)
  cancel         → user clicked Cancel (edit discarded)
  remove         → user clicked Delete on a row

SELECTION
────────────────────────────────────────────────────────────────
  change         → selected row changed (also fires for data change)

NAVIGATION
────────────────────────────────────────────────────────────────
  columnResize   → column was resized
  columnReorder  → column was dragged to new position
  columnHide     → column was hidden
  columnShow     → column was shown
  filterMenuOpen → filter menu was opened
  sortable       → column header was clicked to sort
  page           → page was changed
```

---

## 🔑 Two Ways to Attach Events

### Way 1 — In the Tag Helper (declarative)

```cshtml
<kendo-grid name="employeeGrid">

    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("Read", "Employee")" type="POST" />
        </transport>

        {{!-- DataSource events --}}
        <events on-request-start="onRequestStart"
                on-request-end="onRequestEnd"
                on-error="onDataSourceError" />

    </datasource>

    {{!-- Grid widget events --}}
    <events on-data-bound="onDataBound"
            on-edit="onEdit"
            on-save="onSave"
            on-remove="onRemove"
            on-change="onSelectionChange" />

</kendo-grid>

@section Scripts {
<script>
    function onDataBound(e)      { ... }
    function onEdit(e)           { ... }
    function onSave(e)           { ... }
    function onRemove(e)         { ... }
    function onSelectionChange(e){ ... }
    function onRequestStart(e)   { ... }
    function onRequestEnd(e)     { ... }
    function onDataSourceError(e){ ... }
</script>
}
```

### Way 2 — With `.bind()` in JavaScript (dynamic)

```javascript
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");
    var ds   = grid.dataSource;

    // Grid widget events
    grid.bind("dataBound",    onDataBound);
    grid.bind("edit",         onEdit);
    grid.bind("save",         onSave);
    grid.bind("remove",       onRemove);
    grid.bind("change",       onSelectionChange);

    // DataSource events
    ds.bind("requestStart",   onRequestStart);
    ds.bind("requestEnd",     onRequestEnd);
    ds.bind("error",          onDataSourceError);
});
```

---

## 🔑 `dataBound` — After Rows Are Rendered

Fires every time the grid finishes drawing rows — on initial load, page change, sort, filter.
The most commonly used event.

```javascript
function onDataBound(e) {
    // e.sender = the grid widget

    var grid  = e.sender;
    var total = grid.dataSource.total();
    var count = grid.dataSource.data().length;

    // ── Update counters ───────────────────────────────────────
    $("#totalCount").text("Total: " + total + " employees");
    $("#pageCount").text("Showing " + count + " on this page");

    // ── Conditionally style rows ──────────────────────────────
    // After rows render, loop through and add CSS
    grid.tbody.find("tr").each(function() {
        var dataItem = grid.dataItem($(this));
        if (dataItem && !dataItem.IsActive) {
            $(this).addClass("inactive-row");  // grey out inactive rows
        }
        if (dataItem && dataItem.Salary > 100000) {
            $(this).addClass("high-earner-row");  // highlight high earners
        }
    });

    // ── Refresh a chart that shows summary data ───────────────
    $("#salaryChart").data("kendoChart").dataSource.read();

    // ── Re-initialize tooltips on new rows ────────────────────
    grid.tbody.find("[data-tooltip]").kendoTooltip({ position: "top" });
}
```

---

## 🔑 `edit` — When a Row Enters Edit Mode

Fires when inline edit starts, popup opens, or incell edit begins.

```javascript
function onEdit(e) {
    // e.model     = the data item being edited
    // e.container = the edit form container (row for inline, window for popup)
    // e.sender    = the grid

    var model = e.model;

    // ── Different title for Add vs Edit (popup mode) ──────────
    if (model.isNew()) {
        e.container.data("kendoWindow").title("Add New Employee");
        // Set default values for new record
        model.set("IsActive",   true);
        model.set("HireDate",   new Date());
        model.set("Department", "IT");
    } else {
        e.container.data("kendoWindow")
            .title("Edit — " + model.Name);
    }

    // ── Focus first field automatically ──────────────────────
    setTimeout(function() {
        e.container.find("[name='Name']").focus();
    }, 50);

    // ── Disable a field for existing records ─────────────────
    if (!model.isNew()) {
        var emailInput = e.container.find("[name='Email']");
        emailInput.attr("disabled", true);
        // OR for Kendo widget:
        // emailInput.data("kendoTextBox").enable(false);
    }

    // ── Load cascade dropdown based on existing value ─────────
    if (!model.isNew()) {
        var dept = model.Department;
        loadRolesForDepartment(dept, e.container);
    }
}
```

---

## 🔑 `save` — Before the Server Call

Fires after user clicks Update/Save but BEFORE the data is sent to the server.
Use to validate, transform data, or cancel the save.

```javascript
function onSave(e) {
    // e.model     = the data item (all current values)
    // e.container = the edit container
    // e.values    = only the changed values (for inline/popup)
    // e.sender    = the grid

    // ── Custom validation ─────────────────────────────────────
    if (e.model.Salary > 500000) {
        e.preventDefault();   // CANCEL the save

        kendo.alert("Salary cannot exceed $500,000. Please check.");
        return;
    }

    // ── Transform before save ─────────────────────────────────
    if (e.model.Name) {
        // Capitalise first letter
        e.model.set("Name",
            e.model.Name.charAt(0).toUpperCase() +
            e.model.Name.slice(1)
        );
    }

    // ── Log the change ────────────────────────────────────────
    console.log("Saving:", e.model.Name, "— Changes:", e.values);
}
```

---

## 🔑 `saveChanges` — InCell Bulk Save

Fires when user clicks the "Save Changes" toolbar button in InCell mode.
This is different from `save` — it covers ALL pending changes at once.

```javascript
function onSaveChanges(e) {
    // e.sender = the grid

    var grid   = e.sender;
    var hasNew = false;

    // Check if there are unsaved new rows
    grid.dataSource.data().forEach(function(item) {
        if (item.isNew()) hasNew = true;
    });

    if (hasNew) {
        // Confirm before saving new records
        e.preventDefault();

        kendo.confirm("This will add new employees. Continue?")
            .then(function() {
                grid.dataSource.sync();  // proceed with save
            });
    }
}
```

---

## 🔑 `remove` — Before Deletion

Fires when user clicks Delete on a row — BEFORE the destroy request is sent.

```javascript
function onRemove(e) {
    // e.model  = the data item being deleted
    // e.row    = the jQuery <tr> element
    // e.sender = the grid

    var name = e.model.Name;

    // ── Custom confirm ────────────────────────────────────────
    e.preventDefault();  // stop automatic deletion

    kendo.confirm(
        "Delete <strong>" + name + "</strong>?" +
        "<br><small>This cannot be undone.</small>"
    ).then(function() {
        // User confirmed — proceed
        e.sender.dataSource.remove(e.model);
        e.sender.dataSource.sync();
    });

    // ── Log deletions ─────────────────────────────────────────
    console.log("Deleting employee ID:", e.model.Id);
}
```

---

## 🔑 `change` (Selection) — Row Selected

When `<selectable>` is enabled, the `change` event fires when selection changes.

```javascript
// Enable row selection
<selectable mode="single" />

// Handle selection
function onSelectionChange(e) {
    var grid    = e.sender;
    var selected = grid.select();  // selected <tr> element(s)

    if (!selected.length) return;

    var employee = grid.dataItem(selected[0]);

    // ── Load detail data below the grid ──────────────────────
    loadEmployeeProjects(employee.Id);
    loadEmployeeDetails(employee.Id);

    // ── Update a form ─────────────────────────────────────────
    $("#detailName").text(employee.Name);
    $("#detailDept").text(employee.Department);
    $("#detailSalary").text(kendo.format("{0:C0}", employee.Salary));

    // ── Enable action buttons ─────────────────────────────────
    $("#editSelectedBtn").prop("disabled", false);
    $("#deleteSelectedBtn").prop("disabled", false);
}
```

---

## 🔑 `requestStart` and `error` — DataSource Events

```javascript
// ── requestStart: show spinner ────────────────────────────────
function onRequestStart(e) {
    kendo.ui.progress($("#employeeGrid"), true);

    // e.type = "read" / "create" / "update" / "destroy"
    if (e.type === "create") {
        console.log("Creating new record...");
    }
}

// ── requestEnd: hide spinner ─────────────────────────────────
function onRequestEnd(e) {
    kendo.ui.progress($("#employeeGrid"), false);

    // After a successful save, refresh other parts of the page
    if (e.type === "create" || e.type === "update") {
        updateDashboardStats();
    }
}

// ── error: handle failures ────────────────────────────────────
function onDataSourceError(e) {
    kendo.ui.progress($("#employeeGrid"), false);

    if (e.errors) {
        // Server-side validation errors
        var messages = [];
        $.each(e.errors, function(field, detail) {
            messages.push(field + ": " + detail.errors.join(", "));
        });
        kendo.alert("Please fix the following:\n" + messages.join("\n"));
    } else {
        var code = e.xhr ? e.xhr.status : "unknown";
        kendo.alert("Request failed (Error " + code + "). Please try again.");
    }

    // CRITICAL: cancel pending changes so grid resets to clean state
    e.sender.cancelChanges();
}
```

---

## 💻 Complete Real-World Event Setup

Putting it all together — a production-ready event setup:

```javascript
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");
    var ds   = grid.dataSource;

    // ── Loading indicator ─────────────────────────────────────
    ds.bind("requestStart", function() {
        kendo.ui.progress($("#employeeGrid"), true);
    });
    ds.bind("requestEnd", function(e) {
        kendo.ui.progress($("#employeeGrid"), false);
        if (e.type === "create" || e.type === "update" || e.type === "destroy") {
            showNotification("Changes saved successfully", "success");
            updateEmployeeCount();
        }
    });

    // ── Error handling ────────────────────────────────────────
    ds.bind("error", function(e) {
        kendo.ui.progress($("#employeeGrid"), false);
        if (e.errors) {
            var msgs = [];
            $.each(e.errors, function(f, d) { msgs.push(f + ": " + d.errors[0]); });
            showNotification("Validation: " + msgs.join(" | "), "error");
        } else {
            showNotification("Server error " + (e.xhr ? e.xhr.status : ""), "error");
        }
        e.sender.cancelChanges();
    });

    // ── After rows render ─────────────────────────────────────
    grid.bind("dataBound", function(e) {
        updateEmployeeCount();
        highlightInactiveRows(e.sender);
    });

    // ── Popup opens ───────────────────────────────────────────
    grid.bind("edit", function(e) {
        if (e.model.isNew()) {
            e.container.data("kendoWindow").title("Add New Employee");
            e.model.set("IsActive", true);
            e.model.set("HireDate", new Date());
        } else {
            e.container.data("kendoWindow").title("Edit: " + e.model.Name);
        }
        setTimeout(function() {
            e.container.find("[name='Name']").focus();
        }, 50);
    });

    // ── Before save validation ────────────────────────────────
    grid.bind("save", function(e) {
        if (e.model.Salary < 0) {
            e.preventDefault();
            showNotification("Salary cannot be negative", "error");
        }
    });

    // ── Selection ─────────────────────────────────────────────
    grid.bind("change", function() {
        var row = this.select();
        if (row.length) {
            var emp = this.dataItem(row[0]);
            loadEmployeeProjects(emp.Id);
        }
    });
});

function updateEmployeeCount() {
    var total = $("#employeeGrid").data("kendoGrid").dataSource.total();
    $("#empCount").text("Total: " + total);
}

function highlightInactiveRows(grid) {
    grid.tbody.find("tr").each(function() {
        var item = grid.dataItem($(this));
        if (item && !item.IsActive) $(this).addClass("row-inactive");
    });
}
```

---

## 📊 Events Quick Reference

| Event            | `e`Contains               | Common Use                                  |
| ---------------- | --------------------------- | ------------------------------------------- |
| `dataBound`    | `e.sender`(grid)          | Style rows, update counters, refresh charts |
| `edit`         | `e.model`,`e.container` | Set title, set defaults, disable fields     |
| `save`         | `e.model`,`e.values`    | Validate before save, transform data        |
| `saveChanges`  | `e.sender`                | Confirm bulk InCell save                    |
| `remove`       | `e.model`,`e.row`       | Custom confirm dialog                       |
| `change`       | `e.sender`                | Load detail data for selected row           |
| `requestStart` | `e.type`                  | Show spinner                                |
| `requestEnd`   | `e.type`,`e.response`   | Hide spinner, refresh related widgets       |
| `error`        | `e.errors`,`e.xhr`      | Show errors, call `cancelChanges()`       |

---

## ⚠️ Common Mistakes

| Mistake                                                              | Symptom                                           | Fix                                                                  |
| -------------------------------------------------------------------- | ------------------------------------------------- | -------------------------------------------------------------------- |
| Not calling `cancelChanges()`in `error`handler                   | Grid stays in broken edit state after failed save | Always call `e.sender.cancelChanges()`in error                     |
| Using `save`event instead of `requestEnd`to show "saved" message | Message shows before server confirms success      | Show message in `requestEnd`where `e.type === "create"/"update"` |
| Forgetting `e.preventDefault()`before custom confirm in `remove` | Record deletes immediately before user confirms   | Call `e.preventDefault()`first, then confirm                       |
| Accessing popup window before it's ready in `edit`                 | `data("kendoWindow")`returns undefined          | Popup window is only available in `edit`event, not earlier         |
| Styling rows in `dataBound`without checking `dataItem`           | JS error on group/summary rows                    | Always check `if (dataItem && dataItem.Id)`before accessing        |

---

## ❓ Interview Questions

**Q: What is the `dataBound` event and when does it fire?**

> It fires every time the grid finishes rendering its rows — on initial load, page change, sort, and filter. It's the right place to apply custom row styling, update counters, and refresh related widgets.

**Q: What does `e.preventDefault()` do in the `save` event?**

> It cancels the save operation — the data is NOT sent to the server. Use it when custom client-side validation fails, giving you a chance to show the user what to fix before proceeding.

**Q: Why must you call `e.sender.cancelChanges()` inside the error handler?**

> When a save fails, the DataSource still has the unsaved changes in a "dirty" state. Without `cancelChanges()`, the grid stays in edit mode or shows the row as modified. Calling it resets the DataSource to its last clean state.

**Q: What is the difference between the `save` event and `requestEnd`?**

> `save` fires client-side before the AJAX request is sent — use it for pre-save validation. `requestEnd` fires after the server responds — use it for post-save actions like showing a success notification or refreshing a chart, because at that point you know the server accepted the change.

**Q: How do you show different popup titles for Add New vs Edit?**

> In the `edit` event handler, check `e.model.isNew()`. If true, call `e.container.data("kendoWindow").title("Add New...")`. If false, it's an existing record — set the title with the record's name or ID.
>
