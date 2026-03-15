
# 09 — Grid Refresh and Destroy

---

## 🎯 One-Line Definition

> **Refresh reloads the grid's data from the server without touching the widget. Destroy completely removes the grid widget from the DOM element so you can rebuild it from scratch with new configuration.**

---

## 🔑 Refresh vs Destroy — Know the Difference

```
┌───────────────────────────────────────────────────────────────┐
│  REFRESH                         │  DESTROY                   │
│  ────────────────────────────    │  ─────────────────────     │
│  Grid widget stays               │  Grid widget is removed    │
│  Configuration unchanged         │  Configuration gone        │
│  Same columns, same options      │  Element is empty <div>    │
│  Just reloads data from server   │  Can now re-init with      │
│                                  │  completely new options     │
│  When: data changed on server,   │  When: columns, config,    │
│  filter/sort reset needed        │  or data source must change│
└───────────────────────────────────────────────────────────────┘
```

---

## 🔑 Refresh — Three Levels

There are three different kinds of "refresh" — each does something different:

```
Level 1 — Reload data from server (most common)
  grid.dataSource.read()
  → Sends AJAX request, gets fresh data, re-renders rows

Level 2 — Re-render rows with existing data (no AJAX)
  grid.refresh()
  → Redraws rows from already-loaded data, no server call

Level 3 — Force full rebuild (nuclear option)
  grid.destroy() + re-initialize
  → Removes widget, creates fresh with new config
```

---

## 🔑 Level 1 — Reload Data from Server

```javascript
var grid = $("#employeeGrid").data("kendoGrid");

// ── Most common: reload data after external change ────────────
grid.dataSource.read();
// Sends POST /Employee/Read with current page/sort/filter
// Updates the rows with fresh data from server

// ── After saving from a separate form ─────────────────────────
$.post("/Employee/Create", formData, function(response) {
    if (response.success) {
        showNotification("Saved!");
        grid.dataSource.read();  // ← refresh the grid
    }
});

// ── Reload and jump to first page ─────────────────────────────
grid.dataSource.page(1);        // goes to page 1 AND reads data
// OR:
grid.dataSource.query({         // reset everything at once
    page:     1,
    pageSize: 10,
    sort:     [],
    filter:   {}
});

// ── Reload and keep current page/sort/filter ──────────────────
grid.dataSource.read();         // reads with current state intact

// ── Reload silently (no loading indicator) ───────────────────
grid.dataSource.fetch(function() {
    console.log("Data refreshed — total:", grid.dataSource.total());
});
```

---

## 🔑 Level 2 — Re-Render Without AJAX

```javascript
// Re-draws rows using data already in the DataSource
// Useful when you've updated dataItems locally

var grid = $("#employeeGrid").data("kendoGrid");

// Modify a data item directly
var item = grid.dataSource.get(5);  // get record with id=5
item.set("Department", "Finance");  // update locally

// Re-render rows (no AJAX — just redraws)
grid.refresh();
```

---

## 🔑 Destroy — Complete Widget Removal

```javascript
var grid = $("#employeeGrid").data("kendoGrid");

// Destroys the Kendo Grid widget:
//   → Removes all event handlers
//   → Removes all Kendo CSS classes
//   → Leaves the original <div id="employeeGrid"> empty in the DOM
grid.destroy();

// After destroy, the element is a plain empty div
// $("#employeeGrid").data("kendoGrid") returns undefined
```

---

## 🔑 The Most Important Use Case — Rebuild with New Config

This is the main reason to use `destroy()`. If you need to change columns or DataSource configuration at runtime, you MUST destroy first:

```javascript
// ── WRONG — trying to change columns without destroy ──────────
var grid = $("#employeeGrid").data("kendoGrid");
// Can't just change columns on an existing grid — it's already rendered
// The change won't take effect properly

// ── CORRECT — destroy then rebuild ───────────────────────────
function rebuildGridForDepartment(departmentId) {

    // Step 1: Destroy existing grid
    var existing = $("#employeeGrid").data("kendoGrid");
    if (existing) {
        existing.destroy();
        $("#employeeGrid").empty();  // clear leftover HTML
    }

    // Step 2: Rebuild with new configuration
    $("#employeeGrid").kendoGrid({
        dataSource: {
            transport: {
                read: {
                    url:  "/Employee/Read",
                    type: "POST",
                    data: { departmentId: departmentId }  // new filter
                }
            },
            schema: { data: "Data", total: "Total", model: { id: "Id" } },
            pageSize: 10,
            serverPaging: true,
            serverSorting: true
        },
        columns: [
            { field: "Id",         title: "#",         width: 60  },
            { field: "Name",       title: "Name",      width: 200 },
            { field: "Salary",     title: "Salary",    width: 130,
              format: "{0:C0}" }
        ],
        pageable:   true,
        sortable:   true,
        filterable: true,
        editable:   "popup"
    });
}

// Call it when user selects a department
$("#DepartmentDdl").on("change", function() {
    rebuildGridForDepartment($(this).val());
});
```

---

## 🔑 Real Pattern — Secondary Detail Grid

The most common real use case for destroy+rebuild: a master grid with a detail grid below it.
When user selects a row in the master, the detail grid must show data for that row.

