
# 05 — Passing Data to Controller

---

## 🎯 One-Line Definition

> **There are four places data can come from in an HTTP request — the URL path, the query string, the request body, and headers — and ASP.NET has a binding attribute for each one.**

---

## 🗺️ The Four Ways Data Reaches a Controller

```
REQUEST
────────────────────────────────────────────────────────────
URL:    /Employee/5/projects?page=2&dept=IT
        │        │           └──────────────── Query String
        │        └────────────────────────── Route segment
        │
BODY:   {"name":"Alice","salary":75000}    ← Request Body
HEADER: Authorization: Bearer eyJh...     ← Headers
```

```
ATTRIBUTE    READS FROM         USE FOR
──────────── ─────────────────  ─────────────────────────────
[FromRoute]  URL path segment   /Employee/{id} → id
[FromQuery]  URL query string   ?page=2&dept=IT → page, dept
[FromBody]   Request body JSON  {"name":"Alice"} → Employee obj
[FromHeader] Request headers    Authorization header → token
```

---

## 🔑 1 — `[FromRoute]` — Data in the URL Path

```javascript
// JavaScript — ID is in the URL path
$.get("/api/employee/5", function(emp) {
    populateForm(emp);
});

$.ajax({
    url:  "/api/employee/5/projects",
    type: "GET",
    success: function(projects) { renderProjects(projects); }
});
```

```csharp
// Route template: [controller]/{id}
[HttpGet("{id}")]
public IActionResult GetById([FromRoute] int id)
// id comes from the 5 in /api/employee/5
{
    var emp = _db.Employees.Find(id);
    return emp == null ? NotFound() : Ok(emp);
}

// Multiple route parameters
[HttpGet("{empId}/projects/{projectId}")]
public IActionResult GetProject(
    [FromRoute] int empId,
    [FromRoute] int projectId)
{
    ...
}

// [FromRoute] is optional when name matches — ASP.NET infers it
[HttpGet("{id}")]
public IActionResult GetById(int id)  // same result without [FromRoute]
{...}
```

---

## 🔑 2 — `[FromQuery]` — Data in the Query String

```javascript
// Parameters become ?key=value in the URL
$.get("/Employee/Search", {
    name:       "Alice",
    department: "IT",
    page:       1,
    pageSize:   10
}, function(data) {
    renderTable(data);
});
// URL: /Employee/Search?name=Alice&department=IT&page=1&pageSize=10

$.ajax({
    url:  "/Employee/GetAll",
    type: "GET",
    data: { activeOnly: true, sortBy: "name" },
    success: function(data) { renderTable(data); }
});
// URL: /Employee/GetAll?activeOnly=true&sortBy=name
```

```csharp
[HttpGet]
public JsonResult Search(
    [FromQuery] string name,
    [FromQuery] string department,
    [FromQuery] int page     = 1,
    [FromQuery] int pageSize = 10)
{
    var query = _db.Employees.AsQueryable();

    if (!string.IsNullOrEmpty(name))
        query = query.Where(e => e.Name.Contains(name));

    if (!string.IsNullOrEmpty(department))
        query = query.Where(e => e.Department == department);

    return Json(query.Skip((page - 1) * pageSize).Take(pageSize).ToList());
}

// Bind multiple query params to one object
public class SearchFilters
{
    public string Name       { get; set; }
    public string Department { get; set; }
    public int    Page       { get; set; } = 1;
    public int    PageSize   { get; set; } = 10;
}

[HttpGet]
public JsonResult Search([FromQuery] SearchFilters filters)
// /Employee/Search?name=Alice&page=2 → filters.Name="Alice", filters.Page=2
{...}
```

---

## 🔑 3 — `[FromBody]` — Data in the Request Body

```javascript
// REQUIRED: contentType + JSON.stringify when using [FromBody]
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",           // ← tells server: body is JSON
    data:        JSON.stringify({
                     name:       "Alice",
                     department: "IT",
                     salary:     75000,
                     isActive:   true
                 }),                           // ← convert object to string
    success: function(response) {
        showNotification("Created: " + response.id);
    }
});

// Sending a nested object
$.ajax({
    url:         "/Employee/CreateWithAddress",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify({
                     name:    "Alice",
                     address: { city: "New York", zip: "10001" },
                     skills:  ["C#", "SQL"]
                 }),
    success: function(r) { console.log(r); }
});
```

```csharp
[HttpPost]
public JsonResult Create([FromBody] Employee employee)
// [FromBody] reads the JSON string from the body and deserializes it
{
    if (!ModelState.IsValid)
        return Json(new { success = false, errors = GetErrors() });

    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true, id = employee.Id });
}

// Without [FromBody] — uses form-encoded binding (no JSON.stringify needed)
[HttpPost]
public JsonResult Create(Employee employee)
// reads from form body: name=Alice&department=IT&salary=75000
{...}
```

---

## 🔑 Form-Encoded vs JSON Body — The Key Decision

```
FORM-ENCODED (no [FromBody])          JSON BODY ([FromBody])
──────────────────────────────────    ──────────────────────────────────
$.post(url, { name: "Alice" })        $.ajax({ contentType: "application/json",
                                               data: JSON.stringify(obj) })

Travels as:                           Travels as:
name=Alice&dept=IT&salary=75000       {"name":"Alice","dept":"IT","salary":75000}

Controller:                           Controller:
public ActionResult Save(Employee e)  public ActionResult Save([FromBody] Employee e)

Works with: nested objects? ❌        Works with: nested objects? ✅
Works with: arrays? ❌                Works with: arrays? ✅
Complexity: simple flat objects       Complexity: any structure
```

