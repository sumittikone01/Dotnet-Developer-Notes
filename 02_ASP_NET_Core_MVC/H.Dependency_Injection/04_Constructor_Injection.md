
# 04 — Constructor Injection

---

## 🎯 One-Line Definition

> **Constructor Injection is the primary way to use DI in ASP.NET Core — you declare every dependency your class needs as a constructor parameter, and the DI container reads those parameters, resolves each registered service, and passes them in automatically every time the class is created.**

---

## 🔷 How Constructor Injection Works

```
REGISTRATION (Program.cs — once at startup):
  builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();
  builder.Services.AddScoped<IDepartmentBAL, DepartmentBAL>();

                            │
                            ▼

CLASS DECLARATION (declare what you need — don't create it):
  public class EmployeeController : Controller
  {
      public EmployeeController(IEmployeeBAL bal, IDepartmentBAL deptBal)
      {                          ↑                 ↑
          _bal     = bal;      // need these     need these
          _deptBal = deptBal;  // ← container provides both
      }
  }

                            │
                            ▼

REQUEST ARRIVES — container reads the constructor:
  "EmployeeController needs IEmployeeBAL and IDepartmentBAL"
  "IEmployeeBAL is registered as EmployeeBAL → create it"
  "IDepartmentBAL is registered as DepartmentBAL → create it"
  "Now create EmployeeController(empBal, deptBal)"
  → action method runs
```

---

## 🔷 Constructor Injection in Controllers

```csharp
// ── MVC Controller ─────────────────────────────────────────────────
public class EmployeeController : Controller
{
    // Private readonly fields — set once in constructor, never changed
    private readonly IEmployeeBAL   _empBal;
    private readonly IDepartmentBAL _deptBal;
    private readonly ILogger<EmployeeController> _logger;

    // Constructor — ASP.NET Core calls this and injects everything
    public EmployeeController(
        IEmployeeBAL   empBal,
        IDepartmentBAL deptBal,
        ILogger<EmployeeController> logger)
    {
        _empBal  = empBal;
        _deptBal = deptBal;
        _logger  = logger;
    }

    public IActionResult Index()
    {
        _logger.LogInformation("Loading employee list");

        ViewBag.DeptList = new SelectList(
            _deptBal.GetAll(), "DeptId", "DeptName");

        var employees = _empBal.GetActiveEmployees();
        return View(employees);
    }

    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid)
        {
            ViewBag.DeptList = new SelectList(
                _deptBal.GetAll(), "DeptId", "DeptName");
            return View(emp);
        }

        _empBal.Add(emp);
        _logger.LogInformation("Employee {Name} created", emp.EmpName);
        TempData["Success"] = "Employee created successfully!";
        return RedirectToAction("Index");
    }
}

// ── API Controller ──────────────────────────────────────────────────
[ApiController]
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    private readonly IEmployeeBAL _bal;
    private readonly ILogger<EmployeeApiController> _logger;

    public EmployeeApiController(
        IEmployeeBAL bal,
        ILogger<EmployeeApiController> logger)
    {
        _bal    = bal;
        _logger = logger;
    }

    [HttpGet]
    public IActionResult GetAll(int skip = 0, int take = 10)
    {
        _logger.LogDebug("GetAll called — skip:{Skip}, take:{Take}", skip, take);
        var data  = _bal.GetPaged(skip, take, out int total);
        return Ok(new { data, total });
    }

    [HttpGet("{id:int:min(1)}")]
    public IActionResult GetById(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null)
            return NotFound(new { message = $"Employee {id} not found" });
        return Ok(emp);
    }

    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        int newId = _bal.Add(emp);
        emp.EmpId = newId;
        _logger.LogInformation("Employee created with ID {Id}", newId);
        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    }
}
```

---

## 🔷 Constructor Injection in BAL and DAL