```javascript
var masterGrid;
var detailGrid;

$(function() {
    masterGrid = $("#masterGrid").data("kendoGrid");

    // When a row in master grid is clicked
    masterGrid.bind("change", function() {
        var selected = this.select();
        if (!selected.length) return;

        var employee = this.dataItem(selected[0]);
        loadDetailGrid(employee.Id, employee.Name);
    });
});

function loadDetailGrid(employeeId, employeeName) {

    // Step 1: Destroy old detail grid if it exists
    var existing = $("#detailGrid").data("kendoGrid");
    if (existing) {
        existing.destroy();
        $("#detailGrid").empty();
    }

    // Step 2: Show the container
    $("#detailSection").show();
    $("#detailTitle").text("Projects for: " + employeeName);

    // Step 3: Build fresh detail grid for this employee
    $("#detailGrid").kendoGrid({
        dataSource: {
            transport: {
                read: {
                    url:  "/Project/GetByEmployee",
                    type: "POST",
                    data: { employeeId: employeeId }   // key parameter
                }
            },
            schema:      { data: "Data", total: "Total" },
            pageSize:    5,
            serverPaging: true
        },
        columns: [
            { field: "Title",      title: "Project",    width: 250 },
            { field: "Status",     title: "Status",     width: 120 },
            { field: "StartDate",  title: "Started",    width: 120,
              format: "{0:MM/dd/yyyy}" },
            { field: "Budget",     title: "Budget",     width: 130,
              format: "{0:C0}" }
        ],
        pageable: true,
        sortable: true,
        height:   300
    });
}
```

---

## 🔑 Refresh After External Modal Save

When you use a Kendo Window or custom modal to edit data outside the grid:

```javascript
// Open a custom edit window
function openEditWindow(empId) {
    var win = $("#editWindow").data("kendoWindow");

    // Load the form
    win.refresh({ url: "/Employee/EditForm/" + empId });
    win.open().center();
}

// When modal saves successfully — refresh the grid
function onModalSaveSuccess() {
    // Close the window
    $("#editWindow").data("kendoWindow").close();

    // Refresh the grid to show updated data
    var grid = $("#employeeGrid").data("kendoGrid");
    grid.dataSource.read();

    showNotification("Employee updated", "success");
}
```

---

## 🔑 Clearing Filters and Sorting Before Refresh

```javascript
var grid = $("#employeeGrid").data("kendoGrid");
var ds   = grid.dataSource;

// ── Clear filter only, keep sorting and page ──────────────────
ds.filter({});

// ── Clear sort only ───────────────────────────────────────────
ds.sort({});

// ── Reset everything to initial state ─────────────────────────
ds.query({
    page:     1,
    pageSize: 10,
    sort:     [],
    filter:   {},
    group:    []
});

// ── Helpful Reset button ──────────────────────────────────────
$("#resetBtn").on("click", function() {
    var grid = $("#employeeGrid").data("kendoGrid");
    grid.dataSource.query({
        page: 1, pageSize: 10,
        sort: [], filter: {}, group: []
    });
});
```

---

## 📊 Method Quick Reference

| Method                                           | What It Does                       | When to Use                         |
| ------------------------------------------------ | ---------------------------------- | ----------------------------------- |
| `ds.read()`                                    | Reload data from server            | After external create/update/delete |
| `ds.page(1)`                                   | Go to page 1 (triggers read)       | After applying a new filter         |
| `ds.filter({})`                                | Clear all filters                  | Reset button                        |
| `ds.query({...})`                              | Reset page + sort + filter at once | Full reset                          |
| `grid.refresh()`                               | Re-render rows, no AJAX            | After local dataItem change         |
| `grid.destroy()`                               | Remove widget completely           | Before rebuilding with new config   |
| `grid.destroy()`+`$("#id").empty()`+ re-init | Full rebuild                       | Changing columns or data source     |

---

## ⚠️ Common Mistakes

| Mistake                                             | Symptom                                   | Fix                                                  |
| --------------------------------------------------- | ----------------------------------------- | ---------------------------------------------------- |
| Calling `kendoGrid()`again without `destroy()`  | Grid renders twice or behaves incorrectly | Always `destroy()`+`empty()`before re-init       |
| Not calling `$("#id").empty()`after `destroy()` | Leftover HTML causes rendering issues     | Always `empty()`after `destroy()`                |
| Calling `grid.refresh()`instead of `ds.read()`  | Old data shown — no server call made     | Use `ds.read()`when you need fresh server data     |
| Rebuilding on every filter change (overkill)        | Slow, loses scroll position               | Use `ds.filter({...})`instead — no rebuild needed |

---

## ❓ Interview Questions

**Q: What is the difference between `grid.dataSource.read()` and `grid.refresh()`?**

> `dataSource.read()` sends an AJAX request to the server, gets fresh data, and re-renders the grid. `grid.refresh()` just re-renders the rows using data already loaded in the DataSource — no server call. Use `read()` when the server data may have changed.

**Q: Why do you need to call `destroy()` before reinitialising a grid?**

> Calling `kendoGrid()` on an element that already has a grid creates a second widget on the same element — event handlers stack up, the UI breaks. `destroy()` cleanly removes the existing widget so the element is a plain div again, ready for a fresh init.

**Q: When would you use destroy and rebuild instead of just refreshing data?**

> When the grid configuration itself must change — different columns, different DataSource URL, different edit mode. Data refreshes keep the same config. Only destroy+rebuild changes the structure.

**Q: How do you reset a grid back to its initial state (page 1, no filters, no sort)?**

> Call `grid.dataSource.query({ page: 1, pageSize: 10, sort: [], filter: {}, group: [] })` — this resets all state and triggers a fresh server read in one call.
>
