
# 02 — DataSource Transport

---

## 🎯 One-Line Definition

> **Transport is the section of DataSource that tells it WHERE to send each operation — which URL, which HTTP method, and what extra data to include.**

---

## 🚚 What "Transport" Means

Think of Transport as a  **delivery routing map** :

```
  OPERATION          TRANSPORT SENDS IT TO
  ─────────          ─────────────────────
  Read (load data) ──────► POST /Employee/Read
  Create (new row) ──────► POST /Employee/Create
  Update (edit row)──────► POST /Employee/Update
  Destroy (delete) ──────► POST /Employee/Destroy
```

Without transport, DataSource has no idea where to send requests.

---

## 🔑 Transport — All Four Operations

```javascript
transport: {

    read: {
        url:  "/Employee/Read",    // URL of your Read action
        type: "POST"               // HTTP method
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

}
```

> **Why POST for everything?** Kendo sends complex JSON bodies (paging info, the model, etc.). GET requests can't carry a body — so POST is used for all operations including Read in Kendo.

---

## 🔑 Transport in Tag Helper (What You Use Daily)

```cshtml
<datasource type="DataSourceTagHelperType.Ajax">
    <transport>
        <read    url="@Url.Action("Read",    "Employee")" type="POST" />
        <create  url="@Url.Action("Create",  "Employee")" type="POST" />
        <update  url="@Url.Action("Update",  "Employee")" type="POST" />
        <destroy url="@Url.Action("Destroy", "Employee")" type="POST" />
    </transport>
</datasource>
```

`@Url.Action("Read", "Employee")` generates the correct URL for you — `/Employee/Read`. Always use this instead of hard-coding URLs.

---

## 🔑 What Gets Sent to the Server

### What Kendo sends on Read

When the Grid requests data, Kendo automatically includes paging, sorting, and filter info in the body:

```
POST /Employee/Read

Body (form-encoded, handled by [DataSourceRequest]):
  page=1
  pageSize=10
  sort[0][field]=Name
  sort[0][dir]=asc
  filter[logic]=and
  filter[filters][0][field]=Department
  filter[filters][0][operator]=eq
  filter[filters][0][value]=IT
```

You don't build this manually. `[DataSourceRequest]` on the controller side reads it automatically.

### What Kendo sends on Create/Update

The model object is sent as form fields:

```
POST /Employee/Create

Body:
  Name=Alice
  Department=IT
  Salary=75000
  HireDate=2021-06-15
  IsActive=true
```

Your controller parameter `Employee employee` gets auto-populated from these fields.

---

## 🔑 Sending Extra Parameters with Transport

Sometimes you need to send extra data along with the request — like a parent ID to filter by, or a flag.

### Option 1 — Static extra data (same every time)

```javascript
transport: {
    read: {
        url:  "/Employee/Read",
        type: "POST",
        data: {
            departmentId: 3,    // always send this
            activeOnly:   true
        }
    }
}
```

### Option 2 — Dynamic extra data (changes at runtime)

```javascript
transport: {
    read: {
        url:  "/Employee/Read",
        type: "POST",
        data: function() {
            // This function runs every time a read happens
            // Return whatever the current values are
            return {
                departmentId: $("#DeptFilter").val(),
                searchTerm:   $("#SearchBox").val()
            };
        }
    }
}
```

### In the Controller — receive extra parameters

```csharp
[HttpPost]
public JsonResult Read(
    [DataSourceRequest] DataSourceRequest request,
    int?   departmentId,   // ← receives the extra parameter
    bool   activeOnly,
    string searchTerm)
{
    var query = _db.Employees.AsQueryable();

    if (departmentId.HasValue)
        query = query.Where(e => e.DepartmentId == departmentId);

    if (activeOnly)
        query = query.Where(e => e.IsActive);

    if (!string.IsNullOrEmpty(searchTerm))
        query = query.Where(e => e.Name.Contains(searchTerm));

    return Json(query.ToDataSourceResult(request));
}
```

---

## 🔑 Refreshing Data After Filter Changes

When user changes a filter dropdown and you want to reload the grid:

```javascript
function applyDepartmentFilter() {
    var grid = $("#employeeGrid").data("kendoGrid");

    // Option 1 — update the data function and re-read
    grid.dataSource.read();

    // Option 2 — directly pass new params to read
    grid.dataSource.read({ departmentId: $("#DeptFilter").val() });
}
```

