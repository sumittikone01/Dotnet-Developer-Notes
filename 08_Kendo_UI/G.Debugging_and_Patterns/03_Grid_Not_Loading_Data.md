
# 03 — Grid Not Loading Data

---

## 🎯 One-Line Definition

> **"Grid Not Loading Data" is the most common Kendo problem — this chapter is a complete step-by-step diagnostic guide that walks you from symptom to root cause to fix, covering every possible reason the grid shows empty.**

---

## 🔑 The Problem Has Only 3 Sources

```
Grid shows empty. Every cause fits into one of these 3 buckets:

┌────────────────────────────────────────────────────────────────┐
│  SOURCE 1 — Request never reaches the controller               │
│  (wrong URL, JS error, CORS, script order)                     │
├────────────────────────────────────────────────────────────────┤
│  SOURCE 2 — Controller reached but returns wrong/empty data    │
│  (bad query, null return, wrong JSON shape)                    │
├────────────────────────────────────────────────────────────────┤
│  SOURCE 3 — Controller returns correct data but Grid           │
│  can't read it (schema mismatch, field name case)              │
└────────────────────────────────────────────────────────────────┘

Find which bucket your problem is in → fix it.
DevTools Network tab tells you within 30 seconds.
```

---

## 🔑 The Diagnostic Flowchart

```
GRID SHOWS EMPTY
       │
       ▼
Open F12 → Network → Fetch/XHR → trigger page load
       │
       ├── No request appears in Network tab?
       │         └──► GO TO: Section A — Request Not Firing
       │
       └── Request IS in the list
               │
               ├── Status is RED (4xx / 5xx)?
               │         └──► GO TO: Section B — Request Fails
               │
               └── Status is 200 (green)?
                         │
                         ├── Preview shows Data: [] or Data: null?
                         │         └──► GO TO: Section C — Controller Returns Empty
                         │
                         └── Preview shows data IS there?
                                   └──► GO TO: Section D — Schema Mismatch
```

---

## 🔑 Section A — Request Not Firing

The grid is not making any AJAX call at all.

**Check 1 — Is there a JavaScript error before the grid initialises?**

```
Open Console tab in DevTools.
Look for RED errors.

Most common:
  Uncaught ReferenceError: $ is not defined
  → jQuery not loaded or loaded after Kendo

  Uncaught TypeError: Cannot read properties of null
  → Accessing a DOM element that doesn't exist

Any JS error before the grid init stops the grid from running.
Fix the JS error first.
```

**Check 2 — Is the script load order correct?**

```html
❌ BROKEN — wrong order:
<script src="kendo.all.min.js"></script>
<script src="jquery.min.js"></script>

✅ FIXED:
<script src="jquery.min.js"></script>          ← 1st
<script src="kendo.all.min.js"></script>       ← 2nd
<script src="kendo.aspnetmvc.min.js"></script> ← 3rd
```

**Check 3 — Is `auto-bind` set to false accidentally?**

```cshtml
❌ BROKEN — auto-bind disabled, no manual read:
<datasource type="DataSourceTagHelperType.Ajax"
            auto-bind="false">   ← grid doesn't load on init
    ...
</datasource>
{{!-- No JavaScript to trigger .read() either --}}

✅ FIX A — remove auto-bind (default is true):
<datasource type="DataSourceTagHelperType.Ajax">   ← loads automatically

✅ FIX B — keep auto-bind false but trigger manually:
$(function() {
    $("#employeeGrid").data("kendoGrid").dataSource.read();
});
```

**Check 4 — Is the grid element actually in the DOM?**

```javascript
// Paste in Console tab to verify:
console.log($("#employeeGrid").length);
// 1 = element exists ✅
// 0 = element not found — check your HTML id spelling
```

---

## 🔑 Section B — Request Fires But Fails (4xx / 5xx)

The request reaches the server but gets rejected or crashes.

### Status 404 — URL Not Found

```
SYMPTOM: Network shows 404 for the Read request.
CAUSE:   The URL in transport read doesn't match the controller route.
```

