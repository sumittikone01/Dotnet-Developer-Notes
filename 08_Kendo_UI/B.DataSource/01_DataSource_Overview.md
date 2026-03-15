
# 01 — Kendo DataSource Overview

---

## 🎯 One-Line Definition

> **The DataSource is the middleman between a Kendo widget and your server — it handles all data loading, saving, and state tracking so your widget doesn't have to.**

---

## 🤔 Why Does the DataSource Exist?

Without DataSource, every widget would need to know:

* How to make AJAX calls
* How to handle paging, sorting, filtering
* How to track which rows changed
* How to batch or sync changes back

That's too much for a Grid to care about. So Kendo separated it.

```
WITHOUT DataSource (imaginary, messy):
──────────────────────────────────────
  Grid → makes its own AJAX call
  Grid → handles paging itself
  Grid → tracks changes itself
  Grid → sends saves itself
  Chart → makes its own AJAX call
  DropDownList → makes its own AJAX call
  (every widget reinvents the same wheel)


WITH DataSource (how Kendo actually works):
────────────────────────────────────────────
  Grid         ─┐
  Chart        ─┤──► DataSource ──► Server
  DropDownList ─┘
  (one component handles all data concerns)
```

---

## 🏗️ Where DataSource Sits in the Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│   KENDO WIDGET (Grid, Chart, DropDownList...)                 │
│   "Show me the data"                                          │
│          │                                                    │
│          ▼                                                    │
│   KENDO DATASOURCE                                            │
│   ┌─────────────────────────────────────────────────────┐    │
│   │  • Knows what URL to call (transport)               │    │
│   │  • Knows what the JSON looks like (schema)          │    │
│   │  • Handles paging, sorting, filtering               │    │
│   │  • Tracks new / changed / deleted rows              │    │
│   │  • Caches data locally                              │    │
│   └─────────────────────────────────────────────────────┘    │
│          │  AJAX (JSON)                                       │
│          ▼                                                    │
│   YOUR ASP.NET CONTROLLER                                     │
│   Returns: { "Data": [...], "Total": 150 }                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 The Two Types of DataSource

| Type             | Data comes from           | When to use                                      |
| ---------------- | ------------------------- | ------------------------------------------------ |
| **Remote** | Server via AJAX           | Database data, large lists, CRUD operations      |
| **Local**  | Inline in your JavaScript | Small static lists, dropdowns with fixed options |

```javascript
// ── REMOTE DataSource — data from server ─────────────────────
var remoteDS = new kendo.data.DataSource({
    transport: {
        read: { url: "/Employee/GetAll", type: "GET" }
    }
});

// ── LOCAL DataSource — data inline ───────────────────────────
var localDS = new kendo.data.DataSource({
    data: [
        { id: 1, name: "Alice", dept: "IT"      },
        { id: 2, name: "Bob",   dept: "HR"      },
        { id: 3, name: "Carol", dept: "Finance" }
    ]
});
```

---

## 🔑 DataSource Main Configuration Areas

A DataSource has 4 main sections. Know what each one does:

```
┌─────────────────────────────────────────────────────────────┐
│  new kendo.data.DataSource({                                │
│                                                             │
│    transport: { ... }   ← WHERE to get/send data (URLs)    │
│                                                             │
│    schema:   { ... }    ← WHAT the JSON looks like         │
│                                                             │
│    pageSize: 10,        ← HOW MANY records per page        │
│    serverPaging: true,  ← WHO handles paging (server/client│
│    serverSorting: true, ← WHO handles sorting              │
│    serverFiltering: true← WHO handles filtering            │
│                                                             │
│  });                                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔑 Full DataSource Example with Explanation

```javascript
var employeeDataSource = new kendo.data.DataSource({

    // ── TRANSPORT: tell DataSource which URLs to call ─────────
    transport: {
        read: {
            url:  "/Employee/Read",
            type: "POST"
        },
        create: {
            url:  "/Employee/Create",
            type: "POST"
        },
        update: {
            url:  "/Employee/Update",
            type: "POST"
        },
        destroy: {
            url:  "/Employee/Destroy",
            type: "POST"
        }
    },

    // ── SCHEMA: tell DataSource what the JSON response looks like
    schema: {
        data:   "Data",    // the array of records is in response.Data
        total:  "Total",   // total record count is in response.Total
        errors: "Errors",  // validation errors from server
        model: {
            id: "id",      // which field is the primary key
            fields: {
                id:         { type: "number",  editable: false },
                name:       { type: "string"                   },
                department: { type: "string"                   },
                salary:     { type: "number"                   },
                hireDate:   { type: "date"                     },
                isActive:   { type: "boolean"                  }
            }
        }
    },

    // ── PAGING ────────────────────────────────────────────────
    pageSize:        10,
    serverPaging:    true,   // server handles paging (recommended for large data)
    serverSorting:   true,   // server handles sorting
    serverFiltering: true    // server handles filtering

});
```

---

## 🔑 Server-Side vs Client-Side Operations

This is a critical decision you must understand:

```
serverPaging: true         serverPaging: false
─────────────────────      ───────────────────────────────
Request page 1 of 10  →   Load ALL 10,000 records at once
Server returns 10 rows     Browser stores all 10,000 in RAM
Fast ✅                    Slow for large data ❌
Low memory ✅              High memory usage ❌
Correct total count ✅     Works fine for < 200 rows ✅
```

**Rule of thumb:**

* More than ~200 rows → use `serverPaging: true`
* Small static list → `serverPaging: false` is fine (simpler)

```javascript
// ── SERVER-SIDE (recommended for database data) ──────────────
var ds = new kendo.data.DataSource({
    transport: { read: { url: "/Employee/Read", type: "POST" } },
    schema:    { data: "Data", total: "Total" },
    pageSize:        10,
    serverPaging:    true,   // ← server returns only 10 rows
    serverSorting:   true,   // ← server sorts before returning
    serverFiltering: true    // ← server filters before returning
});

