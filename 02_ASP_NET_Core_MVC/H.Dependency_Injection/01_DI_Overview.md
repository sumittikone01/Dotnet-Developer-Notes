
# 01 — Dependency Injection Overview

---

## 🎯 One-Line Definition

> **Dependency Injection is a design pattern where a class receives the objects it needs (its dependencies) from the outside rather than creating them itself — ASP.NET Core has a built-in DI container that creates, manages, and delivers those objects automatically.**

---

## 🔷 The Problem DI Solves

```
A "dependency" is any object a class needs to do its work.

EmployeeController needs EmployeeBAL to get data.
EmployeeBAL        needs EmployeeDAL to run SQL.
EmployeeDAL        needs a connection string to open the database.

These are dependencies.
```

```
WITHOUT DI — every class creates its own dependencies:
──────────────────────────────────────────────────────────────
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController()
    {
        // Controller builds its own dependency chain:
        var dal = new EmployeeDAL("Server=.;Database=EmpDB;...");
        _bal = new EmployeeBAL(dal);
        // Problems ↓ ↓ ↓
    }
}
```

```
PROBLEMS that causes:
──────────────────────────────────────────────────────────────
❌ TIGHT COUPLING
   EmployeeController knows HOW to construct EmployeeBAL.
   Change EmployeeBAL's constructor → edit every class that creates it.

❌ HARD TO TEST
   Can't swap EmployeeDAL with a FakeDAL in unit tests.
   Controller always creates the real one that hits SQL Server.

❌ CONNECTION STRING HARDCODED
   "Server=.;Database=EmpDB;..." lives inside a C# class.
   Change server name → find and recompile every file.

❌ LIFETIME MANAGEMENT ON YOU
   Do you create one EmployeeBAL per request? One for the whole app?
   Two controllers might create two different instances for the same request.
   You have to think about it. You will get it wrong eventually.
```

```
WITH DI — dependencies are declared, not created:
──────────────────────────────────────────────────────────────
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal)
    //                         ↑ "I need a BAL — give me one"
    {
        _bal = bal;
        // ASP.NET Core created it, configured it, injected it.
        // Controller has zero knowledge of HOW it was built.
    }
}
```

---

## 🔷 The Three Moving Parts of DI

```
┌─────────────────────────────────────────────────────────────────┐
│                      DI HAS THREE PARTS                         │
│                                                                 │
│  1. SERVICE           The class or interface you want to use    │
│                       e.g. EmployeeBAL, IEmployeeBAL            │
│                                                                 │
│  2. REGISTRATION      Telling the DI container:                 │
│                       "When someone needs IEmployeeBAL,         │
│                        give them an EmployeeBAL"                │
│                       Done once in Program.cs                   │
│                                                                 │
│  3. INJECTION         Declaring the dependency in a             │
│                       constructor parameter                     │
│                       ASP.NET Core reads it and provides        │
│                       the registered instance automatically      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 DI in ASP.NET Core — The Complete Flow

```
STEP 1: REGISTER in Program.cs (once, at startup)
──────────────────────────────────────────────────────────────
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();
builder.Services.AddScoped<IEmployeeDAL, EmployeeDAL>();
// "Container: when IEmployeeBAL is needed → create EmployeeBAL"
// "Container: when IEmployeeDAL is needed → create EmployeeDAL"


STEP 2: DECLARE in constructors (in every class that needs them)
──────────────────────────────────────────────────────────────
public class EmployeeController : Controller
{
    public EmployeeController(IEmployeeBAL bal) { _bal = bal; }
    //                         ↑ declaration — not creation
}

public class EmployeeBAL : IEmployeeBAL
{
    public EmployeeBAL(IEmployeeDAL dal) { _dal = dal; }
    //                  ↑ declaration — not creation
}

public class EmployeeDAL : IEmployeeDAL
{
    public EmployeeDAL(IConfiguration config)
    //                  ↑ IConfiguration is registered by ASP.NET Core automatically
    {
        _conn = config.GetConnectionString("DefaultConnection");
    }
}


STEP 3: REQUEST ARRIVES — container builds the whole chain
──────────────────────────────────────────────────────────────
GET /Employee/Index arrives
    │
    Container reads EmployeeController constructor:
    "needs IEmployeeBAL"
    │
    Container reads EmployeeBAL constructor:
    "needs IEmployeeDAL"
    │
    Container reads EmployeeDAL constructor:
    "needs IConfiguration"
    │
    IConfiguration: already registered ✅
    Create EmployeeDAL(config)            → dal
    Create EmployeeBAL(dal)               → bal
    Create EmployeeController(bal)        → controller
    │
    EmployeeController.Index() runs