```cshtml
❌ BROKEN — URL typo or wrong controller name:
<read url="/Employe/Read" type="POST" />   ← "Employe" missing 'e'
<read url="/Employee/GetAll" type="POST" /> ← action is "Read" not "GetAll"

✅ FIXED — use @Url.Action to generate the URL (never hardcode):
<read url="@Url.Action("Read", "Employee")" type="POST" />
{{!-- @Url.Action generates the correct URL based on your routes --}}
{{!-- It fails at compile time if action/controller doesn't exist --}}
```

### Status 405 — Method Not Allowed

```
SYMPTOM: Network shows 405 for the Read request.
CAUSE:   Sending GET but controller expects POST, or vice versa.
```

```cshtml
❌ BROKEN — method mismatch:
<read url="@Url.Action("Read", "Employee")" type="GET" />
                                                   ↑ GET

// But controller:
[HttpPost]   ← expects POST
public JsonResult Read([DataSourceRequest] DataSourceRequest request) { }

✅ FIXED — both must be POST:
<read url="@Url.Action("Read", "Employee")" type="POST" />

[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request) { }
```

### Status 400 — Bad Request (Anti-Forgery)

```
SYMPTOM: Network shows 400. Response says antiforgery token missing.
CAUSE:   [ValidateAntiForgeryToken] on controller but no token in request.
```

```javascript
// ✅ Fix — add to layout or shared JS, runs once for all requests:
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});
```

```cshtml
@* Add token to the view: *@
@Html.AntiForgeryToken()
```

### Status 500 — Server Exception

```
SYMPTOM: Network shows 500. Response tab shows HTML error page
         OR JSON with an error message.

CAUSE:   Unhandled exception in the controller action.

HOW TO READ THE ERROR:
  Response tab → look for the exception message in the HTML
  OR
  Visual Studio → Output window → look for the stack trace

Common causes of 500 on Grid Read:
  - NullReferenceException: _db is null (DI not configured)
  - Invalid column name: model property doesn't match DB column
  - Connection string wrong: can't reach the database
  - LINQ can't translate: complex expression EF can't convert to SQL
```

```csharp
// Debug: add try-catch temporarily to see the exact error
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    try
    {
        var result = _db.Employees.ToDataSourceResult(request);
        return Json(result);
    }
    catch (Exception ex)
    {
        // Return the error as JSON so you can read it in Network → Response
        return Json(new { error = ex.Message, stack = ex.StackTrace });
    }
}
// Remove this try-catch after debugging — it's only for diagnosis
```

---

## 🔑 Section C — Request Succeeds But Returns Empty Data

Status 200 but Preview shows `"Data": []` or `"Total": 0`.

**Check 1 — Is there an accidental filter in the query?**

```csharp
// Check your Read action carefully:
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    var result = _db.Employees
                    .Where(e => e.ManagerId == currentUserId)   // ← is this filter correct?
                    .ToDataSourceResult(request);

    // If currentUserId is wrong → returns 0 results
    return Json(result);
}
```

**Check 2 — Is the database actually populated?**

```csharp
// Add this temporarily to check:
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    var count = _db.Employees.Count();
    Console.WriteLine($"Total employees in DB: {count}");
    // Check Visual Studio Output window for this value

    return Json(_db.Employees.ToDataSourceResult(request));
}
```

**Check 3 — Is a grid-level filter hiding all results?**

```javascript
// Check if a filter was set programmatically:
var grid = $("#employeeGrid").data("kendoGrid");
console.log("Current filter:", grid.dataSource.filter());
// If this shows filters, clear them:
grid.dataSource.filter({});
```

**Check 4 — Is EF returning no results due to wrong Include / projection?**

```csharp
❌ BROKEN — Include with broken navigation:
_db.Employees
   .Include(e => e.Department)
   .Where(e => e.Department.Name == "IT")  // If Department is null → no results

✅ FIXED — null-safe:
_db.Employees
   .Include(e => e.Department)
   .Where(e => e.Department != null && e.Department.Name == "IT")
```

---

## 🔑 Section D — Data Returns But Grid Still Shows Empty

Status 200, Preview shows records — but grid is blank. This is always a configuration mismatch.

**Check 1 — `schema.data` missing or wrong**