```csharp
// The injection chain — each layer injects what it needs:
//
// EmployeeController  ← injects IEmployeeBAL
//     EmployeeBAL     ← injects IEmployeeDAL
//         EmployeeDAL ← injects IConfiguration

// ── DAL — injects IConfiguration ──────────────────────────────────
public class EmployeeDAL : IEmployeeDAL
{
    private readonly string _connectionString;

    public EmployeeDAL(IConfiguration configuration)
    //                  ↑ IConfiguration registered by ASP.NET Core automatically
    {
        _connectionString =
            configuration.GetConnectionString("DefaultConnection");
        // Reads from appsettings.json — no hardcoding
    }

    public List<Employee> GetAll()
    {
        var list = new List<Employee>();
        using var con = new SqlConnection(_connectionString);
        using var cmd = new SqlCommand("SELECT * FROM Employees WHERE IsActive = 1", con);
        con.Open();
        using var rdr = cmd.ExecuteReader();
        while (rdr.Read())
        {
            list.Add(new Employee
            {
                EmpId   = (int)rdr["EmpId"],
                EmpName = rdr["EmpName"].ToString(),
                Salary  = (decimal)rdr["Salary"]
            });
        }
        return list;
    }

    public Employee GetById(int id)
    {
        using var con = new SqlConnection(_connectionString);
        using var cmd = new SqlCommand(
            "SELECT * FROM Employees WHERE EmpId = @Id", con);
        cmd.Parameters.AddWithValue("@Id", id);
        con.Open();
        using var rdr = cmd.ExecuteReader();
        if (!rdr.Read()) return null;
        return new Employee
        {
            EmpId   = (int)rdr["EmpId"],
            EmpName = rdr["EmpName"].ToString(),
            Salary  = (decimal)rdr["Salary"]
        };
    }
    // ... Insert, Update, Delete
}

// ── BAL — injects IEmployeeDAL ─────────────────────────────────────
public class EmployeeBAL : IEmployeeBAL
{
    private readonly IEmployeeDAL _dal;
    private readonly ILogger<EmployeeBAL> _logger;

    public EmployeeBAL(IEmployeeDAL dal, ILogger<EmployeeBAL> logger)
    {
        _dal    = dal;
        _logger = logger;
    }

    public List<Employee> GetActiveEmployees()
    {
        _logger.LogDebug("Fetching active employees");
        return _dal.GetAll();
    }

    public Employee GetById(int id)
    {
        if (id <= 0) throw new ArgumentException("ID must be positive", nameof(id));
        return _dal.GetById(id);
    }

    public int Add(Employee emp)
    {
        // Business rule: no duplicate email
        // Business rule: salary must not be negative
        if (emp.Salary < 0)
            throw new BusinessRuleException("Salary cannot be negative.");

        _logger.LogInformation("Adding employee: {Name}", emp.EmpName);
        return _dal.Insert(emp);
    }
}

// ── Controller — injects IEmployeeBAL ─────────────────────────────
// (shown above — the chain is complete)
```

---

## 🔷 The readonly Pattern — Why It Matters

```csharp
// ✅ CORRECT — private readonly field
public class EmployeeController : Controller
{
    private readonly IEmployeeBAL _bal;

    public EmployeeController(IEmployeeBAL bal)
    {
        _bal = bal;   // set once in constructor
        // readonly = can NEVER be reassigned after this
        // _bal = somethingElse;  ← compile error
    }
}

// Benefits of readonly:
// ✅ Guarantees the dependency never changes during the object's life
// ✅ Makes the class immutable with respect to its dependencies
// ✅ Thread-safe reads (Singleton case — readonly after construction)
// ✅ Communicates intent: "this is set once and never changes"

// ❌ WRONG — public field, settable, mutable
public class EmployeeController : Controller
{
    public IEmployeeBAL Bal;  // ← public, mutable, no protection
    // Anyone can replace it: controller.Bal = null;
}
```

---

## 🔷 Injecting Multiple Dependencies

```csharp
// Controllers often need multiple services — all injected via constructor
public class ReportController : Controller
{
    private readonly IEmployeeBAL   _empBal;
    private readonly IDepartmentBAL _deptBal;
    private readonly IPayrollBAL    _payBal;
    private readonly IReportBuilder _reportBuilder;
    private readonly IEmailService  _emailService;
    private readonly ILogger<ReportController> _logger;

    public ReportController(
        IEmployeeBAL   empBal,
        IDepartmentBAL deptBal,
        IPayrollBAL    payBal,
        IReportBuilder reportBuilder,
        IEmailService  emailService,
        ILogger<ReportController> logger)
    {
        _empBal        = empBal;
        _deptBal       = deptBal;
        _payBal        = payBal;
        _reportBuilder = reportBuilder;
        _emailService  = emailService;
        _logger        = logger;
    }

    public async Task<IActionResult> GenerateMonthlyReport(int year, int month)
    {
        _logger.LogInformation("Generating report {Y}/{M}", year, month);

        var employees   = _empBal.GetAll();
        var departments = _deptBal.GetAll();
        var payroll     = _payBal.GetForMonth(year, month);

        var reportBytes = _reportBuilder.Build(employees, departments, payroll);

        await _emailService.SendAsync(
            to:      "hr@company.com",
            subject: $"Monthly Report {month}/{year}",
            body:    "Please find attached the monthly report.",
            attachment: reportBytes);

        return File(reportBytes, "application/xlsx", $"Report_{year}_{month}.xlsx");
    }
}
```

