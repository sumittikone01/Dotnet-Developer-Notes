
# 05 — jQuery AJAX Methods

---

## 🎯 One-Line Definition

> **jQuery's AJAX methods — `$.get`, `$.post`, `$.ajax`, `$.getJSON` — are the simplest way to send requests to your ASP.NET controller and handle the response, all in a few lines.**

---

## 🗺️ The Four Methods — When to Use Which

```
┌──────────────────────────────────────────────────────────────┐
│  $.get(url, callback)                                        │
│  → Simplest. GET request. For loading data.                 │
│  → Use when: reading a list, loading dropdown options       │
├──────────────────────────────────────────────────────────────┤
│  $.post(url, data, callback)                                 │
│  → Simple. POST request. For saving data.                   │
│  → Use when: creating, updating, deleting                   │
├──────────────────────────────────────────────────────────────┤
│  $.getJSON(url, callback)                                    │
│  → GET request that auto-parses JSON.                       │
│  → Use when: loading JSON data simply                       │
├──────────────────────────────────────────────────────────────┤
│  $.ajax({ ... })                                             │
│  → Full control. Any method. Any headers. Any options.      │
│  → Use when: you need custom headers, error handling,       │
│              content-type, or non-standard setup            │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 `$.get()` — Load Data from Server

```javascript
// Basic form:
$.get(url, callback)
$.get(url, data, callback)

// ── Simple load ───────────────────────────────────────────────
$.get("/Employee/GetAll", function(data) {
    // data = parsed JavaScript array/object (jQuery auto-parses JSON)
    console.log(data);
    renderTable(data);
});

// ── Load with parameters (sent as query string) ───────────────
$.get("/Employee/GetByDept", { departmentId: 3 }, function(data) {
    // Sends:  GET /Employee/GetByDept?departmentId=3
    renderTable(data);
});

// ── With error handling ───────────────────────────────────────
$.get("/Employee/GetAll")
    .done(function(data) {
        renderTable(data);            // success
    })
    .fail(function(xhr) {
        showError("Load failed: " + xhr.status);  // failure
    })
    .always(function() {
        hideSpinner();                // always runs
    });
```

**Controller to match:**

```csharp
public JsonResult GetAll()
{
    var employees = _db.Employees.ToList();
    return Json(employees);
}

public JsonResult GetByDept(int departmentId)
{
    var employees = _db.Employees
        .Where(e => e.DepartmentId == departmentId)
        .ToList();
    return Json(employees);
}
```

---

## 🔑 `$.post()` — Send Data to Server

```javascript
// Basic form:
$.post(url, data, callback)

// ── Save a new record ─────────────────────────────────────────
var employee = {
    name:       $("#Name").val(),
    department: $("#Department").val(),
    salary:     parseFloat($("#Salary").val())
};

$.post("/Employee/Create", employee, function(response) {
    // response = whatever your controller returned as JSON
    if (response.success) {
        showNotification("Employee saved!");
        refreshGrid();
    } else {
        showError(response.message);
    }
});

// ── Delete a record ───────────────────────────────────────────
$.post("/Employee/Delete", { id: empId }, function(response) {
    if (response.success) {
        $("tr[data-id='" + empId + "']").remove();
    }
});

// ── With .done() / .fail() ────────────────────────────────────
$.post("/Employee/Update", formData)
    .done(function(response) {
        showNotification("Updated successfully");
        refreshGrid();
    })
    .fail(function(xhr) {
        showError("Update failed: " + xhr.status);
    })
    .always(function() {
        $("#saveBtn").prop("disabled", false);  // re-enable button
    });
```

**Controller to match:**

```csharp
[HttpPost]
public JsonResult Create(Employee employee)
{
    if (!ModelState.IsValid)
        return Json(new { success = false, message = "Validation failed" });

    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true, id = employee.Id });
}
```

---

## 🔑 `$.getJSON()` — GET That Auto-Parses JSON

```javascript
// Exactly like $.get() but always expects JSON back
// No need to specify dataType

$.getJSON("/Employee/GetAll", function(data) {
    // data is already a JS object/array
    populateGrid(data);
});

