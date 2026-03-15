
# 03 — Controller Responsibilities

---

## 🎯 One-Line Definition

> **The Controller is the traffic director of your MVC app — it receives the HTTP request, decides what data to fetch (by calling BAL/DAL), prepares that data, and chooses what to send back: a View for the browser, or JSON for AJAX/Kendo.**

---

## 🔷 Where the Controller Sits

```
Browser / Kendo / AJAX
        │
        │  GET /Employee/Index
        ▼
┌───────────────────────────────────────────────────────────┐
│              Middleware Pipeline                          │
│  UseRouting → UseAuthentication → UseAuthorization       │
└───────────────────────┬───────────────────────────────────┘
                        │
              ┌─────────▼──────────┐
              │    CONTROLLER      │  ← YOU ARE HERE
              │                    │
              │  Receives request  │
              │  Calls BAL         │
              │  Prepares data     │
              │  Returns response  │
              └─────────┬──────────┘
                        │
              ┌─────────▼──────────┐
              │   BAL → DAL        │
              │   SQL Server       │
              └─────────┬──────────┘
                        │
              ┌─────────▼──────────┐
              │  View (HTML)       │  ← if MVC
              │  OR JSON           │  ← if API
              └────────────────────┘
```

---

## 🔷 What a Controller Is — and What It Is NOT

```
CONTROLLER IS:                         CONTROLLER IS NOT:
──────────────────────────────────     ──────────────────────────────────
✅ Entry point for HTTP requests       ❌ Where SQL queries are written
✅ Reads request data (form, query)    ❌ Where business logic lives
✅ Calls BAL for business rules        ❌ Where validation rules are defined
✅ Passes data to View or returns JSON ❌ Where HTML is built
✅ Returns the correct HTTP response   ❌ Where data is stored

THIN CONTROLLER RULE:
  If your controller action has more than ~15 lines of logic,
  something is wrong. Business rules belong in BAL. SQL belongs in DAL.
  The controller just wires them together.
```

---

## 🔷 The 5 Responsibilities of a Controller — One By One

### Responsibility 1: Receive the HTTP Request and Extract Data

```csharp
// From URL route:   GET /Employee/Details/5
public IActionResult Details(int id)
//                           ↑ id = 5, extracted from URL automatically

// From query string:  GET /Employee/Index?page=2&dept=HR
public IActionResult Index(int page = 1, string dept = null)
//                         ↑ page = 2, dept = "HR" from query string

// From form POST:   POST /Employee/Create  (form body)
[HttpPost]
public IActionResult Create(Employee emp)
//                           ↑ emp object populated from form fields

// From JSON body:   POST /api/employee  (AJAX/Kendo)
[HttpPost]
public IActionResult Create([FromBody] Employee emp)
//                           ↑ emp populated from JSON body
```

---

### Responsibility 2: Validate Input (Basic Only)

```csharp
[HttpPost]
public IActionResult Create(Employee emp)
{
    // Controller checks ModelState (Data Annotations validation result):
    if (!ModelState.IsValid)
    {
        // Re-render the form with validation errors
        return View(emp);
    }

    // ← Business validation (duplicate name, salary range) goes in BAL, NOT here
    _bal.AddEmployee(emp);
    return RedirectToAction("Index");
}
```

```
What validates where:
──────────────────────────────────────────────────────────────
Data Annotations ([Required], [Range]) → checked by ModelState
ModelState.IsValid check               → Controller
Business rules (no duplicate names)    → BAL
SQL constraints (UNIQUE, FK)           → Database / DAL catches exception
```

---

### Responsibility 3: Call BAL (Never Call DAL Directly)

```csharp
// ✅ CORRECT — Controller calls BAL, BAL calls DAL
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal)
    {
        _bal = bal;   // ← injected by DI (registered in Program.cs)
    }

    public IActionResult Index()
    {
        var employees = _bal.GetActiveEmployees();  // ← calls BAL
        return View(employees);
    }
}

// ❌ WRONG — Controller talking directly to DAL
public class EmployeeController : Controller
{
    private readonly EmployeeDAL _dal;   // ← skip BAL? No.

    public IActionResult Index()
    {
        var employees = _dal.GetAll();   // ← bypasses business rules entirely
        return View(employees);          //   what if you have "active only" rule?
    }
}
```

---

### Responsibility 4: Prepare Data for the View