---

## 🔷 ILogger — The Automatically Registered Service

```csharp
// ILogger<T> is registered by ASP.NET Core automatically — no AddScoped needed
// Just declare it and it's injected

public class EmployeeBAL : IEmployeeBAL
{
    private readonly ILogger<EmployeeBAL> _logger;
    private readonly IEmployeeDAL _dal;

    public EmployeeBAL(ILogger<EmployeeBAL> logger, IEmployeeDAL dal)
    {
        _logger = logger;
        _dal    = dal;
    }

    public List<Employee> GetAll()
    {
        _logger.LogInformation("Fetching all employees from DAL");
        var result = _dal.GetAll();
        _logger.LogDebug("Fetched {Count} employees", result.Count);
        return result;
    }
}

// Log levels (least to most severe):
// _logger.LogTrace    ("very detailed");
// _logger.LogDebug    ("debug info");
// _logger.LogInformation("normal operation");
// _logger.LogWarning  ("something unexpected but recoverable");
// _logger.LogError    ("something failed", exception);
// _logger.LogCritical ("system is broken");
```

---

## 🔷 IConfiguration — The Other Automatically Registered Service

```csharp
// IConfiguration also registered automatically — reads from appsettings.json

public class AppSettingsService : IAppSettings
{
    public string AppName        { get; }
    public int    PageSize       { get; }
    public string SupportEmail   { get; }

    public AppSettingsService(IConfiguration configuration)
    {
        AppName      = configuration["AppSettings:AppName"]   ?? "Employee Portal";
        PageSize     = configuration.GetValue<int>("AppSettings:PageSize", 10);
        SupportEmail = configuration["AppSettings:SupportEmail"] ?? "support@company.com";
    }
}

// DAL reads connection string:
public class EmployeeDAL : IEmployeeDAL
{
    private readonly string _conn;

    public EmployeeDAL(IConfiguration config)
    {
        _conn = config.GetConnectionString("DefaultConnection")
            ?? throw new InvalidOperationException(
                   "DefaultConnection string not found in appsettings.json");
    }
}
```

---

## 🔷 Constructor Injection vs Other Patterns

```
┌──────────────────────────────────────────────────────────────────┐
│  Pattern            │  How it works        │  Should you use it? │
├──────────────────────────────────────────────────────────────────┤
│  Constructor        │  Declare in ctor     │  ✅ YES — always    │
│  Injection          │  container injects   │  preferred          │
├──────────────────────────────────────────────────────────────────┤
│  Property Injection │  public settable     │  ❌ Rarely          │
│                     │  property on class   │  optional deps only │
│                     │  ASP.NET Core DI     │  ASP.NET Core DI    │
│                     │  doesn't support     │  doesn't support    │
├──────────────────────────────────────────────────────────────────┤
│  Method Injection   │  [FromServices] on   │  ✅ OK for one-     │
│                     │  action parameter    │  action-only deps   │
├──────────────────────────────────────────────────────────────────┤
│  Service Locator    │  manually call       │  ❌ Anti-pattern    │
│  (anti-pattern)     │  GetService<T>()     │  hides dependencies │
│                     │  from inside code    │                     │
└──────────────────────────────────────────────────────────────────┘
```

```csharp
// Method Injection — [FromServices] for action-only deps:
[HttpGet("export")]
public IActionResult Export(
    [FromServices] IReportBuilder reportBuilder)
//   ↑ Only this action uses IReportBuilder — don't pollute the constructor
{
    var file = reportBuilder.Build(_bal.GetAll());
    return File(file, "application/xlsx", "export.xlsx");
}

// Service Locator — the anti-pattern (don't do this):
public IActionResult Index()
{
    // ❌ Reaching into container manually
    var bal = HttpContext.RequestServices.GetService<IEmployeeBAL>();
    // Hidden dependency — not visible in constructor
    // Hard to test — must mock HttpContext and ServiceProvider
}
```

---

## 🔷 Unit Testing With Constructor Injection

