
# 05 — Model Binding in Controllers

---

## 🎯 One-Line Definition

> **Model Binding is ASP.NET Core's automatic process of reading data from an incoming HTTP request — URL segments, query strings, form fields, JSON body, headers — and mapping it directly into your action method's parameters, so you never manually parse `Request.QueryString` or `Request.Form`.**

---

## 🔷 What Model Binding Solves

```
WITHOUT Model Binding (what you'd have to do manually):
──────────────────────────────────────────────────────────────
public IActionResult Index()
{
    // Manually read query string:
    string pageStr = Request.Query["page"];
    int page = int.Parse(pageStr ?? "1");

    string dept = Request.Query["dept"];

    // Manually read form POST body:
    string name   = Request.Form["EmpName"];
    string salary = Request.Form["Salary"];
    decimal sal   = decimal.Parse(salary);

    // Manually deserialize JSON body (for AJAX):
    string body = new StreamReader(Request.Body).ReadToEnd();
    var emp = JsonSerializer.Deserialize<Employee>(body);

    // 30+ lines just to READ the input data
}

WITH Model Binding (what you actually write):
──────────────────────────────────────────────────────────────
public IActionResult Index(int page = 1, string dept = null)
{
    // page and dept ARE the query string values — already parsed
}

[HttpPost]
public IActionResult Create(Employee emp)
{
    // emp IS the form data — already deserialized
}
```

---

## 🔷 Where Data Can Come From — The 5 Sources

```
HTTP Request
│
├── 1. Route Segment      /Employee/Details/5
│                                          ↑ this "5" is a route value
│
├── 2. Query String       /Employee/Index?page=2&dept=HR
│                                          ↑          ↑ key=value pairs after ?
│
├── 3. Request Body       POST body with JSON or form data
│   ├── Form Data         Content-Type: application/x-www-form-urlencoded
│   │                     EmpName=John&Salary=50000
│   └── JSON Body         Content-Type: application/json
│                         { "empName": "John", "salary": 50000 }
│
├── 4. HTTP Headers       Authorization: Bearer eyJ...
│                         X-Custom-Header: value
│
└── 5. Cookies            Cookie: session=abc123
```

---

## 🔷 Binding Source Attributes — Explicit Control

```csharp
// ── [FromRoute] — value from URL route segment ─────────────────────
// URL: GET /Employee/Details/5
[HttpGet("{id:int}")]
public IActionResult Details([FromRoute] int id)
//                            ↑ reads from {id} in the route
// id = 5


// ── [FromQuery] — value from query string ──────────────────────────
// URL: GET /Employee/Index?page=2&dept=HR
public IActionResult Index(
    [FromQuery] int    page = 1,
    [FromQuery] string dept = null)
// page = 2, dept = "HR"


// ── [FromBody] — value from JSON body (AJAX/Kendo POST) ─────────────
// Body: { "empName": "John", "salary": 50000 }
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
// emp.EmpName = "John", emp.Salary = 50000


// ── [FromForm] — value from HTML form POST ─────────────────────────
// Body: EmpName=John&Salary=50000  (form submit)
[HttpPost]
public IActionResult Create([FromForm] Employee emp)
// emp.EmpName = "John", emp.Salary = 50000


// ── [FromHeader] — value from HTTP header ──────────────────────────
// Header: X-Employee-Token: abc123
public IActionResult SecureAction(
    [FromHeader(Name = "X-Employee-Token")] string token)
// token = "abc123"


// ── [FromServices] — value from DI container ───────────────────────
// Inject a service directly into an action (instead of constructor)
public IActionResult Export(
    [FromServices] IExportService exportService)
// exportService is resolved from the DI container
// Useful for services only needed in ONE action
```

---

## 🔷 Automatic Inference — What [ApiController] Does For You

With `[ApiController]` on an API controller, ASP.NET Core infers the binding source automatically:

```csharp
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
{
    // Simple type in route → [FromRoute] inferred
    [HttpGet("{id}")]
    public IActionResult GetById(int id)     // ← reads from {id} route segment

    // Simple type NOT in route → [FromQuery] inferred
    [HttpGet]
    public IActionResult GetAll(int skip, int take)  // ← reads from ?skip=0&take=10

    // Complex type in POST/PUT → [FromBody] inferred
    [HttpPost]
    public IActionResult Create(Employee emp)  // ← reads from JSON body

    // IFormFile → [FromForm] inferred
    [HttpPost("upload")]
    public IActionResult Upload(IFormFile file)
}
```

```
[ApiController] Inference Rules:
──────────────────────────────────────────────────────────────
Simple type (int, string, bool, decimal, Guid, DateTime)
  AND exists in route template                 → [FromRoute]

Simple type
  AND NOT in route template                    → [FromQuery]

Complex type (your model classes)
  in POST / PUT                                → [FromBody]

IFormFile                                      → [FromForm]

Without [ApiController]:  you must add attributes manually
With [ApiController]:     inferred automatically
```

---

