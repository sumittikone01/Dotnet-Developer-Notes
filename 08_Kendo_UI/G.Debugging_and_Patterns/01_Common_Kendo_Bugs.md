
# 01 — Common Kendo Bugs

---

## 🎯 One-Line Definition

> **This chapter is your Kendo debugging handbook — every bug you will hit is listed here with its exact symptom, the root cause, and the precise fix, so you spend minutes solving problems instead of hours.**

---

## 🔑 How to Read This Chapter

```
Each bug follows this structure:

  SYMPTOM   → what you see (or don't see) on the page
  CAUSE     → why it happens
  FIX       → exactly what to change
  EXAMPLE   → before (broken) and after (fixed) code
```

---

## 🐛 Bug Category 1 — Setup and Script Bugs

---

### Bug 1.1 — Nothing Renders, Console Shows `$ is not defined`

```
SYMPTOM:  Blank page. Browser console shows:
          Uncaught ReferenceError: $ is not defined

CAUSE:    Kendo JS loaded BEFORE jQuery.
          Kendo depends on jQuery — loading order matters.

FIX:      jQuery MUST come first.
```

```html
❌ BROKEN — Kendo before jQuery:
<script src="kendo.all.min.js"></script>
<script src="jquery.min.js"></script>

✅ FIXED — jQuery first:
<script src="jquery.min.js"></script>
<script src="kendo.all.min.js"></script>
<script src="kendo.aspnetmvc.min.js"></script>
```

---

### Bug 1.2 — Tag Helpers Not Recognised (Razor Shows `<kendo-grid>` as Plain Text)

```
SYMPTOM:  <kendo-grid> appears as literal text in the browser.
          No grid renders at all.

CAUSE:    The Kendo Tag Helper line is missing from _ViewImports.cshtml.

FIX:      Add the addTagHelper line.
```

```cshtml
❌ BROKEN — missing in _ViewImports.cshtml:
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
// Kendo line missing

✅ FIXED:
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, Telerik.UI.for.AspNet.Core   ← add this
```

---

### Bug 1.3 — `ToDataSourceResult()` Not Found (Compile Error)

```
SYMPTOM:  Build fails:
          'IQueryable<Employee>' does not contain a definition
          for 'ToDataSourceResult'

CAUSE:    Missing using statement.

FIX:      Add both using statements to your controller.
```

```csharp
❌ BROKEN — missing usings:
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    return Json(_db.Employees.ToDataSourceResult(request));
    // ↑ compile error
}

✅ FIXED:
using Kendo.Mvc.Extensions;   // ← for .ToDataSourceResult()
using Kendo.Mvc.UI;           // ← for [DataSourceRequest]

public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    return Json(_db.Employees.ToDataSourceResult(request));  // ✅
}
```

---

### Bug 1.4 — Widgets Unstyled (Raw HTML, No Kendo Look)

```
SYMPTOM:  Grid or DatePicker renders as a plain HTML table / input.
          No Kendo styling applied.

CAUSE:    Theme CSS file is missing or loading after the page renders.

FIX:      Add the theme CSS link in <head> — never in <body>.
```

```html
❌ BROKEN — CSS missing or in body:
<body>
    <link rel="stylesheet" href="kendo.default.css" />  ← wrong place

✅ FIXED — CSS in head:
<head>
    <link rel="stylesheet"
          href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css" />
</head>
```

---

## 🐛 Bug Category 2 — Grid Data Bugs

---

### Bug 2.1 — Grid Shows "No Records to Display" Even Though Controller Returns Data

```
SYMPTOM:  Grid renders but shows empty. Controller is definitely
          returning records. Network tab shows 200 OK with JSON.

CAUSE A:  schema.data is missing — DataSource can't find the array.
CAUSE B:  Field name case mismatch between JSON and schema model.
CAUSE C:  serverPaging: true but Total is 0 or missing.
```

```javascript
❌ BROKEN — no schema.data:
schema: {
    model: { id: "Id" }
    // "Data" array not mapped
}
// Server returns: { "Data": [...], "Total": 50 }
// Kendo can't find the array — shows empty

✅ FIXED:
schema: {
    data:  "Data",     // ← "the records are in response.Data"
    total: "Total",    // ← "the total count is in response.Total"
    model: { id: "Id" }
}
```

```javascript
❌ BROKEN — field name case mismatch:
// JSON from server: { "employeeId": 1, "fullName": "Alice" }
// Schema model says:
fields: {
    EmployeeId: { type: "number" },   // ← capital E — WRONG
    FullName:   { type: "string" }    // ← capital F — WRONG
}

✅ FIXED — names must match JSON exactly:
fields: {
    employeeId: { type: "number" },   // ← matches "employeeId"
    fullName:   { type: "string" }    // ← matches "fullName"
}
```

