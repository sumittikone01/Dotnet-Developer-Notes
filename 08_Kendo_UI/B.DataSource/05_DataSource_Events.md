
# 05 — DataSource Events

---

## 🎯 One-Line Definition

> **DataSource events let you run your own code at specific moments — when data arrives, when a request starts, when an error happens, or when the data changes.**

---

## 🔑 Why Events Exist

The DataSource runs silently in the background. Events are how you hook into that process:

```
DataSource lifecycle → your code runs at any of these points:

  requestStart   ← request is about to be sent
       │
       ▼
  [AJAX request travels to server]
       │
       ▼
  requestEnd     ← response received (success OR error)
       │
       ├── success path:
       │       change  ← data has changed (new data loaded, or item added/removed)
       │
       └── error path:
               error   ← something went wrong
```

---

## 🔑 The Events — One by One

### `requestStart` — fires just before any AJAX request

Use it to: show a loading spinner.

```javascript
var dataSource = new kendo.data.DataSource({
    transport: { read: { url: "/Employee/Read", type: "POST" } },

    requestStart: function(e) {
        // e.type = "read" / "create" / "update" / "destroy"
        kendo.ui.progress($("#gridContainer"), true);  // show spinner
        console.log("Requesting:", e.type);
    }
});
```

---

### `requestEnd` — fires after every request (success or error)

Use it to: hide the loading spinner.

```javascript
requestEnd: function(e) {
    // Always fires — whether request succeeded or failed
    kendo.ui.progress($("#gridContainer"), false);  // hide spinner

    // e.type    = "read" / "create" / "update" / "destroy"
    // e.response = the raw server response object
    console.log("Request finished:", e.type);
}
```

---

### `error` — fires when a request fails

Use it to: show error messages to the user.

```javascript
error: function(e) {
    // e.status      = "error" or "timeout"
    // e.errorThrown = the error description
    // e.xhr         = the raw XMLHttpRequest object
    // e.xhr.status  = HTTP status code (400, 404, 500...)
    // e.errors      = validation errors from server (on create/update)

    var httpCode = e.xhr.status;

    if (e.errors) {
        // Validation errors returned by ToDataSourceResult with ModelState
        var messages = [];
        $.each(e.errors, function(field, detail) {
            messages.push(field + ": " + detail.errors.join(", "));
        });
        showNotification("Validation failed: " + messages.join(" | "), "error");
    }
    else if (httpCode === 401) {
        window.location.href = "/Login";
    }
    else if (httpCode === 500) {
        showNotification("Server error — please try again.", "error");
    }
    else {
        showNotification("Request failed: " + e.errorThrown, "error");
    }

    // IMPORTANT: call this to cancel the error state
    // so the Grid doesn't stay in a broken state
    this.cancelChanges();
}
```

---

### `change` — fires when data in the DataSource changes

Fires when: new data is loaded (read), an item is added, edited, or removed locally.

```javascript
change: function(e) {
    // e.action = undefined (initial load / read)
    //          = "add"     (item added)
    //          = "remove"  (item removed)
    //          = "itemchange" (item field changed)

    if (e.action === undefined) {
        // Data was freshly loaded from server
        console.log("Loaded " + this.data().length + " records");
        updateRecordCount(this.total());
    }

    if (e.action === "add") {
        console.log("New row added locally");
    }

    if (e.action === "itemchange") {
        // e.items[0] = the changed item
        // e.field    = which field changed
        console.log(e.field + " was changed on row:", e.items[0].name);
    }
}
```

---

## 🔑 Two Ways to Attach Events

### Way 1 — In the DataSource config (at creation)

```javascript
var ds = new kendo.data.DataSource({
    transport: { ... },
    requestStart: function(e) { showSpinner(); },
    requestEnd:   function(e) { hideSpinner(); },
    error:        function(e) { showError(e);  }
});
```

### Way 2 — With `.bind()` after creation (useful when grid already exists)

```javascript
// Get the existing DataSource from the Grid
var ds = $("#employeeGrid").data("kendoGrid").dataSource;

// Attach events after the fact
ds.bind("requestStart", function(e) { showSpinner(); });
ds.bind("requestEnd",   function(e) { hideSpinner(); });
ds.bind("error",        function(e) { showError(e);  });
ds.bind("change",       function(e) { updateStats(); });
```

In Tag Helper (Kendo Grid) — events go in a `<datasource>` event block:

```cshtml
<datasource type="DataSourceTagHelperType.Ajax" page-size="10">
    <transport>
        <read url="@Url.Action("Read", "Employee")" type="POST" />
    </transport>
    <events on-request-start="onRequestStart"
            on-request-end="onRequestEnd"
            on-error="onError"
            on-change="onChange" />
</datasource>

@section Scripts {
<script>
    function onRequestStart(e) { kendo.ui.progress($("#grid"), true);  }
    function onRequestEnd(e)   { kendo.ui.progress($("#grid"), false); }
    function onError(e)        { showError(e.xhr.status); }
    function onChange(e)       { console.log("data changed"); }
</script>
}
```

---

## 🔑 Real Pattern — Loading Spinner + Error Handling

This is the pattern you'll use in almost every real grid:

```javascript
var ds = new kendo.data.DataSource({

    transport: {
        read:    { url: "/Employee/Read",    type: "POST" },
        create:  { url: "/Employee/Create",  type: "POST" },
        update:  { url: "/Employee/Update",  type: "POST" },
        destroy: { url: "/Employee/Destroy", type: "POST" }
    },

    schema: {
        data: "Data", total: "Total", errors: "Errors",
        model: { id: "Id" }
    },

    pageSize: 10, serverPaging: true, serverSorting: true,

    // ── Show spinner when any request starts ──────────────────
    requestStart: function() {
        kendo.ui.progress($("#employeeGrid"), true);
    },

    // ── Hide spinner when any request ends ────────────────────
    requestEnd: function() {
        kendo.ui.progress($("#employeeGrid"), false);
    },

    // ── Handle all errors in one place ────────────────────────
    error: function(e) {
        kendo.ui.progress($("#employeeGrid"), false);  // hide spinner

        if (e.errors) {
            // Server-side validation errors (from ModelState)
            var msg = [];
            $.each(e.errors, function(key, val) {
                msg.push(val.errors.join(", "));
            });
            alert("Please fix: " + msg.join("\n"));
        } else {
            alert("Error " + e.xhr.status + ": " + e.errorThrown);
        }

        this.cancelChanges();   // reset grid to last good state
    }
});
```

---

## 🔑 Updating Dashboard Stats After Grid Changes

A common requirement: update a "Total Employees: 47" counter every time data reloads.

```javascript
// Using change event
ds.bind("change", function() {
    // this = the DataSource
    $("#totalCount").text("Total Employees: " + this.total());
});

// Using requestEnd — fires after every successful read
ds.bind("requestEnd", function(e) {
    if (e.type === "read") {
        // Refresh other widgets that depend on this data
        $("#salaryChart").data("kendoChart").dataSource.read();
    }
});
```

---

## 📊 Events Quick Reference

| Event            | When It Fires                       | Common Use                     |
| ---------------- | ----------------------------------- | ------------------------------ |
| `requestStart` | Before any AJAX request             | Show loading spinner           |
| `requestEnd`   | After any request (success or fail) | Hide loading spinner           |
| `error`        | When a request fails                | Show error message             |
| `change`       | When data in DataSource changes     | Update counters, other widgets |

---

## ⚠️ Common Mistakes

| Mistake                                                                   | What Happens                                           | Fix                                                              |
| ------------------------------------------------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------- |
| Not calling `this.cancelChanges()`in error                              | Grid stays in edit mode after a failed save            | Always call it inside the `error`handler                       |
| Showing errors in `error`but not hiding spinner                         | Spinner stays visible forever after an error           | Handle both in `error`— hide spinner AND show message         |
| Using `error`event to catch validation errors, but no `e.errors`check | Other HTTP errors misidentified as validation failures | Check `if (e.errors)`first, then fall back to `e.xhr.status` |
| Binding the same event twice                                              | Handler fires twice                                    | Use config-based events or unbind before rebinding               |

---

## ❓ Interview Questions

**Q: What is the difference between `requestEnd` and `change`?**

> `requestEnd` fires after every AJAX request finishes — read, create, update, destroy. `change` fires when the data in the DataSource actually changes — when new data is loaded or when items are added, removed, or modified locally. A failed request fires `requestEnd` but not `change`.

**Q: When would you use the `error` event?**

> To handle all AJAX failures in one place — show user-friendly error messages, handle session timeouts (401), and reset the grid to a clean state with `this.cancelChanges()`.

**Q: What does `this.cancelChanges()` do inside the `error` handler?**

> It reverts all unsaved changes in the DataSource and the Grid — the rows go back to their original values and any "dirty" state is cleared. Without it, the Grid can be stuck in a broken edit state after a failed save.

**Q: What is `e.errors` in the error event?**

> Validation errors returned from the server via `ToDataSourceResult(request, ModelState)`. It's an object where each key is a field name and the value contains an array of error messages for that field.
>
