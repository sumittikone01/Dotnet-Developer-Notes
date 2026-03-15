
# 04 — Destroy and Recreate Grid

---

## 🎯 One-Line Definition

> **Destroy removes the Kendo Grid widget completely from a DOM element. Recreate builds a fresh one on the same element — with a completely new configuration. Used when columns, data source, or structure must change at runtime.**

---

## 🔑 Why Destroy Exists — The Core Problem

```
A Kendo Grid is NOT just HTML — it's a JavaScript widget object
attached to a DOM element.

When you call $("#myGrid").kendoGrid({...}):
  → JavaScript widget object created in memory
  → DOM element filled with Grid HTML
  → Event handlers attached
  → DataSource initialised

The PROBLEM:
  You can't just change columns or DataSource by calling kendoGrid() again.
  The old widget is still there.
  Calling kendoGrid() a second time creates a SECOND widget on the same element.
  → Two sets of event handlers (everything fires twice)
  → Visual glitches
  → Unpredictable behaviour
```

---

## 🔑 Refresh vs Destroy — Choose the Right Tool

```
┌───────────────────────────────────────────────────────────────────┐
│  Use REFRESH when:               Use DESTROY + RECREATE when:     │
│  ────────────────────────        ─────────────────────────────    │
│  Data changed on server          Columns must change               │
│  Filter needs applying           DataSource URL must change        │
│  Sort needs resetting            Edit mode must change             │
│  Page needs changing             Different schema needed           │
│                                  Showing different entity type     │
│  How:                            How:                              │
│  grid.dataSource.read()          grid.destroy()                   │
│  grid.dataSource.filter({})      $("#id").empty()                 │
│  grid.dataSource.page(1)         rebuild with new config          │
│                                                                     │
│  FAST — same widget              COMPLETE RESET — brand new       │
└───────────────────────────────────────────────────────────────────┘
```

---

## 🔑 The Correct Destroy Pattern — Every Time

```javascript
function destroyAndRebuild(newConfig) {

    // ── Step 1: Check if widget exists ───────────────────────
    var existing = $("#employeeGrid").data("kendoGrid");

    if (existing) {
        // ── Step 2: Destroy the widget ───────────────────────
        existing.destroy();
        //  → removes all Kendo event handlers
        //  → removes DataSource
        //  → removes the widget object from jQuery data
        //  → DOM element still exists but is empty-ish

        // ── Step 3: Clear leftover HTML ───────────────────────
        $("#employeeGrid").empty();
        //  → removes ALL child HTML Kendo generated
        //  → leaves: <div id="employeeGrid"></div>  (clean)

        // If you skip .empty(), leftover HTML can cause visual issues
    }

    // ── Step 4: Rebuild with new configuration ────────────────
    $("#employeeGrid").kendoGrid(newConfig);
}
```

> **Always do all 3 steps: `destroy()` → `empty()` → `kendoGrid()`.**
> Skipping any one of them causes problems.

---

## 🔑 What `destroy()` Does and Doesn't Do

```
destroy() DOES:
  ✅ Removes all event handlers attached by Kendo
  ✅ Removes the widget instance from jQuery .data()
  ✅ Stops any pending AJAX requests
  ✅ Cleans up DataSource subscriptions

destroy() DOES NOT:
  ❌ Remove the <div id="employeeGrid"> itself from the DOM
  ❌ Remove the HTML Kendo generated inside the div
     (you must call .empty() for this)
  ❌ Remove event handlers you attached yourself with .on()
     (if you used $(document).on("click", ".editBtn", ...) those persist)
```

---

## 🔑 Most Common Use Case — Master-Detail Grid

The most common real-world reason to destroy and recreate:
clicking a row in a master grid loads a detail grid for that entity.

```
MASTER GRID (always there):
┌────┬───────────────┬────────────┐
│ #  │ Department    │ Manager    │
├────┼───────────────┼────────────┤
│ 1  │ IT            │ Alice      │  ← click → load IT employees below
│ 2  │ HR            │ Bob        │  ← click → load HR employees below
│ 3  │ Finance       │ Carol      │
└────┴───────────────┴────────────┘

DETAIL GRID (changes on each click):
┌────┬───────────────┬──────────┐
│ #  │ Name          │ Salary   │
├────┼───────────────┼──────────┤
│    IT employees load here...  │
└────┴───────────────┴──────────┘
```