---

### Bug 2.2 — Pager Shows "1 of 1" But There Are 500 Records

```
SYMPTOM:  Grid loads 10 rows but pager says "Page 1 of 1".
          Cannot navigate to next page.

CAUSE:    schema.total is missing. Kendo doesn't know the total count.
          serverPaging is true but Total field isn't mapped.

FIX:      Add schema.total pointing to the Total field.
```

```javascript
❌ BROKEN:
schema: {
    data: "Data"
    // total missing — Kendo thinks total = current page count = 10
}

✅ FIXED:
schema: {
    data:  "Data",
    total: "Total"   // ← Kendo now knows: "500 total records, show pager"
}
```

---

### Bug 2.3 — Grid Loads All Records Instead of One Page (serverPaging Ignored)

```
SYMPTOM:  All 5,000 rows load at once. Page is very slow.
          Server seems to ignore page/pageSize parameters.

CAUSE:    .ToList() called BEFORE .ToDataSourceResult().
          All data loaded into memory first — paging becomes client-side.

FIX:      Pass IQueryable, not List<T>, to ToDataSourceResult.
```

```csharp
❌ BROKEN — ToList() first:
var employees = _db.Employees.ToList();          // ← ALL rows in memory
return Json(employees.ToDataSourceResult(request));  // paging in memory

✅ FIXED — IQueryable:
var employees = _db.Employees.AsQueryable();     // ← no SQL yet
return Json(employees.ToDataSourceResult(request));  // paging in SQL
```

---

### Bug 2.4 — Sorting or Filtering Breaks, Causes 500 Error

```
SYMPTOM:  Grid loads fine. User clicks a column header to sort.
          Console shows 500 error.

CAUSE A:  Sorting by a navigation property directly
          (e.g. employee.Department.Name — EF can't translate)

CAUSE B:  IEnumerable passed instead of IQueryable.
```

```csharp
❌ BROKEN — navigation property sort fails:
var employees = _db.Employees.Include(e => e.Department);
return Json(employees.ToDataSourceResult(request));
// User sorts by "Department.Name" → EF throws

✅ FIXED — project to flat DTO first:
var employees = _db.Employees.Select(e => new EmployeeDto {
    Id             = e.Id,
    Name           = e.Name,
    DepartmentName = e.Department.Name,   // ← flattened
    Salary         = e.Salary
});
return Json(employees.ToDataSourceResult(request));
```

---

## 🐛 Bug Category 3 — Edit and Save Bugs

---

### Bug 3.1 — Edit Popup Opens but All Fields Are Empty

```
SYMPTOM:  User clicks Edit. Popup opens. All inputs are blank.
          No data pre-filled.

CAUSE A:  schema.model id not set — Kendo can't identify the row.
CAUSE B:  Field names in schema don't match JSON property names.
CAUSE C:  editable: false on fields that should be editable.
```

```javascript
❌ BROKEN — no id in schema model:
schema: {
    data: "Data", total: "Total",
    model: {
        // id missing — Kendo can't find which row was clicked
        fields: { name: { type: "string" } }
    }
}

✅ FIXED:
schema: {
    data: "Data", total: "Total",
    model: {
        id: "Id",    // ← tells Kendo which field is the primary key
        fields: {
            Id:   { type: "number", editable: false },
            Name: { type: "string" }
        }
    }
}
```

---

### Bug 3.2 — Save/Delete Returns 400 Bad Request (Anti-Forgery Token)

```
SYMPTOM:  Grid read works fine (GET). Create/Update/Destroy
          return 400 Bad Request. Server logs show:
          "The required antiforgery request token was not supplied."

CAUSE:    [ValidateAntiForgeryToken] on controller but token
          not sent in AJAX headers.

FIX:      Add $.ajaxSetup to send the token with every request.
```

```cshtml
@* Add token to the view *@
@Html.AntiForgeryToken()
```

```javascript
// ✅ Add once in layout or shared JS file:
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});
```

---

### Bug 3.3 — Create Succeeds but Grid Shows New Row with ID = 0

```
SYMPTOM:  Save works (database gets the record). But the new row
          in the grid still shows Id = 0 instead of the real DB Id.

CAUSE:    Controller not returning the saved model with its new Id.

FIX:      Return the saved employee (with Id populated by DB).
```

