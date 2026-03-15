
# 01 — Model Binding

---

## 🎯 One-Line Definition

> **Model Binding is ASP.NET Core's automatic process of reading data from an incoming HTTP request — URL segments, query strings, form fields, JSON body, headers — and mapping it into your action method's parameters so you never manually parse `Request.Form["Name"]` or `Request.QueryString["page"]`.**

---

## 🔷 What Model Binding Solves

```
WITHOUT Model Binding — what you'd write manually:
──────────────────────────────────────────────────────────────
[HttpPost]
public IActionResult Create()
{
    // Read from form body manually:
    string name   = Request.Form["EmpName"];
    string salary = Request.Form["Salary"];
    string deptId = Request.Form["DepartmentId"];

    // Parse types manually — crashes if value is wrong:
    decimal sal  = decimal.Parse(salary);
    int     dept = int.Parse(deptId);

    // Build object manually:
    var emp = new Employee
    {
        EmpName      = name,
        Salary       = sal,
        DepartmentId = dept
    };

    // That was 10 lines just to READ the input
}

WITH Model Binding — what you actually write:
──────────────────────────────────────────────────────────────
[HttpPost]
public IActionResult Create(Employee emp)
{
    // emp.EmpName, emp.Salary, emp.DepartmentId are ALL populated.
    // Zero manual parsing. Zero manual type conversion.
    // 1 parameter declaration.
}
```

---

## 🔷 Where Data Can Come From — The 5 Sources

```
Every HTTP Request has these data sources:
──────────────────────────────────────────────────────────────

1. ROUTE SEGMENT          /Employee/Details/5
                                             ↑ "5" captured as id

2. QUERY STRING           /Employee/Index?page=2&dept=HR&active=true
                                           ↑    ↑      ↑  key=value pairs

3. REQUEST BODY
   ├─ Form Data           POST body, Content-Type: application/x-www-form-urlencoded
   │                      EmpName=John&Salary=50000&DepartmentId=2
   └─ JSON Body           POST body, Content-Type: application/json
                          { "empName": "John", "salary": 50000 }

4. HTTP HEADERS           Authorization: Bearer eyJ...
                          X-Api-Key: abc123

5. COOKIES (rare)         Set-Cookie: session=abc
```

---

## 🔷 The Binding Source Attributes

```csharp
// ── [FromRoute] — from URL segment ────────────────────────────────
// URL: GET /Employee/Details/5
[HttpGet("{id:int}")]
public IActionResult Details([FromRoute] int id)
// id = 5  ← from the {id} route segment

// ── [FromQuery] — from query string ───────────────────────────────
// URL: GET /Employee/Index?page=2&dept=HR
[HttpGet]
public IActionResult Index(
    [FromQuery] int    page = 1,
    [FromQuery] string dept = null)
// page = 2, dept = "HR"

// ── [FromBody] — from JSON body ────────────────────────────────────
// Body: { "empName": "John", "salary": 50000 }
// Header: Content-Type: application/json
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
// emp.EmpName = "John", emp.Salary = 50000

// ── [FromForm] — from HTML form POST ──────────────────────────────
// Body: EmpName=John&Salary=50000  (standard form submit)
// Header: Content-Type: application/x-www-form-urlencoded
[HttpPost]
public IActionResult Create([FromForm] Employee emp)
// emp.EmpName = "John", emp.Salary = 50000

// ── [FromHeader] — from HTTP request header ───────────────────────
// Header: X-Api-Key: secret-123
[HttpGet]
public IActionResult GetSecure(
    [FromHeader(Name = "X-Api-Key")] string apiKey)
// apiKey = "secret-123"

// ── [FromServices] — from DI container ────────────────────────────
// Injects a service into just ONE action (not all actions in controller)
[HttpGet]
public IActionResult ExportReport(
    [FromServices] IReportExporter exporter)
// exporter = resolved from DI, for this action only
```

---

## 🔷 Automatic Inference — What ASP.NET Core Figures Out For You

