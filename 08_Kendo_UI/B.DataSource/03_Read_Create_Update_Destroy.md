
# 03 — Read, Create, Update, Destroy

---

## 🎯 One-Line Definition

> **These are the four CRUD operations that the DataSource performs — each one maps to one controller action that returns JSON.**

---

## 🗺️ The Full Picture

```
  KENDO GRID ACTION          DATASOURCE CALLS        CONTROLLER ACTION
  ──────────────────         ─────────────────       ─────────────────
  Grid loads / page turn  →  Read    transport   →   Read()    → Json(data)
  User adds new row       →  Create  transport   →   Create()  → Json(saved)
  User edits a row        →  Update  transport   →   Update()  → Json(updated)
  User deletes a row      →  Destroy transport   →   Destroy() → Json(deleted)
```

Every operation follows the same pattern:

1. Kendo sends a POST request with the data
2. Your controller processes it
3. Controller returns JSON with the result + any errors
4. DataSource updates the widget

---

## 🔑 READ — Loading Data

### What happens

```
Grid needs data
    │
    ▼
DataSource sends POST to /Employee/Read
with paging, sorting, filter info in the body
    │
    ▼
Controller queries DB using those params
    │
    ▼
Returns: { "Data": [...rows...], "Total": 150 }
    │
    ▼
DataSource gives rows to Grid
Grid renders the table
```

### Controller

```csharp
using Kendo.Mvc.Extensions;   // for .ToDataSourceResult()
using Kendo.Mvc.UI;           // for DataSourceRequest

[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
{
    // [DataSourceRequest] reads page, sort, filter from the POST body
    // .ToDataSourceResult() applies those to your IQueryable automatically

    var result = _db.Employees
                    .AsQueryable()
                    .ToDataSourceResult(request);

    return Json(result);

    // Returns: { "Data": [...], "Total": 150, "Errors": null }
}
```

### JSON sent by Grid to server

```json
{
  "page": 1,
  "pageSize": 10,
  "sort": [{ "field": "Name", "dir": "asc" }],
  "filter": {
    "logic": "and",
    "filters": [{ "field": "Department", "operator": "eq", "value": "IT" }]
  }
}
```

### JSON server sends back

```json
{
  "Data": [
    { "id": 2, "name": "Bob",   "department": "IT", "salary": 70000 },
    { "id": 5, "name": "Dave",  "department": "IT", "salary": 65000 }
  ],
  "Total": 12,
  "Errors": null
}
```

---

## 🔑 CREATE — Adding a New Record

### What happens

```
User clicks "Add New" → fills popup form → clicks Save
    │
    ▼
DataSource sends POST to /Employee/Create
with the new record's field values in the body
    │
    ▼
Controller validates, adds to DB, gets the new ID
    │
    ▼
Returns the saved object (with the new ID assigned)
    │
    ▼
DataSource updates the temporary row with the real ID
Grid reflects the saved record
```

### Controller

```csharp
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                         Employee employee)
{
    if (ModelState.IsValid)
    {
        _db.Employees.Add(employee);
        _db.SaveChanges();
        // employee.Id is now populated by the database
    }

    // Pass ModelState to return validation errors to the Grid
    return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
}
```

### JSON server sends back (success)

```json
{
  "Data": [{ "id": 26, "name": "Alice", "department": "IT", "salary": 75000 }],
  "Total": 1,
  "Errors": null
}
```

### JSON server sends back (validation failed)

```json
{
  "Data": [{ "id": 0, "name": "", "department": "IT", "salary": -100 }],
  "Total": 1,
  "Errors": {
    "Name":   { "errors": ["Name is required"] },
    "Salary": { "errors": ["Salary must be between 10000 and 500000"] }
  }
}
```

> The Grid reads the `Errors` field and automatically shows the error messages next to the relevant fields in the edit popup.

---

## 🔑 UPDATE — Editing a Record

### What happens

```
User clicks Edit on a row → changes values → clicks Update
    │
    ▼
DataSource sends POST to /Employee/Update
with the ENTIRE updated record (all fields, not just changed ones)
    │
    ▼
Controller finds the existing record, applies changes, saves
    │
    ▼
Returns the updated record
    │
    ▼
DataSource refreshes that row in the Grid
```

### Controller

```csharp
[HttpPost]
public JsonResult Update([DataSourceRequest] DataSourceRequest request,
                         Employee employee)
{
    if (ModelState.IsValid)
    {
        _db.Employees.Update(employee);
        _db.SaveChanges();
    }

    return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
}
```

> **Important:** Kendo sends the full model on update — all fields, not just the changed ones. Your model must have `id` configured in schema so Kendo knows which record to update.

---

## 🔑 DESTROY — Deleting a Record

### What happens

```
User clicks Delete → confirmation (optional) → confirmed
    │
    ▼
DataSource sends POST to /Employee/Destroy
with the record's ID (and other fields) in the body
    │
    ▼
Controller finds by ID, removes from DB
    │
    ▼
Returns the deleted record (Kendo convention — return what was sent)
    │
    ▼
DataSource removes the row from the Grid
```

### Controller

```csharp
[HttpPost]
public JsonResult Destroy([DataSourceRequest] DataSourceRequest request,
                          Employee employee)
{
    var existing = _db.Employees.Find(employee.Id);

    if (existing != null)
    {
        _db.Employees.Remove(existing);
        _db.SaveChanges();
    }

    return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
}
```