$.getJSON("/Department/GetDropdown", { activeOnly: true }, function(depts) {
    depts.forEach(function(dept) {
        $("#deptDdl").append(
            $("<option>").val(dept.id).text(dept.name)
        );
    });
});
```

> `$.getJSON` is equivalent to `$.get` with `dataType: "json"`. Pick whichever is clearer.

---

## 🔑 `$.ajax()` — Full Control

Use `$.ajax()` when you need things `$.get` and `$.post` can't do simply — custom headers, specific content types, custom timeout.

```javascript
// Full structure:
$.ajax({
    url:         "/Employee/Create",   // where to send
    type:        "POST",               // HTTP method
    contentType: "application/json",   // what format the BODY is
    dataType:    "json",               // what format you expect BACK
    data:        JSON.stringify(emp),  // the data (stringified for JSON)
    timeout:     10000,                // fail after 10 seconds

    success: function(response) {
        // Fired when status is 2xx
        showNotification("Saved!");
    },

    error: function(xhr, status, error) {
        // Fired for any non-2xx status
        handleError(xhr.status, xhr.responseJSON);
    },

    complete: function(xhr, status) {
        // Always fires — after success OR error
        hideSpinner();
    }
});
```

---

## 🔑 `contentType` vs `dataType` — Know the Difference

This confuses everyone. Memorise it:

```
contentType = "what format am I SENDING to the server"
dataType    = "what format am I EXPECTING back from the server"

                 YOU                          SERVER
                 ───                          ──────
contentType: → "my body is JSON"    →    [FromBody] reads it
dataType:    ← "I want JSON back"   ←    return Json(...)
```

```javascript
// Sending JSON to controller ([FromBody] model)
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",      // ← "I'm sending JSON"
    dataType:    "json",                  // ← "I want JSON back"
    data:        JSON.stringify(employee) // ← stringify for JSON body
});


// Sending form data to controller (normal model binding)
$.ajax({
    url:      "/Employee/Create",
    type:     "POST",
    // contentType defaults to "application/x-www-form-urlencoded"
    // ← "I'm sending form fields"
    dataType: "json",                     // ← "I want JSON back"
    data:     { name: "Alice", dept: "IT" }  // plain object, no stringify
});
```

---

## 🔑 `.done()`, `.fail()`, `.always()` — Promise Style

`$.ajax`, `$.get`, `$.post`, and `$.getJSON` all return a **jqXHR object** which works like a Promise:

```javascript
$.get("/Employee/GetAll")
    .done(function(data, textStatus, jqXHR) {
        // Success — jqXHR.status = 200
        renderTable(data);
    })
    .fail(function(jqXHR, textStatus, errorThrown) {
        // Failure — jqXHR.status = 400, 404, 500...
        var code = jqXHR.status;
        var body = jqXHR.responseJSON;   // parsed error response if JSON
        handleError(code, body);
    })
    .always(function() {
        // Always — whether success or failure
        hideSpinner();
        $("#saveBtn").prop("disabled", false);
    });

// ── Chaining is cleaner than nesting ──────────────────────────
// ❌ Nesting callbacks (messy):
$.get("/api/user/1", function(user) {
    $.get("/api/dept/" + user.deptId, function(dept) {
        console.log(dept);   // pyramid
    });
});

// ✅ Chaining with .then():
$.get("/api/user/1")
    .then(function(user) {
        return $.get("/api/dept/" + user.deptId);
    })
    .then(function(dept) {
        console.log(dept);   // flat
    });
```

---

## 🔑 Anti-Forgery Token — Required for POST in ASP.NET

ASP.NET Core rejects POST requests without an anti-forgery token when `[ValidateAntiForgeryToken]` is used.

**Set it once for ALL jQuery AJAX calls:**

```javascript
// Put this in your _Layout.cshtml scripts section or a shared JS file
// It adds the token header to every single $.ajax / $.post / $.get call

$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});
```

**In your Razor view — add the token:**

```cshtml
@Html.AntiForgeryToken()
@* Generates: <input type="hidden" name="__RequestVerificationToken" value="..."> *@
```

**In your controller:**

```csharp
[HttpPost]
[ValidateAntiForgeryToken]   // ← protected
public JsonResult Create(Employee emp) { ... }
```

---

## 🔑 Global AJAX Events

jQuery has global hooks that fire for every AJAX request in the page — useful for a single loading spinner that covers all requests:

```javascript
// Fires when ANY $.ajax call starts
$(document).on("ajaxStart", function() {
    $("#globalSpinner").show();
});

// Fires when ALL $.ajax calls have finished
$(document).on("ajaxStop", function() {
    $("#globalSpinner").hide();
});