```
With [ApiController] on an API controller:
──────────────────────────────────────────────────────────────

PARAMETER TYPE          IN ROUTE TEMPLATE?    INFERRED SOURCE
─────────────────────────────────────────────────────────────
int, string, bool,          YES               [FromRoute]
decimal, Guid, DateTime
─────────────────────────────────────────────────────────────
int, string, bool,          NO                [FromQuery]
decimal, Guid, DateTime
─────────────────────────────────────────────────────────────
Complex class               NO (POST/PUT)     [FromBody]
─────────────────────────────────────────────────────────────
IFormFile                   —                 [FromForm]
─────────────────────────────────────────────────────────────
```

```csharp
// These two are IDENTICAL (with [ApiController]):

// Explicit:
[HttpGet("{id:int}")]
public IActionResult GetById([FromRoute] int id, [FromQuery] string format) { }

// Implicit — [ApiController] infers the same:
[HttpGet("{id:int}")]
public IActionResult GetById(int id, string format) { }
// id     → in route template → [FromRoute]
// format → not in route      → [FromQuery]

// Complex type on POST — implicit [FromBody]:
[HttpPost]
public IActionResult Create(Employee emp) { }
// emp → complex type, POST → [FromBody] inferred
```

---

## 🔷 Binding Simple Types — All Scenarios

```csharp
// ── From route ──────────────────────────────────────────────────
// URL: /Employee/Details/5
[HttpGet("{id:int}")]
public IActionResult Details(int id)      // id = 5

// ── From query string ──────────────────────────────────────────
// URL: /Employee/Index?page=2&dept=HR&active=true
public IActionResult Index(
    int    page   = 1,
    string dept   = null,
    bool   active = true)
// page = 2, dept = "HR", active = true

// ── Optional with nullable type ────────────────────────────────
// URL: /Employee/Index?minSalary=30000  (maxSalary absent)
public IActionResult Index(decimal? minSalary, decimal? maxSalary)
// minSalary = 30000, maxSalary = null  ← null if not in URL

// ── Array from query string ────────────────────────────────────
// URL: /Reports/Generate?ids=1&ids=3&ids=5&ids=9
public IActionResult Generate(int[] ids)
// ids = [1, 3, 5, 9]

public IActionResult Generate(List<int> ids)
// ids = [1, 3, 5, 9]

// ── DateTime from query string ─────────────────────────────────
// URL: /Reports/Range?from=2024-01-01&to=2024-12-31
public IActionResult Range(DateTime from, DateTime to)
// from = Jan 1 2024, to = Dec 31 2024
```

---

## 🔷 Binding Complex Types — Model Classes

```csharp
// ── From HTML form POST ─────────────────────────────────────────
// Form fields: EmpName=John, Salary=50000, DepartmentId=2, IsActive=true
[HttpPost]
public IActionResult Create(Employee emp)
// emp.EmpName      = "John"
// emp.Salary       = 50000
// emp.DepartmentId = 2
// emp.IsActive     = true
// Field names must match property names (case-insensitive)

// ── From JSON Body (AJAX / Kendo) ───────────────────────────────
// Body: { "empName": "John", "salary": 50000, "departmentId": 2 }
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
// Same result — JSON keys match property names (case-insensitive)

// ── Nested object from JSON ─────────────────────────────────────
public class CreateOrderRequest
{
    public int         CustomerId { get; set; }
    public List<int>   ProductIds { get; set; }
    public Address     Shipping   { get; set; }
}
public class Address
{
    public string Street { get; set; }
    public string City   { get; set; }
}

// JSON Body:
// {
//   "customerId": 5,
//   "productIds": [1, 2, 3],
//   "shipping": {
//     "street": "123 MG Road",
//     "city": "Mumbai"
//   }
// }
[HttpPost]
public IActionResult PlaceOrder([FromBody] CreateOrderRequest req)
// req.CustomerId       = 5
// req.ProductIds       = [1, 2, 3]
// req.Shipping.Street  = "123 MG Road"
// req.Shipping.City    = "Mumbai"
```

---

## 🔷 Combining Sources in One Action