---

## 🔑 Transport with Anti-Forgery Token

ASP.NET Core requires an anti-forgery token for POST requests with `[ValidateAntiForgeryToken]`.

```javascript
// Set this ONCE — applies to ALL jQuery AJAX calls including Kendo
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});
```

In your view, include the token:

```cshtml
@Html.AntiForgeryToken()
```

In your controller:

```csharp
[HttpPost]
[ValidateAntiForgeryToken]   // ← now protected
public JsonResult Create([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    ...
}
```

---

## 🔑 Read-Only Transport (Just Read, No CRUD)

For Charts, DropDownLists, and read-only Grids — you only need `read`:

```javascript
// JavaScript DataSource with only read
var departmentSource = new kendo.data.DataSource({
    transport: {
        read: {
            url:  "/Department/GetAll",
            type: "GET"             // GET is fine when there's no body
        }
    }
});

// Tag Helper version (DropDownList)
<kendo-dropdownlist name="DeptDdl"
                    data-text-field="name"
                    data-value-field="id">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetAll", "Department")" />
        </transport>
    </datasource>
</kendo-dropdownlist>
```

---

## 🔑 Custom Transport — For Non-Standard APIs

If your API doesn't fit the standard Kendo format, you can write a fully custom transport:

```javascript
transport: {
    read: function(options) {
        // options.success(data) — call this when data is ready
        // options.error(error)  — call this on failure

        $.ajax({
            url:     "/api/employees",
            type:    "GET",
            headers: { "Authorization": "Bearer " + getToken() },
            success: function(response) {
                options.success(response.items);  // pass data to Kendo
            },
            error: function(xhr) {
                options.error(xhr);
            }
        });
    },

    create: function(options) {
        $.ajax({
            url:         "/api/employees",
            type:        "POST",
            contentType: "application/json",
            data:        JSON.stringify(options.data),  // the new record
            success: function(response) {
                options.success(response);
            }
        });
    }
}
```

Use custom transport when: the API uses a different URL pattern, requires special headers (like a Bearer token), or returns JSON in a format that doesn't match Kendo's schema.

---

## 📊 Transport Quick Reference

| Property        | What it does            | Example value                  |
| --------------- | ----------------------- | ------------------------------ |
| `read.url`    | URL to fetch data from  | `"/Employee/Read"`           |
| `read.type`   | HTTP method for read    | `"POST"`                     |
| `read.data`   | Extra params to send    | `{ deptId: 5 }`or a function |
| `create.url`  | URL to save new records | `"/Employee/Create"`         |
| `update.url`  | URL to save edits       | `"/Employee/Update"`         |
| `destroy.url` | URL to delete records   | `"/Employee/Destroy"`        |

---

## ⚠️ Common Transport Mistakes

| Mistake                                     | Symptom                                  | Fix                                       |
| ------------------------------------------- | ---------------------------------------- | ----------------------------------------- |
| Using GET for Read with large sort/filter   | 414 URL Too Long error                   | Use `type: "POST"`for Read              |
| Hard-coded URL strings                      | Breaks when app is hosted in subfolder   | Use `@Url.Action()`                     |
| Forgetting anti-forgery token               | 400 Bad Request on Create/Update/Destroy | Add `$.ajaxSetup`with token header      |
| Extra params as static object, not function | Filter sends old value, not current      | Use `data: function() { return {...} }` |

---

## ❓ Interview Questions

**Q: What is the Transport in a Kendo DataSource?**

> The section that configures which URLs and HTTP methods to use for each data operation — Read, Create, Update, and Destroy.

**Q: Why does Kendo use POST for the Read operation instead of GET?**

> The Read request includes paging, sorting, and filtering information which can be large and complex. POST allows sending this in the request body — GET has URL length limits and no body.

**Q: How do you send extra parameters like a filter value to a Read endpoint?**

> Use `data: function() { return { paramName: currentValue } }` inside the read transport config. A function is used (not a static object) so it evaluates the current value at request time, not at init time.

**Q: What is a custom transport and when do you use it?**

> A custom transport replaces the URL config with your own JavaScript functions that make the AJAX calls manually. Used when the API has non-standard URL patterns, requires special headers like auth tokens, or returns a JSON format that doesn't match Kendo's expectations.
>