```csharp
// Constructor injection makes unit testing trivial
// You can inject a fake or mock instead of the real implementation

// Manual fake:
public class FakeEmployeeBAL : IEmployeeBAL
{
    public List<Employee> GetActiveEmployees() => new()
    {
        new Employee { EmpId = 1, EmpName = "Alice", Salary = 50000 },
        new Employee { EmpId = 2, EmpName = "Bob",   Salary = 60000 }
    };

    public Employee GetById(int id) =>
        new Employee { EmpId = id, EmpName = "Test Employee" };

    public int Add(Employee emp) => 99;
}

// Test — no database, no HTTP context:
[Fact]
public void Index_Returns_View_With_Employees()
{
    // Arrange
    var fakeBAL     = new FakeEmployeeBAL();
    var fakeDeptBAL = new FakeDepartmentBAL();
    var fakeLogger  = new NullLogger<EmployeeController>();

    var controller  = new EmployeeController(fakeBAL, fakeDeptBAL, fakeLogger);

    // Act
    var result = controller.Index() as ViewResult;

    // Assert
    Assert.NotNull(result);
    var employees = result.Model as List<Employee>;
    Assert.Equal(2, employees.Count);
}

// With Moq (auto-generated fake):
[Fact]
public void GetById_Returns_NotFound_When_Employee_Missing()
{
    var mockBAL = new Mock<IEmployeeBAL>();
    mockBAL.Setup(b => b.GetById(99)).Returns((Employee)null);

    var controller = new EmployeeApiController(
        mockBAL.Object,
        new NullLogger<EmployeeApiController>());

    var result = controller.GetById(99);

    Assert.IsType<NotFoundObjectResult>(result);
}
// Without DI + interfaces: impossible — controller always hits the real DB
```

---

## 🔷 Complete Injection Chain — Your Project

```
REQUEST: GET /Employee/Index
         │
         DI Container builds:
         │
         ├── IConfiguration        (auto-registered, reads appsettings.json)
         │        │
         ├── IEmployeeDAL ──────── EmployeeDAL(IConfiguration)
         │        │                  ↑ gets connection string
         │        │
         ├── IEmployeeBAL ──────── EmployeeBAL(IEmployeeDAL, ILogger<EmployeeBAL>)
         │        │                  ↑ business logic layer
         │        │
         ├── IDepartmentBAL ─────  DepartmentBAL(IDepartmentDAL, ILogger<DeptBAL>)
         │        │
         ├── ILogger<EmployeeController> (auto-registered)
         │        │
         └── EmployeeController(IEmployeeBAL, IDepartmentBAL, ILogger<...>)
                  │
                  EmployeeController.Index() runs
                  _empBal.GetActiveEmployees()
                  _deptBal.GetAll()
                  return View(employees)
```

---

## ⭐ Interview Quick-Fire

| Question                                                                     | Answer                                                                                                                                                                 |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What is Constructor Injection?                                               | Declaring dependencies as constructor parameters — the DI container reads them and provides the registered instances automatically                                    |
| Why use `private readonly`for injected fields?                             | Guarantees the dependency is set once in the constructor and never replaced — signals intent and prevents accidental reassignment                                     |
| Is `ILogger<T>`registered manually?                                        | ❌ No — registered automatically by ASP.NET Core, just declare it in the constructor                                                                                  |
| Is `IConfiguration`registered manually?                                    | ❌ No — registered automatically, reads from `appsettings.json`                                                                                                     |
| What is the difference between Constructor Injection and `[FromServices]`? | Constructor injection adds the dependency for all actions.`[FromServices]`resolves it only for that specific action — useful for one-action-only dependencies       |
| Why does Constructor Injection make testing easy?                            | You pass a fake/mock through the constructor — no real database, no HTTP context needed                                                                               |
| What is the Service Locator anti-pattern?                                    | Calling `HttpContext.RequestServices.GetService<T>()`inside code instead of declaring the dependency in the constructor — hides dependencies and breaks testability |
| Can ASP.NET Core DI do Property Injection?                                   | ❌ No — the built-in container only supports Constructor Injection and `[FromServices]`on action parameters                                                         |
| What happens if a constructor parameter is not registered in the container?  | `InvalidOperationException`at request time: "No service for type 'X' has been registered"                                                                            |
| What pattern do you use to avoid too many constructor parameters?            | Group related services into a Facade class or use extension methods to register them together — if a constructor has 7+ parameters, consider splitting the class      |