---

## 🔑 All Four Controllers Together — The Standard Pattern

```csharp
public class EmployeeController : Controller
{
    private readonly AppDbContext _db;
    public EmployeeController(AppDbContext db) => _db = db;

    // Page render — no data, just returns the View
    public IActionResult Index() => View();

    // ── CRUD Actions ─────────────────────────────────────────

    [HttpPost]
    public JsonResult Read([DataSourceRequest] DataSourceRequest request)
        => Json(_db.Employees.ToDataSourceResult(request));

    [HttpPost]
    public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                             Employee emp)
    {
        if (ModelState.IsValid)
        {
            _db.Employees.Add(emp);
            _db.SaveChanges();
        }
        return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
    }

    [HttpPost]
    public JsonResult Update([DataSourceRequest] DataSourceRequest request,
                             Employee emp)
    {
        if (ModelState.IsValid)
        {
            _db.Employees.Update(emp);
            _db.SaveChanges();
        }
        return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
    }

    [HttpPost]
    public JsonResult Destroy([DataSourceRequest] DataSourceRequest request,
                              Employee emp)
    {
        var item = _db.Employees.Find(emp.Id);
        if (item != null)
        {
            _db.Employees.Remove(item);
            _db.SaveChanges();
        }
        return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
    }
}
```

---

## 🔑 When Does Each Operation Fire?

| User action in Grid                       | DataSource fires                                         |
| ----------------------------------------- | -------------------------------------------------------- |
| Grid loads / changes page                 | `read`                                                 |
| User sorts a column                       | `read`(if serverSorting: true)                         |
| User filters                              | `read`(if serverFiltering: true)                       |
| Clicks "Add New" → fills form → Save    | `create`                                               |
| Clicks "Edit" → changes values → Update | `update`                                               |
| Clicks "Delete" → confirms               | `destroy`                                              |
| `grid.dataSource.sync()`called in JS    | `create`+`update`+`destroy`for all pending changes |

---

## 🔑 The `ToDataSourceResult` Method

This is the most important Kendo server-side method. It does a lot automatically:

```csharp
// Simple form
_db.Employees.ToDataSourceResult(request)
// ↑ Applies paging, sorting, filtering from the request
// ↑ Returns { Data: [...], Total: N }

// With ModelState (for Create/Update)
new[] { emp }.ToDataSourceResult(request, ModelState)
// ↑ Same as above
// ↑ Also includes validation errors from ModelState in the Errors field
// ↑ Grid shows these errors next to the form fields automatically
```

```
WITHOUT ToDataSourceResult — you'd write this manually:
──────────────────────────────────────────────────────
var query  = _db.Employees.AsQueryable();
var total  = query.Count();
var paged  = query.Skip((page - 1) * pageSize).Take(pageSize).ToList();
return Json(new { Data = paged, Total = total });
// Plus implement sort, filter, group... that's a lot of work

WITH ToDataSourceResult — one line:
──────────────────────────────────────────────────────
return Json(_db.Employees.ToDataSourceResult(request));
```

---

## ⚠️ Most Common Mistakes

| Mistake                                                                  | What Happens                                          | Fix                                              |
| ------------------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------ |
| `return Json(_db.Employees.ToList())`instead of `ToDataSourceResult` | Grid shows all rows, paging breaks, total count wrong | Always use `.ToDataSourceResult(request)`      |
| Not passing `ModelState`on Create/Update                               | Validation errors don't show in the popup             | Use `.ToDataSourceResult(request, ModelState)` |
| Returning `new[] { emp }`without the array brackets                    | JSON is wrong shape                                   | Always wrap in `new[] { emp }`                 |
| No `[HttpPost]`attribute                                               | 404 or 405 error on Read/Create/Update/Destroy        | Add `[HttpPost]`to all four actions            |
| Missing `using Kendo.Mvc.Extensions`                                   | `ToDataSourceResult`not found (compile error)       | Add the using statement                          |

---

## ❓ Interview Questions

**Q: What does `[DataSourceRequest]` do?**

> It's a model binder attribute from Kendo.Mvc.UI that reads the paging, sorting, and filtering parameters from the POST body and packages them into a `DataSourceRequest` object.

**Q: What does `.ToDataSourceResult(request)` do?**

> It takes an `IQueryable`, applies the paging, sorting, and filtering from the request, and returns a `DataSourceResult` object with a `Data` array and `Total` count — exactly the JSON structure Kendo expects.

**Q: Why do Create and Update return `new[] { employee }` instead of the whole list?**

> After saving one record, Kendo only needs the saved record back (with its DB-assigned ID for Create, or to confirm the values for Update). Returning the whole list would be wasteful.

**Q: Why must you pass `ModelState` to `ToDataSourceResult` in Create/Update?**

> So that server-side validation errors (from Data Annotations like `[Required]`) are included in the `Errors` field of the response JSON. Kendo reads this and displays the errors next to the correct fields in the edit popup automatically.

**Q: What HTTP method should Read, Create, Update, and Destroy use?**

> All four should use POST in Kendo MVC applications. Read uses POST because the paging/sorting/filter data is sent in the body, which GET doesn't support reliably.
>