```csharp
❌ BROKEN — returning original model before save:
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                         Employee emp)
{
    _db.Employees.Add(emp);
    _db.SaveChanges();
    return Json(new { success = true });   // ← Kendo gets no model back
    // Grid row stays with Id = 0
}

✅ FIXED — return saved model:
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                         Employee emp)
{
    if (ModelState.IsValid)
    {
        _db.Employees.Add(emp);
        _db.SaveChanges();
        // emp.Id is now populated by EF after SaveChanges
    }
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
    //          ↑ returns saved emp with real Id
}
```

---

### Bug 3.4 — Validation Errors from Server Not Showing in Popup

```
SYMPTOM:  Server returns validation errors (ModelState has errors).
          But the popup just closes — no error messages shown.

CAUSE:    ModelState not passed to ToDataSourceResult.

FIX:      Always pass ModelState as second argument.
```

```csharp
❌ BROKEN — ModelState not passed:
return Json(new[] { emp }.ToDataSourceResult(request));
// Errors field = null → popup closes, errors lost

✅ FIXED:
return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
//                                                    ↑ errors flow to Grid
```

---

## 🐛 Bug Category 4 — Widget Bugs

---

### Bug 4.1 — DropDownList Shows `[object Object]`

```
SYMPTOM:  DropDownList renders but every option shows "[object Object]"
          instead of the name.

CAUSE:    data-text-field is missing or doesn't match the JSON property name.

FIX:      Set data-text-field to the exact property name in the JSON.
```

```cshtml
❌ BROKEN:
<kendo-dropdownlist name="DepartmentId"
                    data-value-field="id" />
{{!-- data-text-field missing — Kendo calls .toString() on the object --}}

✅ FIXED:
<kendo-dropdownlist name="DepartmentId"
                    data-text-field="name"    ← must match JSON: {"id":1,"name":"IT"}
                    data-value-field="id" />
```

---

### Bug 4.2 — Kendo Widget Not Found: `$().data("kendoGrid")` Returns `undefined`

```
SYMPTOM:  JavaScript throws: "Cannot read properties of undefined"
          when calling grid.dataSource.read() or any method.

CAUSE A:  Code runs before the DOM is ready / before grid is initialized.
CAUSE B:  Wrong element ID in the selector.
CAUSE C:  Grid was destroyed and not re-initialized.
```

```javascript
❌ BROKEN — code runs before DOM ready:
var grid = $("#employeeGrid").data("kendoGrid");  // ← too early, undefined
grid.dataSource.read();                          // ← crashes

✅ FIXED — wrap in document ready:
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");
    grid.dataSource.read();  // ✅ grid exists now
});

// Debug tip: check if the element exists first:
console.log($("#employeeGrid").length);  // should be 1, not 0
```

---

### Bug 4.3 — DatePicker Sends Wrong Date (Off by One Day)

```
SYMPTOM:  User picks June 15. Server receives June 14.
          Typically a timezone offset issue.

CAUSE:    JavaScript Date objects use local time. When converted to ISO,
          UTC offset can push the date back by one day.
          e.g. "2021-06-15T00:00:00.000Z" in UTC-5 = June 14 23:00 local.

FIX:      Format the date as a string before sending to avoid TZ issues.
```

```javascript
❌ BROKEN — sending raw Date:
var date = $("#HireDate").data("kendoDatePicker").value();
$.post("/Employee/Save", { hireDate: date });  // timezone shifts date

✅ FIXED — send as formatted string:
var date = $("#HireDate").data("kendoDatePicker").value();
$.post("/Employee/Save", {
    hireDate: date ? kendo.toString(date, "yyyy-MM-dd") : null
    // "2021-06-15" — no timezone, server parses correctly
});
```

---

### Bug 4.4 — NumericTextBox `.val()` Returns Formatted String, Not a Number

```
SYMPTOM:  JavaScript receives "$75,000" instead of 75000.
          Server receives 0 or NaN.

CAUSE:    Reading the raw input value instead of using the Kendo API.

FIX:      Always use .data("kendoNumericTextBox").value().
```

```javascript
❌ BROKEN:
var salary = $("#Salary").val();    // "$75,000.00" — useless
$.post("/Save", { salary: salary }); // server gets "$75,000" = 0

✅ FIXED:
var salary = $("#Salary").data("kendoNumericTextBox").value();  // 75000
$.post("/Save", { salary: salary });  // ✅ server gets 75000
```

---

## 🐛 Bug Category 5 — DataSource and AJAX Bugs

---

### Bug 5.1 — Grid Caches Old Data After Edit (Secondary Grid Bug)