// ── CLIENT-SIDE (for small local/static data) ────────────────
var ds = new kendo.data.DataSource({
    data:     [ {id:1, name:"Alice"}, {id:2, name:"Bob"} ],
    pageSize: 5
    // no serverPaging needed — DataSource handles it locally
});
```

---

## 🔑 schema.data and schema.total — Why They Matter

Your controller returns this:

```json
{
  "Data":  [ {"id":1,"name":"Alice"}, {"id":2,"name":"Bob"} ],
  "Total": 150,
  "Errors": null
}
```

Without `schema.data` and `schema.total`, DataSource has no idea where the array is or how many total records exist.

```javascript
schema: {
    data:  "Data",    // ← "the records array is in response.Data"
    total: "Total"    // ← "the total count is in response.Total"
}

// If your controller returns the array directly (no wrapper):
// [ {"id":1}, {"id":2} ]
// Then you don't need schema.data — DataSource finds it automatically
```

---

## 🔑 DataSource in Tag Helper (Kendo Grid)

You configure DataSource inside the Grid tag — this is what you use daily:

```cshtml
<kendo-grid name="employeeGrid">

    <datasource type="DataSourceTagHelperType.Ajax" page-size="10">

        <transport>
            <read    url="@Url.Action("Read",    "Employee")" type="POST" />
            <create  url="@Url.Action("Create",  "Employee")" type="POST" />
            <update  url="@Url.Action("Update",  "Employee")" type="POST" />
            <destroy url="@Url.Action("Destroy", "Employee")" type="POST" />
        </transport>

        <schema>
            <model id="Id">
                <fields>
                    <field name="Id"         type="number"  editable="false" />
                    <field name="Name"        type="string"                   />
                    <field name="Department"  type="string"                   />
                    <field name="Salary"      type="number"                   />
                    <field name="HireDate"    type="date"                     />
                    <field name="IsActive"    type="boolean"                  />
                </fields>
            </model>
        </schema>

    </datasource>

    <pageable />
    <sortable enabled="true" />

</kendo-grid>
```

---

## 🔑 Key DataSource Methods (JavaScript API)

After the grid/widget is created, you can control the DataSource from JavaScript:

```javascript
// Get reference to grid's DataSource
var grid = $("#employeeGrid").data("kendoGrid");
var ds   = grid.dataSource;

// ── Loading data ─────────────────────────────────────────────
ds.read();                    // fetch fresh data from server
ds.fetch();                   // same as read() but only if not loaded yet

// ── Navigating pages ─────────────────────────────────────────
ds.page(2);                   // jump to page 2
ds.page();                    // get current page number
ds.pageSize(20);              // change page size
ds.totalPages();              // how many pages total

// ── Counts ───────────────────────────────────────────────────
ds.total();                   // total records (from server's Total field)
ds.data().length;             // records currently loaded on this page

// ── Syncing changes back to server ───────────────────────────
ds.sync();                    // send all pending create/update/destroy

// ── Working with records ─────────────────────────────────────
ds.add({ name: "New Person" });    // add a new (unsaved) record
ds.remove(dataItem);               // mark a record for deletion
ds.get(5);                         // get record by primary key (id=5)
ds.getByUid("abc123");             // get record by Kendo's internal UID
```

---

## ⚠️ Most Common DataSource Mistakes

| Mistake                                        | Symptom                                               | Fix                                              |
| ---------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------ |
| Missing `schema.data`                        | Grid shows 0 rows even though controller returns data | Add `schema: { data: "Data", total: "Total" }` |
| `serverPaging: false`on large data           | Page loads very slowly                                | Switch to `serverPaging: true`                 |
| Missing `schema.model id`                    | Edit/delete operations target wrong row               | Add `model: { id: "Id" }`                      |
| Wrong field types in model                     | Date shows as number, booleans don't work             | Set correct `type:`for each field              |
| Not calling `ds.read()`after external change | Grid shows stale data                                 | Call `ds.read()`to force refresh               |

---

## ❓ Interview Questions

**Q: What is the Kendo DataSource?**

> A standalone JavaScript component that handles all data communication between a Kendo widget and the server — AJAX calls, paging, sorting, filtering, and change tracking.

**Q: What is the difference between `serverPaging: true` and `serverPaging: false`?**

> With `serverPaging: true`, the server returns only one page of records at a time (e.g., 10 rows) — fast and memory-efficient for large datasets. With `serverPaging: false`, all records are loaded at once and the browser handles paging — only suitable for small datasets.

**Q: What does `schema.data` do?**

> It tells the DataSource which property in the JSON response contains the array of records. If your server returns `{ "Data": [...], "Total": 50 }`, you set `schema.data = "Data"` so DataSource knows where to find the array.

**Q: What does `ds.sync()` do?**

> It sends all pending changes (new rows, edited rows, deleted rows) to the server in one operation by calling the configured create, update, and destroy URLs.
>