```csharp
public IActionResult Index()
{
    // BAL returns raw List<Employee>
    var employees = _bal.GetActiveEmployees();

    // Controller might:
    //   1. Pass it directly to View
    return View(employees);

    //   2. Pack into a ViewModel for richer views
    var vm = new EmployeeListViewModel
    {
        Employees   = employees,
        Departments = _deptBal.GetAll(),   // for filter dropdown
        TotalCount  = employees.Count,
        CurrentPage = 1
    };
    return View(vm);

    //   3. Store in ViewBag for small pieces of data
    ViewBag.DepartmentList = _deptBal.GetAll();  // for dropdown
    return View(employees);
}
```

---

### Responsibility 5: Return the Correct Response

```csharp
// ── For MVC (returns Views) ────────────────────────────────────────

// Return a View (200 OK + HTML)
return View();                        // Views/Employee/Index.cshtml
return View("Edit", emp);             // Views/Employee/Edit.cshtml with data
return PartialView("_EmpRow", emp);   // renders partial HTML fragment

// Redirect (302 → browser makes a new request)
return RedirectToAction("Index");
return RedirectToAction("Index", "Home");          // different controller
return RedirectToAction("Details", new { id = 5 }); // with route values
return Redirect("https://external.com");           // absolute URL

// Not found / error
return NotFound();       // 404
return BadRequest();     // 400

// ── For API / AJAX (returns JSON) ─────────────────────────────────

return Ok(data);                          // 200 + JSON body
return Ok(new { data = list, total = n }); // 200 + anonymous JSON
return NotFound(new { message = "..." }); // 404 + JSON body
return BadRequest(ModelState);            // 400 + validation errors
return CreatedAtAction(nameof(GetById), new { id = newId }, obj); // 201
return NoContent();                       // 204 (PUT/DELETE success)
return StatusCode(500, new { message }); // custom status code
```

---

## 🔷 MVC Controller vs API Controller — Side by Side

```
┌──────────────────────────────────────────────────────────────────┐
│         MVC Controller                API Controller             │
│   (returns Views — HTML)         (returns JSON — data)           │
├──────────────────────────────────────────────────────────────────┤
│  : Controller                     : ControllerBase               │
│  (has View(), ViewBag etc.)        (no View stuff)               │
├──────────────────────────────────────────────────────────────────┤
│  No [ApiController]               [ApiController] attribute       │
│                                   (auto validation, auto binding) │
├──────────────────────────────────────────────────────────────────┤
│  [Route] optional                 [Route("api/employee")]        │
│  Conventional routing works       [Route] required               │
├──────────────────────────────────────────────────────────────────┤
│  return View(data)                return Ok(data)                │
│  return RedirectToAction(...)     return CreatedAtAction(...)    │
│  return PartialView(...)          return NotFound(...)           │
├──────────────────────────────────────────────────────────────────┤
│  Called by: browser URL           Called by: AJAX, Kendo, fetch  │
└──────────────────────────────────────────────────────────────────┘
```

```csharp
// MVC Controller — full example
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    // GET /Employee/Index
    public IActionResult Index()
    {
        var list = _bal.GetActiveEmployees();
        return View(list);   // → Views/Employee/Index.cshtml
    }

    // GET /Employee/Create  (show empty form)
    public IActionResult Create()
    {
        return View();
    }

    // POST /Employee/Create  (form submitted)
    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);
        _bal.AddEmployee(emp);
        return RedirectToAction("Index");
    }

    // GET /Employee/Edit/5
    public IActionResult Edit(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return View(emp);
    }
}

// API Controller — full example (for AJAX / Kendo)
[ApiController]
[Route("api/employee")]
public class EmployeeApiController : ControllerBase
{
    private readonly EmployeeBAL _bal;
    public EmployeeApiController(EmployeeBAL bal) => _bal = bal;

    // GET /api/employee?skip=0&take=10   ← Kendo Grid calls this
    [HttpGet]
    public IActionResult GetAll(int skip = 0, int take = 10)
    {
        int total = 0;
        var data  = _bal.GetPaged(skip, take, out total);
        return Ok(new { data, total });  // Kendo DataSource expects this shape
    }

    // GET /api/employee/5
    [HttpGet("{id:int}")]
    public IActionResult GetById(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound(new { message = $"Employee {id} not found" });
        return Ok(emp);
    }

    // POST /api/employee   ← Kendo Grid Add row calls this
    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        int newId = _bal.AddEmployee(emp);
        emp.EmpId = newId;
        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    }

    // PUT /api/employee/5  ← Kendo Grid Edit calls this
    [HttpPut("{id:int}")]
    public IActionResult Update(int id, [FromBody] Employee emp)
    {
        if (id != emp.EmpId)
            return BadRequest(new { message = "ID mismatch" });
        bool updated = _bal.UpdateEmployee(emp);
        if (!updated) return NotFound();
        return NoContent();   // 204 — Kendo expects this for successful PUT
    }

    // DELETE /api/employee/5  ← Kendo Grid Delete calls this
    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id)
    {
        bool deleted = _bal.DeleteEmployee(id);
        if (!deleted) return NotFound();
        return NoContent();
    }
}
```