```

---

## 🔷 Without Interface vs With Interface

```csharp
// ── Without interface — concrete type ─────────────────────────────
// Registration:
builder.Services.AddScoped<EmployeeBAL>();

// Injection:
public EmployeeController(EmployeeBAL bal) { _bal = bal; }

// Works. But you're locked to EmployeeBAL.
// Can't swap it in tests without changing the controller.


// ── With interface — the right way ────────────────────────────────
// Interface (the contract — what it CAN DO):
public interface IEmployeeBAL
{
    List<Employee> GetAll();
    Employee       GetById(int id);
    int            Add(Employee emp);
    bool           Update(Employee emp);
    bool           Delete(int id);
}

// Implementation (the HOW):
public class EmployeeBAL : IEmployeeBAL
{
    private readonly IEmployeeDAL _dal;
    public EmployeeBAL(IEmployeeDAL dal) => _dal = dal;

    public List<Employee> GetAll()   => _dal.GetAll();
    public Employee GetById(int id)  => _dal.GetById(id);
    public int Add(Employee emp)     => _dal.Insert(emp);
    public bool Update(Employee emp) => _dal.Update(emp);
    public bool Delete(int id)       => _dal.Delete(id);
}

// Registration:
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();

// Injection:
public EmployeeController(IEmployeeBAL bal) { _bal = bal; }

// NOW you can:
// ✅ Swap EmployeeBAL with FakeEmployeeBAL in tests
// ✅ Swap with CachedEmployeeBAL without touching the controller
// ✅ Controller has zero knowledge of the implementation
```

---

## 🔷 The DI Container — What It Actually Is

```
The DI container is ASP.NET Core's built-in object factory.

You tell it: "IEmployeeBAL → give EmployeeBAL"
It remembers that.
Every time IEmployeeBAL is needed:
  → Creates the EmployeeBAL (and all its dependencies recursively)
  → Hands it to whoever asked
  → Manages its lifetime (scoped/transient/singleton)
  → Disposes it when done

The container in ASP.NET Core is Microsoft.Extensions.DependencyInjection.
It is part of the framework — no NuGet package needed.
It is accessed through: builder.Services (type: IServiceCollection)
```

---

## 🔷 What Happens Without DI — The Snowball

```csharp
// Imagine DepartmentService also needs a Logger and a CacheService:

public class DepartmentService
{
    public DepartmentService(
        DepartmentDAL dal,
        ILogger logger,
        ICacheService cache) { }
}

// WITHOUT DI — whoever creates DepartmentService must also build:
var logger  = new ConsoleLogger(LogLevel.Information);
var cache   = new MemoryCacheService(TimeSpan.FromMinutes(5));
var dal     = new DepartmentDAL("Server=.;Database=EmpDB;...");
var service = new DepartmentService(dal, logger, cache);

// And if DepartmentDAL needs something too... it cascades.
// Every caller must know the full object graph.

// WITH DI — just declare:
public class SomeController : Controller
{
    public SomeController(DepartmentService svc)
    // Container builds the entire graph automatically
    { }
}
```

---

## 🔷 DI Benefits — Concrete, Real

```
BENEFIT 1: LOOSE COUPLING
──────────────────────────────────────────────────────────────
Controller depends on IEmployeeBAL (interface), not EmployeeBAL (class).
You can change the entire EmployeeBAL implementation without touching controllers.

BENEFIT 2: EASY UNIT TESTING
──────────────────────────────────────────────────────────────
// In tests — inject a fake instead of the real BAL:
var fakeBAL = new FakeEmployeeBAL();  // returns hardcoded data, no DB
var controller = new EmployeeController(fakeBAL);

// Test controller logic without a database connection.
// Without DI: impossible — controller always creates the real BAL.

BENEFIT 3: LIFETIME MANAGEMENT
──────────────────────────────────────────────────────────────
You declare: AddScoped<IEmployeeBAL, EmployeeBAL>()
Container guarantees: one instance per request, disposed after.
You write zero cleanup code.

BENEFIT 4: CENTRALIZED CONFIGURATION
──────────────────────────────────────────────────────────────
All wiring lives in ONE place — Program.cs.
To swap EmployeeBAL with a new implementation:
  Change ONE line in Program.cs.
  Zero other files touched.