**Rule:** Simple flat objects → form-encoded (simpler). Nested objects, arrays, complex data → JSON body with `[FromBody]`.

---

## 🔑 4 — `[FromHeader]` — Data in Request Headers

```javascript
// Send data as a custom header
$.ajax({
    url:  "/Employee/GetAll",
    type: "GET",
    headers: {
        "X-Company-Id": getCurrentCompanyId(),
        "Authorization": "Bearer " + getAuthToken()
    },
    success: function(data) { renderTable(data); }
});
```

```csharp
[HttpGet]
public IActionResult GetAll(
    [FromHeader(Name = "X-Company-Id")] int companyId,
    [FromHeader(Name = "Authorization")] string authHeader)
{
    var employees = _db.Employees
        .Where(e => e.CompanyId == companyId)
        .ToList();
    return Ok(employees);
}
```

---

## 🔑 Mixing Sources — Route + Query + Body Together

```javascript
// PUT /api/employee/5 with JSON body
$.ajax({
    url:         "/api/employee/5",        // 5 → [FromRoute]
    type:        "PUT",
    contentType: "application/json",
    data:        JSON.stringify({ id: 5, name: "Alice Updated", salary: 80000 }),
    success: function(r) { console.log(r); }
});
```

```csharp
[HttpPut("{id}")]
public IActionResult Update(
    [FromRoute] int id,           // from URL path
    [FromBody]  Employee employee) // from request body
{
    if (id != employee.Id) return BadRequest("ID mismatch");

    _db.Employees.Update(employee);
    _db.SaveChanges();
    return Ok(employee);
}
```

---

## 🔑 Sending an Array to the Controller

```javascript
// Send array of IDs to delete multiple records
$.ajax({
    url:         "/Employee/DeleteMany",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify([1, 5, 7, 12]),  // array of IDs
    success: function(r) { showNotification("Deleted " + r.count + " records"); }
});

// Send array of objects
$.ajax({
    url:         "/Employee/ImportMany",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify([
                     { name: "Alice", dept: "IT" },
                     { name: "Bob",   dept: "HR" }
                 ]),
    success: function(r) { showNotification("Imported " + r.count); }
});
```

```csharp
// Receive array of IDs
[HttpPost]
public JsonResult DeleteMany([FromBody] List<int> ids)
{
    var employees = _db.Employees.Where(e => ids.Contains(e.Id)).ToList();
    _db.Employees.RemoveRange(employees);
    _db.SaveChanges();
    return Json(new { count = employees.Count });
}

// Receive array of objects
[HttpPost]
public JsonResult ImportMany([FromBody] List<Employee> employees)
{
    _db.Employees.AddRange(employees);
    _db.SaveChanges();
    return Json(new { count = employees.Count });
}
```

---

## 📊 Binding Attributes Quick Reference

| Attribute        | Data Source  | Example URL/Data          | Use For                |
| ---------------- | ------------ | ------------------------- | ---------------------- |
| `[FromRoute]`  | URL segment  | `/employee/5`→`id=5` | Resource ID in URL     |
| `[FromQuery]`  | Query string | `?page=2&dept=IT`       | Filters, pagination    |
| `[FromBody]`   | Request body | `{"name":"Alice"}`      | Full objects, POST/PUT |
| `[FromForm]`   | Form fields  | `name=Alice&dept=IT`    | HTML form submits      |
| `[FromHeader]` | HTTP header  | `X-Company-Id: 3`       | Auth tokens, tenant ID |

---

## ⚠️ Common Mistakes

| Mistake                                                 | Symptom                                 | Fix                                           |
| ------------------------------------------------------- | --------------------------------------- | --------------------------------------------- |
| `[FromBody]`but no `contentType:"application/json"` | Controller receives null model          | Add `contentType: "application/json"`       |
| `[FromBody]`but no `JSON.stringify()`               | Body is `[object Object]`, null model | Wrap data with `JSON.stringify()`           |
| Sending array without `[FromBody]`                    | Array is empty in controller            | Arrays always need `[FromBody]`+ stringify  |
| Route param name doesn't match action param             | 0 or error, binding fails               | Names must match exactly:`{id}`→`int id` |
| Sending nested object without `[FromBody]`            | Nested object properties are null       | Use `[FromBody]`for nested objects          |

---

## ❓ Interview Questions

**Q: What is the difference between `[FromQuery]` and `[FromBody]`?**

> `[FromQuery]` reads values from the URL query string (`?key=value`). `[FromBody]` reads and deserializes a JSON string from the request body. Use `[FromQuery]` for GET parameters and `[FromBody]` for POST/PUT with complex objects.

**Q: When must you use `[FromBody]`?**

> When you need to send a JSON body — complex objects, nested objects, or arrays. It requires `contentType: "application/json"` and `JSON.stringify(data)` on the JavaScript side.

**Q: Can you mix `[FromRoute]` and `[FromBody]` in the same action?**

> Yes. For example a PUT action: `[HttpPut("{id}")]` with `[FromRoute] int id` and `[FromBody] Employee employee` — the ID comes from the URL path and the updated data comes from the JSON body.

**Q: Why does sending an array to a controller require `[FromBody]`?**

> Normal model binding can't construct a list from query string parameters or form fields in a meaningful way. `[FromBody]` reads the JSON array from the body and deserializes it into a `List<T>`.
>