```csharp
// GET /api/departments/2/employees?skip=0&take=10&active=true
// Header: X-Tenant-Id: corp-42
[HttpGet("{deptId:int}/employees")]
public IActionResult GetDeptEmployees(
    [FromRoute]  int    deptId,                              // from URL: 2
    [FromQuery]  int    skip     = 0,                        // from ?skip=0
    [FromQuery]  int    take     = 10,                       // from ?take=10
    [FromQuery]  bool   active   = true,                     // from ?active=true
    [FromHeader(Name = "X-Tenant-Id")] string tenantId = null) // from header
{
    // deptId   = 2
    // skip     = 0
    // take     = 10
    // active   = true
    // tenantId = "corp-42"
}

// PUT /api/employees/5
// Body: { "empId": 5, "empName": "John Updated", "salary": 60000 }
[HttpPut("{id:int}")]
public IActionResult Update(
    [FromRoute] int      id,     // from URL: 5
    [FromBody]  Employee emp)    // from JSON body
{
    if (id != emp.EmpId)
        return BadRequest(new { message = "Route id does not match body id" });
    // id = 5, emp.EmpId = 5, emp.EmpName = "John Updated"
}
```

---

## 🔷 Model Binding in MVC vs API — Key Difference

```
┌──────────────────────────────────┬──────────────────────────────────────┐
│  MVC Controller                  │  API Controller                      │
│  (HTML form submit)              │  (AJAX / Kendo JSON)                 │
├──────────────────────────────────┼──────────────────────────────────────┤
│  Client sends:                   │  Client sends:                       │
│  Content-Type:                   │  Content-Type:                       │
│  application/x-www-form-urlenc.. │  application/json                    │
│                                  │                                      │
│  Body:                           │  Body:                               │
│  EmpName=John&Salary=50000       │  { "empName": "John",               │
│                                  │    "salary": 50000 }                 │
│                                  │                                      │
│  Action parameter:               │  Action parameter:                   │
│  Create(Employee emp)            │  Create([FromBody] Employee emp)     │
│  ← [FromForm] inferred           │  ← [FromBody] explicit or inferred  │
│     (no [ApiController])         │     (with [ApiController])           │
│                                  │                                      │
│  Both result in the same         │  emp object populated                │
│  populated emp object            │                                      │
└──────────────────────────────────┴──────────────────────────────────────┘
```

---

## 🔷 Binding Failures — What Happens

```
SCENARIO 1: Type mismatch
URL: /Employee/Details/abc
Route: {id:int}
→ Route constraint fails BEFORE binding → 404 returned
→ Action never called

SCENARIO 2: Invalid format in query string
URL: /Employee/Index?page=xyz
Parameter: int page
→ Binding fails (can't parse "xyz" as int)
→ With [ApiController]: 400 Bad Request returned automatically
→ Without [ApiController]: page = 0 (default), ModelState has error

SCENARIO 3: Missing required parameter
URL: GET /api/employees/  (no id)
Route: {id:int}  (required, no ?)
→ Route doesn't match → 404

SCENARIO 4: Missing [FromBody] / wrong Content-Type
AJAX POST without contentType: 'application/json'
→ [FromBody] parameter = null (server doesn't know body is JSON)
→ Null crashes your code or [Required] validation fails → 400
```

---

## 🔷 The Two Common AJAX Mistakes

```javascript
// ── Mistake 1: Missing contentType ─────────────────────────────
// ❌ WRONG — [FromBody] receives null
$.ajax({
    url:  '/api/employees',
    type: 'POST',
    data: JSON.stringify(empData)
    // Missing: contentType: 'application/json'
});
// Server: "I received a body but I don't know it's JSON"
// Result: emp = null → NullReferenceException or 400

// ✅ CORRECT
$.ajax({
    url:         '/api/employees',
    type:        'POST',
    contentType: 'application/json',    // ← REQUIRED
    data:        JSON.stringify(empData) // ← REQUIRED
});

// ── Mistake 2: Missing JSON.stringify ──────────────────────────
// ❌ WRONG — sends "[object Object]" as body text
$.ajax({
    url:         '/api/employees',
    type:        'POST',
    contentType: 'application/json',
    data:        empData               // ← missing stringify
});
// Server receives: "[object Object]" → can't deserialize → 400

// ✅ CORRECT
data: JSON.stringify(empData)         // converts JS object → JSON string
```