---

## 🔷 Dependency Injection Into Controllers

```csharp
// Program.cs — register your classes ONCE
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();
builder.Services.AddScoped<DepartmentBAL>();

// Controller — inject via constructor
// ASP.NET Core creates the controller and automatically passes
// the registered instances — you never call "new EmployeeBAL()"
public class EmployeeController : Controller
{
    private readonly EmployeeBAL    _empBal;
    private readonly DepartmentBAL  _deptBal;

    // ↓ ASP.NET Core injects both — you just declare what you need
    public EmployeeController(EmployeeBAL empBal, DepartmentBAL deptBal)
    {
        _empBal  = empBal;
        _deptBal = deptBal;
    }
}
```

---

## 🔷 Passing Data from Controller to View — Quick Map

```csharp
// OPTION 1: Strongly Typed Model (BEST — IntelliSense, type safety)
return View(employeeList);
// → In view: @model List<Employee>  then  @Model.Count

// OPTION 2: ViewBag (dynamic — good for small, secondary data)
ViewBag.DepartmentList = _deptBal.GetAll();
ViewBag.CurrentUser    = "John";
return View(employeeList);
// → In view: @ViewBag.DepartmentList  (no IntelliSense)

// OPTION 3: ViewData (like ViewBag but dictionary syntax)
ViewData["Title"] = "Employee List";
return View(employeeList);
// → In view: @ViewData["Title"]

// OPTION 4: TempData (survives ONE redirect — for success/error messages)
TempData["SuccessMsg"] = "Employee created successfully!";
return RedirectToAction("Index");
// → In Index view: @TempData["SuccessMsg"]
```

---

## 🔷 What a Clean Controller Looks Like

```csharp
// ✅ CLEAN — thin, reads like English, no SQL, no business logic
[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
        return View(emp);

    bool created = _bal.AddEmployee(emp);

    if (!created)
    {
        ModelState.AddModelError("", "Employee with this email already exists.");
        return View(emp);
    }

    TempData["Success"] = "Employee added successfully!";
    return RedirectToAction("Index");
}

// ❌ FAT CONTROLLER — SQL in controller, business rules in controller
[HttpPost]
public IActionResult Create(Employee emp)
{
    // SQL directly in controller — WRONG
    using var con = new SqlConnection(_conn);
    using var cmd = new SqlCommand("SELECT COUNT(*) FROM Employees WHERE Email = @Email", con);
    cmd.Parameters.AddWithValue("@Email", emp.Email);
    con.Open();
    int count = (int)cmd.ExecuteScalar();
    if (count > 0)  // business logic here — WRONG
    {
        ModelState.AddModelError("", "Email exists");
        return View(emp);
    }
    using var insertCmd = new SqlCommand("INSERT INTO Employees...", con);
    // ... 30 more lines
}
```

---

## ⭐ Interview Quick-Fire

| Question                                                             | Answer                                                                                              |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| What is the responsibility of a Controller?                          | Receive HTTP request, validate input, call BAL, prepare data, return response (View or JSON)        |
| Should a Controller contain SQL queries?                             | ❌ No — SQL belongs in DAL                                                                         |
| Should a Controller contain business rules?                          | ❌ No — business rules belong in BAL                                                               |
| What is a "fat controller"?                                          | A controller that contains business logic and/or SQL — a code smell                                |
| What is the difference between `Controller`and `ControllerBase`? | `Controller`= MVC (has `View()`,`ViewBag`).`ControllerBase`= API only (JSON responses)      |
| What does `[ApiController]`do?                                     | Auto validates model, auto infers `[FromBody]`, returns structured error responses                |
| How does a controller get its BAL dependency?                        | Constructor injection — registered in `Program.cs`, provided by ASP.NET Core DI                  |
| What should `return View()`look for?                               | `Views/{ControllerName}/{ActionName}.cshtml`by default                                            |
| What HTTP status does `return NoContent()`give?                    | 204 — used for successful PUT and DELETE that return no body                                       |
| What is `RedirectToAction`used for?                                | Post-Redirect-Get pattern — after a POST succeeds, redirect to a GET to prevent form re-submission |