## 🔷 Binding a Simple Type — All Scenarios

```csharp
// ── From route ───────────────────────────────────────────────────────
// Route: /Employee/Details/5
[HttpGet("{id:int}")]
public IActionResult Details(int id)   // id = 5


// ── From query string ────────────────────────────────────────────────
// URL: /Employee/Index?page=2&dept=HR&active=true
public IActionResult Index(int page = 1, string dept = null, bool active = true)
// page = 2, dept = "HR", active = true


// ── Optional with default value ──────────────────────────────────────
// URL: /Employee/Index  (no query string at all)
public IActionResult Index(int page = 1, int pageSize = 10)
// page = 1 (default), pageSize = 10 (default)


// ── Nullable — when param might not be provided ──────────────────────
// URL: /Employee/Index?dept=HR  (no minSalary)
public IActionResult Index(string dept, decimal? minSalary)
// dept = "HR", minSalary = null  ← nullable, not an error


// ── Multiple same-name values (array from query string) ───────────────
// URL: /Report/Generate?ids=1&ids=2&ids=5
public IActionResult Generate(int[] ids)
// ids = [1, 2, 5]

public IActionResult Generate(List<int> ids)
// ids = [1, 2, 5]
```

---

## 🔷 Binding a Complex Type (Model Class)

```csharp
// Model class:
public class Employee
{
    public int     EmpId      { get; set; }
    public string  EmpName    { get; set; }
    public string  Department { get; set; }
    public decimal Salary     { get; set; }
    public bool    IsActive   { get; set; }
}
```

```csharp
// ── From HTML Form POST ───────────────────────────────────────────────
// Form fields: EmpName=John, Salary=50000, Department=HR
[HttpPost]
public IActionResult Create(Employee emp)
// emp.EmpName = "John"
// emp.Salary  = 50000
// emp.Department = "HR"
// ↑ Form field names must match property names (case-insensitive)


// ── From JSON Body (AJAX / Kendo) ────────────────────────────────────
// Body: { "empName": "John", "salary": 50000, "department": "HR" }
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
// emp.EmpName = "John"
// emp.Salary  = 50000
// emp.Department = "HR"
// ↑ JSON key names must match property names (case-insensitive by default)
```

```csharp
// ── Binding a NESTED object (from JSON body) ─────────────────────────

public class CreateOrderRequest
{
    public int         EmployeeId      { get; set; }
    public List<int>   ProductIds      { get; set; }
    public Address     ShippingAddress { get; set; }
}

public class Address
{
    public string Street { get; set; }
    public string City   { get; set; }
    public string Zip    { get; set; }
}

[HttpPost]
public IActionResult PlaceOrder([FromBody] CreateOrderRequest req)
{
    // req.EmployeeId          = 5
    // req.ProductIds          = [1, 2, 3]
    // req.ShippingAddress.City = "Mumbai"
}

// Client sends:
// {
//   "employeeId": 5,
//   "productIds": [1, 2, 3],
//   "shippingAddress": {
//     "street": "123 MG Road",
//     "city": "Mumbai",
//     "zip": "400001"
//   }
// }
```

---

## 🔷 Combining Multiple Sources in One Action

```csharp
// GET /api/employee/5/reports?year=2024
// Header: X-Tenant-Id: corp-123
[HttpGet("{id:int}/reports")]
public IActionResult GetReports(
    [FromRoute]  int    id,                              // from URL: 5
    [FromQuery]  int    year    = DateTime.Now.Year,     // from ?year=2024
    [FromHeader(Name = "X-Tenant-Id")] string tenantId = null)  // from header
{
    // id = 5, year = 2024, tenantId = "corp-123"
}

// POST /api/employee/5/photo
// Body: multipart/form-data with a file
[HttpPost("{id:int}/photo")]
public IActionResult UploadPhoto(
    [FromRoute] int      id,      // from URL
    [FromForm]  IFormFile photo)  // from multipart form
{
    // id = 5, photo = the uploaded file
}
```

---

## 🔷 Model Binding Failure — What Happens

```
URL: GET /api/employee/abc
Route: {id:int}

Model Binding tries to parse "abc" as int → FAILS
Result: route constraint fails → 404 Not Found
        (action is never called)
```

```
URL: GET /api/employee?page=xyz
Parameter: int page

Model Binding tries to parse "xyz" as int → FAILS
Result: ModelState has error for "page"
        With [ApiController]: automatic 400 Bad Request returned
        Without [ApiController]: page = 0 (default) OR exception
```

```csharp
// Handling binding failure manually (without [ApiController]):
public IActionResult GetAll(int page = 1)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);   // you check manually
    // ...
}

// With [ApiController] on API controller — automatic:
// Bad binding → 400 returned before your action runs
// You never need to check ModelState for binding errors
```

---

## 🔷 Common Mistakes — Binding From AJAX/Kendo