---

## 🔷 Security — Mass Assignment Attack and Prevention

```csharp
// ── THE RISK ───────────────────────────────────────────────────
public class Employee
{
    public int     EmpId   { get; set; }
    public string  EmpName { get; set; }
    public decimal Salary  { get; set; }
    public bool    IsAdmin { get; set; }   // ← sensitive
}

// If you bind directly from request body:
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
// Attacker sends: { "empName": "Hacker", "salary": 999999, "isAdmin": true }
// emp.IsAdmin = true ← bound!  → attacker becomes admin

// ── PREVENTION 1: Use a separate input DTO ─────────────────────
public class CreateEmployeeRequest
{
    public string  EmpName      { get; set; }
    public decimal Salary       { get; set; }
    public int     DepartmentId { get; set; }
    // No EmpId, no IsAdmin — user can't set those
}

[HttpPost]
public IActionResult Create([FromBody] CreateEmployeeRequest req)
{
    var emp = new Employee
    {
        EmpName      = req.EmpName,
        Salary       = req.Salary,
        DepartmentId = req.DepartmentId,
        IsAdmin      = false,           // set server-side, not from request
        CreatedAt    = DateTime.UtcNow
    };
    _bal.Add(emp);
}

// ── PREVENTION 2: [BindNever] on sensitive properties ─────────
public class Employee
{
    public int     EmpId   { get; set; }
    public string  EmpName { get; set; }
    public decimal Salary  { get; set; }

    [BindNever]  // ← never bound from any request
    public bool IsAdmin { get; set; }

    [BindNever]
    public DateTime CreatedAt { get; set; }
}

// ── PREVENTION 3: [Bind] whitelist ────────────────────────────
[HttpPost]
public IActionResult Create(
    [Bind("EmpName,Salary,DepartmentId")] Employee emp)
// ONLY these 3 properties bound — IsAdmin, EmpId IGNORED
```

---

## 🔷 Model Binding Order — How ASP.NET Core Decides

```
When you write:  public IActionResult GetById(int id)

ASP.NET Core searches in this order until it finds the value:
──────────────────────────────────────────────────────────────
1. Form values       (Request.Form["id"])
2. Route values      (URL route segment {id})
3. Query string      (Request.QueryString["id"])

FIRST match wins. Stops searching after finding a value.

WHY this order matters:
  Form POST: /Employee/Edit/5  with form field id=99
  → Form value "99" wins (form is checked first)
  → Route value "5" is ignored
  → Use [FromRoute] explicitly to force route value
```

---

## ⭐ Interview Quick-Fire

| Question                                                                | Answer                                                                                                                                        |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| What is Model Binding?                                                  | ASP.NET Core's automatic process of reading HTTP request data (route, query string, body, headers) and mapping it to action method parameters |
| What are the 5 binding sources?                                         | Route segment, query string, request body (form or JSON), HTTP headers, cookies                                                               |
| What attribute reads data from the JSON body?                           | `[FromBody]`                                                                                                                                |
| What attribute reads data from the URL segment?                         | `[FromRoute]`                                                                                                                               |
| What does `[ApiController]`change about binding?                      | Auto-infers `[FromBody]`for complex types in POST/PUT, and `[FromQuery]`for simple types not in the route                                 |
| Why does forgetting `contentType: 'application/json'`break AJAX POST? | Without it, the server doesn't know the body is JSON so `[FromBody]`receives null                                                           |
| What is a Mass Assignment Attack?                                       | Attacker POSTs extra fields like `isAdmin: true`that get bound to sensitive model properties                                                |
| What are 3 ways to prevent Mass Assignment?                             | Use a separate input DTO,`[BindNever]`on sensitive properties, or `[Bind("Prop1,Prop2")]`whitelist                                        |
| What is the default binding order when no attribute is specified?       | Form values → Route values → Query string (first match wins)                                                                                |
| What happens when binding fails for a type?                             | With `[ApiController]`: 400 returned automatically. Without it: parameter gets default value and `ModelState`has the error                |