BENEFIT 5: AUTOMATIC DISPOSAL
──────────────────────────────────────────────────────────────
If EmployeeDAL implements IDisposable (e.g., holds resources):
  Container calls Dispose() automatically when the request ends.
  You never forget to dispose. No resource leaks.
```

---

## 🔷 DI vs Service Locator — The Anti-Pattern

```csharp
// ❌ SERVICE LOCATOR — anti-pattern, avoid this
public class EmployeeController : Controller
{
    public IActionResult Index()
    {
        // Reaching into the container manually from inside code:
        var bal = HttpContext.RequestServices.GetService<IEmployeeBAL>();
        var list = bal.GetAll();
        return View(list);
    }
}

// Problems:
// - Dependencies are hidden (not visible in constructor)
// - Hard to test (you need to set up the whole service provider)
// - Violates "explicit dependencies" principle

// ✅ CONSTRUCTOR INJECTION — always prefer this
public class EmployeeController : Controller
{
    private readonly IEmployeeBAL _bal;

    public EmployeeController(IEmployeeBAL bal)
    {
        _bal = bal;  // ← explicit, visible, testable
    }

    public IActionResult Index()
    {
        return View(_bal.GetAll());
    }
}
```

---

## 🔷 Your Real Project — Full DI Setup

```csharp
// Program.cs — registering your entire service layer
var builder = WebApplication.CreateBuilder(args);

// ── Framework services ─────────────────────────────────────────
builder.Services.AddControllersWithViews();
builder.Services.AddSession();
builder.Services.AddMemoryCache();

// ── Your DAL classes ───────────────────────────────────────────
builder.Services.AddScoped<IEmployeeDAL,    EmployeeDAL>();
builder.Services.AddScoped<IDepartmentDAL,  DepartmentDAL>();
builder.Services.AddScoped<ILeaveDAL,       LeaveDAL>();
builder.Services.AddScoped<IPayrollDAL,     PayrollDAL>();

// ── Your BAL classes ───────────────────────────────────────────
builder.Services.AddScoped<IEmployeeBAL,    EmployeeBAL>();
builder.Services.AddScoped<IDepartmentBAL,  DepartmentBAL>();
builder.Services.AddScoped<ILeaveBAL,       LeaveBAL>();
builder.Services.AddScoped<IPayrollBAL,     PayrollBAL>();

// ── Shared services ────────────────────────────────────────────
builder.Services.AddTransient<IEmailService,   SmtpEmailService>();
builder.Services.AddSingleton<IAppSettings,    AppSettingsService>();

var app = builder.Build();
// ... rest of pipeline
```

```csharp
// EmployeeController.cs — clean, no knowledge of implementations
public class EmployeeController : Controller
{
    private readonly IEmployeeBAL   _empBal;
    private readonly IDepartmentBAL _deptBal;

    public EmployeeController(
        IEmployeeBAL   empBal,
        IDepartmentBAL deptBal)
    {
        _empBal  = empBal;
        _deptBal = deptBal;
    }

    public IActionResult Create()
    {
        ViewBag.DeptList = new SelectList(
            _deptBal.GetAll(), "DeptId", "DeptName");
        return View(new Employee());
    }
}
```

---

## ⭐ Interview Quick-Fire

| Question                                                | Answer                                                                                                                                           |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| What is Dependency Injection?                           | A pattern where a class receives its dependencies from outside rather than creating them — ASP.NET Core's container provides them automatically |
| What is a dependency?                                   | Any object a class needs to do its work —`EmployeeController`depends on `EmployeeBAL`                                                       |
| What are the three parts of DI?                         | Service (the class), Registration (telling the container), Injection (declaring via constructor)                                                 |
| Why use an interface with DI?                           | Allows swapping implementations without changing the consumer — essential for unit testing                                                      |
| Where do you register services in ASP.NET Core?         | `Program.cs`using `builder.Services.Add*<Interface, Implementation>()`                                                                       |
| What is the DI container?                               | The built-in object factory that builds, manages lifetime, and disposes of all registered services                                               |
| What is the benefit of DI over `new`?                 | Loose coupling, testability, centralized configuration, automatic lifetime management                                                            |
| What is the Service Locator anti-pattern?               | Reaching into the container manually from inside code (`HttpContext.RequestServices.GetService<T>()`) — hides dependencies                    |
| Is a third-party library needed for DI in ASP.NET Core? | ❌ No — DI is built into the framework via `Microsoft.Extensions.DependencyInjection`                                                         |
| What happens if you forget to register a service?       | `InvalidOperationException`at runtime: "No service for type 'X' has been registered"                                                           |
