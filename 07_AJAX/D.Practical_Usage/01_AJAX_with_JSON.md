
# 01 — AJAX with JSON

---

## 🎯 One-Line Definition

> **AJAX with JSON means: JavaScript sends an object to the server as a JSON string, and the server sends data back as a JSON string — both sides convert between string and object automatically.**

---

## 🔄 The Complete Flow

```
BROWSER                                        SERVER
───────                                        ──────

JavaScript Object                              C# Object
{ name:"Alice", salary:75000 }                 Employee { Name="Alice", Salary=75000 }
        │                                              │
        │  JSON.stringify()                            │  JsonSerializer.Serialize()
        ▼                                              ▼
JSON String ──── travels over HTTP ────────► JSON String
'{"name":"Alice","salary":75000}'             '{"name":"Alice","salary":75000}'
        ▲                                              │
        │  jQuery auto-parses                          │  [FromBody] deserializes
        │  (or JSON.parse())                           ▼
JavaScript Object                              C# Object ready to use
```

Both sides do the same thing — convert between object and string.
The string is what travels. The object is what you use in code.

---

## 🔑 Sending JSON to the Controller — Two Approaches

### Approach 1 — Form-encoded (default, simpler)

Data travels as `key=value&key=value` in the POST body.
No `JSON.stringify`, no `contentType` change needed.
Works with normal model binding in the controller.

```javascript
// ── JavaScript ────────────────────────────────────────────────
$.post("/Employee/Create", {
    name:       "Alice",
    department: "IT",
    salary:     75000,
    isActive:   true
}, function(response) {
    console.log(response);
});

// What actually travels to server:
// name=Alice&department=IT&salary=75000&isActive=true
```

```csharp
// ── Controller ────────────────────────────────────────────────
[HttpPost]
public JsonResult Create(Employee employee)
// ↑ No [FromBody] needed — form-encoded is default model binding
{
    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true, id = employee.Id });
}
```

---

### Approach 2 — JSON body (when you need [FromBody])

Data travels as a JSON string in the POST body.
Requires `JSON.stringify` and `contentType: "application/json"`.
Required when the controller uses `[FromBody]`.

```javascript
// ── JavaScript ────────────────────────────────────────────────
var employee = {
    name:       "Alice",
    department: "IT",
    salary:     75000,
    isActive:   true
};

$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",        // ← tell server it's JSON
    data:        JSON.stringify(employee),  // ← convert object to string
    dataType:    "json",
    success: function(response) {
        console.log(response);
    }
});

// What actually travels to server:
// {"name":"Alice","department":"IT","salary":75000,"isActive":true}
```

```csharp
// ── Controller ────────────────────────────────────────────────
[HttpPost]
public JsonResult Create([FromBody] Employee employee)
// ↑ [FromBody] required — reads JSON from request body
{
    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true, id = employee.Id });
}
```

---

## 🔑 Receiving JSON from the Controller

```csharp
// Controller returns JSON — various ways:

// Return a list
public JsonResult GetAll()
{
    var employees = _db.Employees.ToList();
    return Json(employees);
    // → [{"id":1,"name":"Alice",...}, {"id":2,"name":"Bob",...}]
}

// Return a single object
public JsonResult GetById(int id)
{
    var emp = _db.Employees.Find(id);
    return Json(emp);
    // → {"id":1,"name":"Alice","department":"IT","salary":75000}
}

// Return a custom response shape
public JsonResult Save(Employee emp)
{
    _db.Employees.Add(emp);
    _db.SaveChanges();
    return Json(new {
        success    = true,
        id         = emp.Id,
        message    = "Employee saved",
        savedAt    = DateTime.Now
    });
    // → {"success":true,"id":26,"message":"Employee saved","savedAt":"..."}
}
```

```javascript
// jQuery auto-parses the JSON — data is already a JS object
$.get("/Employee/GetAll", function(data) {
    // data = JavaScript array — no JSON.parse needed
    data.forEach(function(emp) {
        console.log(emp.name + " — " + emp.department);
    });
});

$.get("/Employee/GetById", { id: 5 }, function(emp) {
    // emp = JavaScript object
    $("#Name").val(emp.name);
    $("#Department").val(emp.department);
    $("#Salary").val(emp.salary);
});
```

---

## 🔑 JSON Response Shapes — What Your Controller Returns

Keep your JSON responses consistent across the whole project. Here are the standard shapes:

### Shape 1 — Simple data (for reads)