```javascript
var masterGrid = $("#masterGrid").data("kendoGrid");

masterGrid.bind("change", function() {
    var row        = this.select();
    if (!row.length) return;
    var department = this.dataItem(row[0]);

    loadDetailGrid(department.Id, department.Name);
});


function loadDetailGrid(deptId, deptName) {

    // STEP 1 — Destroy existing detail grid
    var existing = $("#detailGrid").data("kendoGrid");
    if (existing) {
        existing.destroy();
        $("#detailGrid").empty();
    }

    // STEP 2 — Show the detail section heading
    $("#detailSection").show();
    $("#detailTitle").text("Employees in " + deptName);

    // STEP 3 — Build fresh grid for this department
    $("#detailGrid").kendoGrid({
        dataSource: {
            transport: {
                read: {
                    url:  "/Employee/Read",
                    type: "POST",
                    data: { departmentId: deptId }   // ← new parameter
                },
                create:  { url: "/Employee/Create",  type: "POST" },
                update:  { url: "/Employee/Update",  type: "POST" },
                destroy: { url: "/Employee/Destroy", type: "POST" }
            },
            schema: {
                data:  "Data",
                total: "Total",
                model: {
                    id: "Id",
                    fields: {
                        Id:         { type: "number", editable: false },
                        Name:       { type: "string" },
                        Salary:     { type: "number" },
                        HireDate:   { type: "date"   },
                        IsActive:   { type: "boolean"}
                    }
                }
            },
            pageSize:     5,
            serverPaging: true
        },
        columns: [
            { field: "Id",       title: "#",         width: 60  },
            { field: "Name",     title: "Name",       width: 200 },
            { field: "Salary",   title: "Salary",     width: 130,
              format: "{0:C0}" },
            { field: "HireDate", title: "Hired",      width: 120,
              format: "{0:MM/dd/yyyy}" },
            { title: "Actions",  width: 160,
              command: ["edit", "destroy"] }
        ],
        toolbar:    ["create"],
        editable:   "popup",
        pageable:   true,
        sortable:   true,
        filterable: false,
        height:     300
    });
}
```

---

## 🔑 Switching Between Different Data Types

Another real use case: one grid container used for different entities
depending on what the user selects:

```javascript
// One page, one grid container, different data based on tab
$(".entity-tab").on("click", function() {
    var entityType = $(this).data("entity");  // "employees", "projects", "invoices"
    loadGridForEntity(entityType);
});


function loadGridForEntity(entityType) {

    // Destroy existing
    var existing = $("#mainGrid").data("kendoGrid");
    if (existing) {
        existing.destroy();
        $("#mainGrid").empty();
    }

    // Different config per entity type
    var configs = {

        employees: {
            dataSource: {
                transport: { read: { url: "/Employee/Read", type: "POST" } },
                schema: {
                    data: "Data", total: "Total",
                    model: { id: "Id", fields: {
                        Id: { type: "number", editable: false },
                        Name: { type: "string" }, Salary: { type: "number" }
                    }}
                },
                pageSize: 10, serverPaging: true
            },
            columns: [
                { field: "Id",     width: 60  },
                { field: "Name",   width: 220 },
                { field: "Salary", width: 130, format: "{0:C0}" },
                { command: ["edit", "destroy"], title: "Actions", width: 150 }
            ],
            toolbar:  ["create"],
            editable: "popup",
            pageable: true, sortable: true
        },

        projects: {
            dataSource: {
                transport: { read: { url: "/Project/Read", type: "POST" } },
                schema: {
                    data: "Data", total: "Total",
                    model: { id: "ProjectId", fields: {
                        ProjectId: { type: "number", editable: false },
                        Title:     { type: "string" },
                        Budget:    { type: "number" },
                        StartDate: { type: "date"   }
                    }}
                },
                pageSize: 10, serverPaging: true
            },
            columns: [
                { field: "ProjectId", title: "#",       width: 60  },
                { field: "Title",     title: "Project", width: 250 },
                { field: "Budget",    width: 130, format: "{0:C0}" },
                { field: "StartDate", width: 130, format: "{0:MM/dd/yyyy}" }
            ],
            pageable: true, sortable: true
        },

        invoices: {
            dataSource: {
                transport: { read: { url: "/Invoice/Read", type: "POST" } },
                schema: {
                    data: "Data", total: "Total",
                    model: { id: "InvoiceId" }
                },
                pageSize: 15, serverPaging: true
            },
            columns: [
                { field: "InvoiceNumber", title: "Invoice #", width: 140 },
                { field: "ClientName",    title: "Client",    width: 220 },
                { field: "Amount",        width: 130, format: "{0:C0}" },
                { field: "DueDate",       width: 130, format: "{0:MM/dd/yyyy}" }
            ],
            pageable: true, sortable: true, filterable: true
        }
    };

    $("#mainGrid").kendoGrid(configs[entityType]);
}
```

---

## 🔑 Reusable Helper Function

Build this once, use it everywhere — makes the pattern one call:

```javascript
// Add to your shared JS file
function rebuildGrid(elementId, config) {
    var $el      = $("#" + elementId);
    var existing = $el.data("kendoGrid");

    if (existing) {
        existing.destroy();
        $el.empty();
    }

    $el.kendoGrid(config);
    return $el.data("kendoGrid");  // return reference for chaining
}


// Usage anywhere:
var grid = rebuildGrid("employeeGrid", {
    dataSource: {
        transport: { read: { url: "/Employee/Read", type: "POST",
                             data: { deptId: selectedDeptId } } },
        schema: { data: "Data", total: "Total", model: { id: "Id" } },
        pageSize: 10, serverPaging: true
    },
    columns: [
        { field: "Id",   width: 60  },
        { field: "Name", width: 220 },
        { command: ["edit", "destroy"], title: "Actions", width: 150 }
    ],
    toolbar:  ["create"],
    editable: "popup",
    pageable: true
});

// Use the returned reference:
grid.dataSource.read();
```

---

## 🔑 After Rebuild — Re-Attach Events

If your code binds events to the grid widget, re-attach after rebuild:

```javascript
function rebuildAndBindEvents(deptId) {
    var grid = rebuildGrid("detailGrid", buildConfig(deptId));

    // Re-attach events every time
    grid.bind("dataBound", function() {
        updateRecordCount(this.dataSource.total());
        highlightNewRecords(this);
    });

    grid.bind("edit", function(e) {
        if (e.model.isNew()) {
            e.model.set("DepartmentId", deptId);  // set default for new records
        }
    });

    grid.dataSource.bind("error", function(e) {
        handleGridError(e);
    });
}
```

---

## 🔑 Avoid Recreating When You Don't Need To

Destroy + recreate is a heavy operation. Many situations only need a refresh:

```javascript
// ✅ Only need a DATA refresh? Use read() — much faster:
grid.dataSource.read();

// ✅ Only need to change filter? No rebuild needed:
grid.dataSource.filter({ field: "DeptId", operator: "eq", value: deptId });

// ✅ Only need to change the read URL parameter? No rebuild needed:
grid.dataSource.transport.options.read.data = { deptId: deptId };
grid.dataSource.read();

// ❌ Need different columns or completely different entity? Use rebuild.
```

---

## 📊 Destroy Pattern Quick Reference

```
GOAL                               APPROACH
──────────────────────────────     ──────────────────────────────────────
Same grid, new page of data        ds.page(newPage)
Same grid, new filter              ds.filter({...})
Same grid, fresh data from server  ds.read()
Same grid, clear all state         ds.query({page:1, sort:[], filter:{}})
New columns or new entity type     destroy() → empty() → kendoGrid({...})
Master → detail grid on row click  destroy() → empty() → kendoGrid({...})
Show one grid for multiple uses    destroy() → empty() → kendoGrid({...})
```

---

## ⚠️ Common Mistakes

| Mistake                                            | Symptom                                                        | Fix                                                                 |
| -------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- |
| Calling `kendoGrid()`again without `destroy()` | Grid initialises twice, all events fire twice, visual glitches | Always `destroy()`+`empty()`first                               |
| `destroy()`but no `empty()`                    | Leftover HTML from old grid causes styling issues              | Always call `empty()`after `destroy()`                          |
| Rebuilding on every data filter change             | Slow and loses scroll position                                 | Use `ds.filter({...})`for filter changes — no rebuild needed     |
| Not re-attaching event handlers after rebuild      | Events (dataBound, edit) don't fire on new grid                | Bind events again after `kendoGrid()`creates the new widget       |
| Calling `destroy()`on `undefined`              | JS error: "Cannot read properties of undefined"                | Always check:`var g = $().data("kendoGrid"); if (g) g.destroy();` |

---

## ❓ Interview Questions

**Q: What does `grid.destroy()` do?**

> It removes the Kendo Grid widget — detaches all event handlers Kendo added, removes the DataSource, and clears the widget reference from jQuery's data store. The DOM element itself remains but needs `.empty()` called to remove the generated HTML.

**Q: Why must you call `.empty()` after `destroy()`?**

> `destroy()` removes the JavaScript widget but leaves the HTML that Kendo generated inside the element. Without `.empty()`, that stale HTML remains and causes visual glitches when the new grid renders on top of it.

**Q: What happens if you call `kendoGrid()` on an element that already has a grid?**

> A second grid widget is created on the same element. Now you have two overlapping grids — both respond to events, both render rows, both make AJAX calls. Everything fires twice and the display breaks. Always `destroy()` first.

**Q: When should you destroy and recreate vs just refreshing the DataSource?**

> Destroy and recreate when the grid's structure must change — different columns, different entity, different edit mode, or different schema. Just refresh (`ds.read()`, `ds.filter()`) when only the data changes — same columns, same entity, just new or filtered records.
>