```javascript
// Response: { "Data": [...records...], "Total": 47 }
// But schema says:

❌ BROKEN:
schema: {
    model: { id: "Id" }
    // schema.data not set → Kendo looks for array at root level
    // Root is an object {}, not an array → 0 records shown
}

✅ FIXED:
schema: {
    data:  "Data",    // ← "the array is inside response.Data"
    total: "Total",
    model: { id: "Id" }
}
```

**Check 2 — Field name case mismatch**

```javascript
// JSON from server (camelCase — common with JsonNamingPolicy.CamelCase):
{ "id": 1, "name": "Alice", "department": "IT" }

❌ BROKEN — schema uses PascalCase:
fields: {
    Id:         { type: "number" },   // ← "Id" ≠ "id"
    Name:       { type: "string" },   // ← "Name" ≠ "name"
    Department: { type: "string" }    // ← "Department" ≠ "department"
}
// Kendo can't map fields → all values undefined → empty grid

✅ FIXED — match JSON exactly:
fields: {
    id:         { type: "number", editable: false },
    name:       { type: "string" },
    department: { type: "string" }
}
```

```cshtml
{{!-- Also fix column field names to match: --}}
❌ <column field="Name" />        {{!-- "Name" ≠ "name" in JSON --}}
✅ <column field="name" />        {{!-- matches camelCase JSON --}}
```

**Check 3 — Program.cs not configured for camelCase**

```csharp
// If controller returns PascalCase but your schema/columns use camelCase:

// In Program.cs, ensure consistent casing:
builder.Services.AddControllersWithViews()
    .AddJsonOptions(options => {
        options.JsonSerializerOptions.PropertyNamingPolicy =
            JsonNamingPolicy.CamelCase;   // ← all JSON becomes camelCase
    });

// Then schema and column fields must use camelCase:
// field="name", field="department" (not "Name", "Department")
```

**Check 4 — DataSource type wrong**

```cshtml
❌ BROKEN — wrong DataSource type:
<datasource type="DataSourceTagHelperType.Custom">
    {{!-- Custom type doesn't auto-handle Kendo's server operations --}}

✅ FIXED — use Ajax for remote data:
<datasource type="DataSourceTagHelperType.Ajax">
```

---

## 🔑 Checklist — Run Through Every Time Grid Is Empty

```
□  1. Open DevTools → Network → Fetch/XHR
□  2. Check: is there a request in the list?
       NO → check script order, JS console errors, auto-bind
□  3. Check: what is the status code?
       404 → fix the URL (use @Url.Action)
       405 → fix HTTP method mismatch
       400 → add anti-forgery token
       500 → read error in Response tab
□  4. Check: what does Preview tab show?
       Data: null or [] → fix the controller query
       Data has records → go to step 5
□  5. Check schema.data
       Is it set to "Data"? (or whatever your wrapper property is)
□  6. Check field name case
       JSON: "name" → schema/columns: field="name" (must match exactly)
□  7. Check Program.cs camelCase vs PascalCase consistency
□  8. Check for accidental filters
       grid.dataSource.filter() in console — clear with filter({})
```

---

## ❓ Interview Questions

**Q: Grid shows empty and status is 200 — what do you check first?**

> Open DevTools → Network → Preview tab for the Read request. If `Data` is populated, the issue is a schema mismatch — check `schema.data` and field name casing. If `Data` is empty, the controller query is returning nothing — check for unintentional filters or database issues.

**Q: What causes a 404 on a Kendo Grid Read request?**

> The URL in `transport.read` doesn't match the controller route. Most common causes: typo in the URL string, wrong action name, or wrong controller name. Fix: always use `@Url.Action("ActionName", "ControllerName")` which is validated at compile time.

**Q: Why does the grid ignore `serverPaging` and load all records?**

> `.ToList()` was called before `.ToDataSourceResult(request)`. Once data is in a `List<T>`, all paging happens in memory. Pass `AsQueryable()` to `ToDataSourceResult` so it builds the SKIP/TAKE into the SQL query.

**Q: Field names in schema are correct but grid still shows empty — what else could it be?**

> `schema.data` might be missing. If the server returns `{ "Data": [...], "Total": 50 }` and schema doesn't specify `data: "Data"`, Kendo looks for an array at the root level, finds an object, and shows zero records.
>