```javascript
// ── MISTAKE 1: Missing contentType ─────────────────────────────────
// ❌ WRONG — server receives null for emp
$.ajax({
    url:  '/api/employee',
    type: 'POST',
    data: JSON.stringify(empData)
    // Missing contentType → server doesn't know it's JSON
});

// ✅ CORRECT
$.ajax({
    url:         '/api/employee',
    type:        'POST',
    contentType: 'application/json',   // ← REQUIRED
    data:        JSON.stringify(empData)
});

// ── MISTAKE 2: Missing JSON.stringify ──────────────────────────────
// ❌ WRONG — sends "[object Object]" as body
data: empData

// ✅ CORRECT — converts JS object to JSON string
data: JSON.stringify(empData)

// ── MISTAKE 3: Route parameter name mismatch ───────────────────────
// Route: [HttpGet("{empId}")]
// Method: public IActionResult GetById(int id)  ← "id" not "empId"
// Result: id = 0 always — binding fails silently

// ✅ Names must match:
// Route: [HttpGet("{id}")]
// Method: public IActionResult GetById(int id)
```

---

## 🔷 [Bind] and [BindNever] — Security Control

```csharp
// SECURITY RISK — Mass Assignment Attack:
// An attacker could POST { "salary": 999999, "isAdmin": true }
// and your model would bind those fields too

// ── [Bind] — whitelist only specific properties ──────────────────
[HttpPost]
public IActionResult Create(
    [Bind("EmpName,Department,Salary")] Employee emp)
//   ↑ ONLY these 3 fields will be bound — EmpId, IsAdmin etc. ignored
{
    // emp.EmpId   = 0    (not bound — user can't set ID)
    // emp.IsAdmin = false (not bound — user can't set admin status)
    // emp.EmpName = "John" (bound — from request)
}


// ── [BindNever] on the model — always exclude a property ─────────
public class Employee
{
    public int     EmpId   { get; set; }

    [BindNever]    // ← never bound from request — only set in code
    public bool    IsAdmin { get; set; }

    public string  EmpName { get; set; }
    public decimal Salary  { get; set; }
}
// Now IsAdmin can never be set by a malicious POST body


// ── BETTER APPROACH — use a separate DTO for input ───────────────
// Separate your API input model from your DB model:
public class CreateEmployeeRequest
{
    [Required]
    public string  EmpName    { get; set; }

    [Required]
    public string  Department { get; set; }

    [Range(0, 999999)]
    public decimal Salary     { get; set; }
    // No EmpId, no IsAdmin, no IsActive — only what the user should provide
}

[HttpPost]
public IActionResult Create([FromBody] CreateEmployeeRequest req)
{
    // Map request to Employee entity in your code:
    var emp = new Employee
    {
        EmpName    = req.EmpName,
        Department = req.Department,
        Salary     = req.Salary,
        IsActive   = true,       // set server-side
        CreatedAt  = DateTime.Now // set server-side
    };
    _bal.Add(emp);
}
```

---

## 🔷 Model Binding vs Model Validation — The Difference

```
Model Binding:         Reading data from request → populating parameters
                       Happens FIRST

Model Validation:      Checking if the populated data follows rules
                       Happens AFTER binding
                       Uses [Required], [Range], [StringLength] etc.
                       Result stored in ModelState

Order:
  1. Request arrives
  2. Model Binding runs → emp object populated from JSON body
  3. Data Annotations validated → ModelState.IsValid set
  4. With [ApiController]: if invalid → 400 returned automatically
  5. Without [ApiController]: your action runs, you check ModelState
```

---

## ⭐ Interview Quick-Fire

| Question                                                                  | Answer                                                                                                                |
| ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| What is Model Binding?                                                    | Automatic mapping of HTTP request data (route, query string, body, headers) into action method parameters             |
| What is `[FromBody]`?                                                   | Tells ASP.NET Core to read the parameter from the JSON request body — required for AJAX/Kendo POST requests          |
| What is `[FromQuery]`?                                                  | Reads parameter from the URL query string (`?page=2&dept=HR`)                                                       |
| What is `[FromRoute]`?                                                  | Reads parameter from the URL route segment (`/employee/5`→`id=5`)                                                |
| What happens if you forget `contentType: 'application/json'`in AJAX?    | `[FromBody]`parameter receives `null`— server doesn't know the body is JSON                                      |
| What happens if route parameter name doesn't match method parameter name? | Binding fails silently — parameter gets default value (0 for int, null for string)                                   |
| What is a Mass Assignment Attack?                                         | Attacker POSTs extra fields (like `isAdmin: true`) that get bound to sensitive model properties                     |
| How do you prevent Mass Assignment?                                       | Use `[Bind("PropA,PropB")]`,`[BindNever]`on sensitive properties, or use a dedicated input DTO                    |
| Does `[ApiController]`change binding behavior?                          | Yes — it automatically infers `[FromBody]`for complex types and `[FromQuery]`for simple types                    |
| What is the difference between Model Binding and Model Validation?        | Binding = reading data into parameters. Validation = checking if that data follows rules (`[Required]`,`[Range]`) |