```
SYMPTOM:  You have a master-detail setup. Select row A → detail grid
          shows A's data. Select row B → detail grid STILL shows A's data.

CAUSE:    DataSource caches the data locally. Reading again with new
          parameters sometimes returns cached results instead of fresh data.

FIX:      Destroy and recreate the detail grid on each selection.
          OR call dataSource.data([]) to clear cache before read.
```

```javascript
❌ BROKEN — only changing filter, data stays cached:
function loadDetailGrid(empId) {
    var ds = $("#detailGrid").data("kendoGrid").dataSource;
    ds.read({ employeeId: empId });  // sometimes returns cached data
}

✅ FIXED — destroy and recreate:
function loadDetailGrid(empId) {
    var existing = $("#detailGrid").data("kendoGrid");
    if (existing) {
        existing.destroy();
        $("#detailGrid").empty();
    }
    // Rebuild with new employeeId
    $("#detailGrid").kendoGrid({
        dataSource: {
            transport: {
                read: {
                    url:  "/Project/GetByEmployee",
                    type: "POST",
                    data: { employeeId: empId }
                }
            },
            schema: { data: "Data", total: "Total" },
            pageSize: 5, serverPaging: true
        },
        columns: [ ... ],
        pageable: true
    });
}
```

---

### Bug 5.2 — AJAX Sends Data But Controller Receives Null Model

```
SYMPTOM:  $.ajax POST fires. Controller action is hit.
          But [FromBody] Employee emp is null.

CAUSE:    Missing contentType: "application/json"
          OR missing JSON.stringify()
          OR both.
```

```javascript
❌ BROKEN — missing contentType and stringify:
$.ajax({
    url:  "/Employee/Create",
    type: "POST",
    data: { name: "Alice", salary: 75000 }
    // No contentType → form-encoded body
    // Controller has [FromBody] → null
});

✅ FIXED:
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",         // ← tell server it's JSON
    data:        JSON.stringify({            // ← convert to JSON string
                     name: "Alice",
                     salary: 75000
                 }),
    success: function(r) { ... }
});
```

---

## 📊 Bug Quick Reference — Most Common Causes

| Symptom                         | Most Likely Cause                          | Quick Check                                  |
| ------------------------------- | ------------------------------------------ | -------------------------------------------- |
| Nothing renders                 | jQuery/Kendo script order                  | Check console for `$ is not defined`       |
| Tag helpers ignored             | Missing `@addTagHelper`                  | Check `_ViewImports.cshtml`                |
| Grid empty (200 OK)             | Missing `schema.data`                    | Check Network → Response tab                |
| Pager stuck at "1 of 1"         | Missing `schema.total`                   | Add `total: "Total"`to schema              |
| 400 on save/delete              | Anti-forgery token missing                 | Add `$.ajaxSetup`with token header         |
| Edit popup empty                | Missing `id`in schema model              | Add `id: "Id"`to schema model              |
| `[object Object]`in DDL       | Missing `data-text-field`                | Add `data-text-field="name"`               |
| `.data("kendoGrid")`undefined | Code runs before DOM ready                 | Wrap in `$(function() { })`                |
| All 5000 rows load at once      | `.ToList()`before `ToDataSourceResult` | Pass `IQueryable`not `List<T>`           |
| NumericTextBox sends string     | Using `.val()`not `.value()`           | Use `.data("kendoNumericTextBox").value()` |

---

## ❓ Interview Questions

**Q: Why does a Kendo Grid show "No Records to Display" even though the controller returns data?**

> Most commonly because `schema.data` is not configured. The server returns `{ "Data": [...], "Total": 50 }` but Kendo doesn't know the array is inside `Data`. Fix: add `schema: { data: "Data", total: "Total" }` to the DataSource.

**Q: Why does create/update/destroy return 400 Bad Request?**

> Usually the anti-forgery token is missing from the AJAX request headers. ASP.NET's `[ValidateAntiForgeryToken]` rejects requests without the token. Fix: add `$.ajaxSetup` with `beforeSend` that sets the `RequestVerificationToken` header.

**Q: Why does `$().data("kendoGrid")` return `undefined`?**

> The code is running before the document is ready (before the grid is initialized), the element ID is wrong, or the grid was destroyed and not recreated. Fix: wrap all grid code in `$(function() { })` and verify the ID matches exactly.

**Q: Why does a secondary (detail) grid cache old data?**

> The DataSource caches loaded data locally. Even with new parameters, it can return cached results. The reliable fix is to destroy the existing grid widget and recreate it fresh with the new configuration — `grid.destroy()` then `$("#id").empty()` then `kendoGrid({...})`.
>