```json
[
  { "id": 1, "name": "Alice", "department": "IT" },
  { "id": 2, "name": "Bob",   "department": "HR" }
]
```

### Shape 2 — Success/fail wrapper (for writes)

```json
{ "success": true,  "id": 26, "message": "Saved successfully" }
{ "success": false, "message": "Name is required" }
```

### Shape 3 — Kendo Grid shape (for Kendo DataSource)

```json
{ "Data": [...], "Total": 150, "Errors": null }
```

### Shape 4 — Validation errors (for form validation display)

```json
{
  "success": false,
  "errors": {
    "Name":   ["Name is required"],
    "Salary": ["Salary must be positive"]
  }
}
```

---

## 🔑 Sending Nested / Complex JSON

```javascript
// Nested object — employee with address
var payload = {
    name:       "Alice",
    department: "IT",
    address: {
        street: "123 Main St",
        city:   "New York",
        zip:    "10001"
    },
    skills: ["C#", "SQL", "JavaScript"]
};

$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify(payload),
    success: function(response) { console.log(response); }
});
```

```csharp
// Matching C# models
public class Employee
{
    public string        Name       { get; set; }
    public string        Department { get; set; }
    public Address       Address    { get; set; }
    public List<string>  Skills     { get; set; }
}

public class Address
{
    public string Street { get; set; }
    public string City   { get; set; }
    public string Zip    { get; set; }
}

[HttpPost]
public JsonResult Create([FromBody] Employee employee)
{
    Console.WriteLine(employee.Address.City);  // "New York"
    Console.WriteLine(employee.Skills[0]);     // "C#"
    ...
}
```

---

## 🔑 Dates in JSON — The Tricky Part

Dates have no native JSON type. They travel as strings.

```csharp
// Controller returns a date
public JsonResult GetEmployee(int id)
{
    var emp = _db.Employees.Find(id);
    return Json(emp);
}
// HireDate becomes a string in JSON:
// "hireDate": "2021-06-15T00:00:00"
```

```javascript
$.get("/Employee/GetById", { id: 1 }, function(emp) {
    // emp.hireDate is a STRING, not a Date object
    console.log(typeof emp.hireDate);    // "string"
    console.log(emp.hireDate);           // "2021-06-15T00:00:00"

    // Convert to JS Date when you need to work with it
    var date = new Date(emp.hireDate);
    console.log(date.getFullYear());     // 2021

    // Format for display
    var formatted = date.toLocaleDateString("en-US");  // "6/15/2021"

    // Set in a Kendo DatePicker
    $("#HireDate").data("kendoDatePicker").value(date);
});
```

---

## ⚠️ Most Common AJAX + JSON Mistakes

| Mistake                                                   | Symptom                           | Fix                                                          |
| --------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------ |
| No `contentType: "application/json"`with `[FromBody]` | Controller receives null model    | Add `contentType: "application/json"`                      |
| No `JSON.stringify()`with `[FromBody]`                | Body is `[object Object]`string | Wrap data in `JSON.stringify()`                            |
| Using `[FromBody]`but sending form-encoded              | Model is null                     | Either remove `[FromBody]`or add the two required settings |
| Not checking `response.success`                         | Treating all responses as success | Always check the flag before acting                          |
| Sending dates as strings without formatting               | Date mismatch or parse error      | Format consistently:`"yyyy-MM-dd"`or ISO 8601              |

---

## ❓ Interview Questions

**Q: What are the two ways to send data to a controller via AJAX POST?**

> Form-encoded (default): pass a plain JS object to `$.post`, controller uses normal model binding. JSON body: use `$.ajax` with `contentType: "application/json"` and `JSON.stringify(data)`, controller uses `[FromBody]`.

**Q: Does jQuery automatically parse JSON responses?**

> Yes — when the server returns `Content-Type: application/json`, jQuery automatically parses the response body into a JavaScript object. You receive the object directly in the callback, no `JSON.parse()` needed.

**Q: Why must you call `JSON.stringify()` when sending JSON to `[FromBody]`?**

> AJAX request bodies must be strings. `JSON.stringify()` converts your JavaScript object into a JSON string. Without it, the body contains `[object Object]` which the server cannot deserialize.

**Q: How do you handle dates when passing them between JavaScript and ASP.NET?**

> Dates have no native JSON type — they travel as ISO 8601 strings (e.g., `"2021-06-15T00:00:00"`). On the JavaScript side, wrap them in `new Date()` to work with them. On the ASP.NET side, `DateTime` properties deserialize automatically from ISO strings.
>