// Fires on any AJAX error
$(document).on("ajaxError", function(e, xhr, settings, error) {
    if (xhr.status === 401) {
        window.location.href = "/Login";   // session expired
    }
    if (xhr.status === 500) {
        showNotification("Server error — please try again", "error");
    }
});
```

---

## 💻 Complete Real Example — Employee CRUD with jQuery AJAX

```javascript
$(function() {

    var grid = $("#employeeGrid").data("kendoGrid");

    // ── Load employees on page open ───────────────────────────
    loadEmployees();

    function loadEmployees() {
        $.get("/Employee/GetAll", function(data) {
            grid.dataSource.data(data);
        }).fail(function() {
            showNotification("Failed to load employees", "error");
        });
    }

    // ── Save (create or update) ───────────────────────────────
    $("#saveBtn").on("click", function() {
        var emp = {
            id:           parseInt($("#EmpId").val()) || 0,
            name:         $("#Name").val().trim(),
            departmentId: parseInt($("#DepartmentId").val()),
            salary:       parseFloat($("#Salary").val())
        };

        var url = emp.id > 0 ? "/Employee/Update" : "/Employee/Create";

        $.post(url, emp)
            .done(function(response) {
                showNotification("Saved successfully", "success");
                loadEmployees();
                closeEditWindow();
            })
            .fail(function(xhr) {
                var errors = xhr.responseJSON;
                displayValidationErrors(errors);
            });
    });

    // ── Delete via event delegation ───────────────────────────
    $(document).on("click", ".deleteBtn", function() {
        var empId = $(this).closest("tr").data("id");

        kendo.confirm("Delete this employee?").then(function() {
            $.post("/Employee/Delete", { id: empId })
                .done(function() {
                    showNotification("Deleted", "success");
                    loadEmployees();
                })
                .fail(function() {
                    showNotification("Delete failed", "error");
                });
        });
    });

    // ── Load cities when country changes ──────────────────────
    $(document).on("change", "#CountryId", function() {
        var countryId = $(this).val();

        $.get("/Location/GetCities", { countryId: countryId }, function(cities) {
            var ddl = $("#CityId").data("kendoDropDownList");
            ddl.setDataSource(cities);
        });
    });

});
```

---

## 📊 Method Comparison Table

| Method                                                                                             | HTTP | Data Location | Auto-parse JSON             | Use For      |
| -------------------------------------------------------------------------------------------------- | ---- | ------------- | --------------------------- | ------------ |
| `$.get(url, data, fn)`                                                                           | GET  | Query string  | ✅ Yes                      | Loading data |
| `$.post(url, data, fn)`                                                                          | POST | Form body     | ✅ Yes                      | Saving data  |
| `$.getJSON(url, data, fn)` | GET  | Query string  | ✅ Yes                       | Same as $.get |      |               |                             |              |
| `$.ajax({ ... })`                                                                                | Any  | Configurable  | ✅ With `dataType:"json"` | Full control |

---

## ❓ Interview Questions

**Q: What is the difference between `$.get` and `$.post`?**

> `$.get` sends a GET request — data goes in the URL query string, no body. Used for loading/reading data. `$.post` sends a POST request — data goes in the request body. Used for creating, updating, or deleting data. In ASP.NET, `$.post` is used with `[HttpPost]` controller actions.

**Q: What is the difference between `contentType` and `dataType` in `$.ajax`?**

> `contentType` describes the format of what you're **sending** to the server — set to `"application/json"` when posting a JSON body so the controller's `[FromBody]` can read it. `dataType` describes the format you **expect back** — set to `"json"` so jQuery auto-parses the response.

**Q: What are `.done()`, `.fail()`, and `.always()`?**

> They are Promise-style callbacks on the jqXHR object returned by AJAX methods. `.done()` fires on success, `.fail()` fires on any HTTP error, `.always()` fires regardless of outcome. They're cleaner than nesting callbacks inside `success` and `error`.

**Q: Why do you need `$.ajaxSetup` with the anti-forgery token?**

> ASP.NET Core's `[ValidateAntiForgeryToken]` requires a CSRF token on every POST request. `$.ajaxSetup` with `beforeSend` attaches that token as a request header automatically — once, globally — so you don't repeat it in every individual AJAX call.

**Q: What is the purpose of `ajaxStart` and `ajaxStop`?**

> Global jQuery events that fire when any AJAX call begins (`ajaxStart`) and when all active calls have finished (`ajaxStop`). Used to show and hide a single global loading spinner that covers the entire application without adding show/hide calls to every individual request.
>
