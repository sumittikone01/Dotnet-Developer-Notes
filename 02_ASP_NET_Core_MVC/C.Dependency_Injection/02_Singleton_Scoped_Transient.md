# 02 — Singleton, Scoped, and Transient

## 📌 What is it?

When you register a service with the DI container, you must choose its **lifetime** — how long a single instance of that service lives before a new one is created. ASP.NET Core has three lifetimes: **Transient**, **Scoped**, and **Singleton**.

## 🤔 Why do we need it?

Different services have different needs. A stateless helper class can be reused freely (Singleton). A service tied to a single HTTP request (like one using `DbContext`) needs a fresh instance per request (Scoped). Getting this wrong causes subtle, hard-to-debug bugs — especially around shared/leaked state.

## 📊 The three lifetimes compared

| Lifetime            | New instance created...                                           | Typical use case                                                      |
| ------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Transient** | Every time it's requested/injected                                | Lightweight, stateless services (e.g., a simple calculator/formatter) |
| **Scoped**    | Once per HTTP request (shared within that request)                | `DbContext`, anything tied to "this request's" data                 |
| **Singleton** | Once for the entire application lifetime (shared by all requests) | Configuration objects, caching services, stateless utility services   |

## 💻 Code example — registration syntax

```csharp
builder.Services.AddTransient<IEmailFormatter, EmailFormatter>();
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddSingleton<IAppCache, MemoryAppCache>();
```

## 🖼 Visualizing lifetimes across two simultaneous requests

```
Request A ──┐                          Request B ──┐
            │                                       │
   Transient: new instance      Transient: new instance
   (fresh, every injection)     (fresh, every injection)
            │                                       │
   Scoped: ONE instance         Scoped: a DIFFERENT instance
   shared within Request A      shared within Request B
            │                                       │
   Singleton: ────────────── SAME instance for BOTH ──────────────
              (created once at app startup, reused everywhere)
```

## ⚙️ Practical example — why lifetime choice matters

```csharp
public class RequestLogger : IRequestLogger
{
    private readonly List<string> _logs = new();
    public void Log(string message) => _logs.Add(message);
    public IEnumerable<string> GetLogs() => _logs;
}
```

- Registered as **Transient** → every class that injects `IRequestLogger` gets its own empty `_logs` list — logs never accumulate together, defeating the purpose.
- Registered as **Scoped** → all classes within the *same request* share one `_logs` list — perfect for collecting logs across a single request's lifecycle.
- Registered as **Singleton** → ALL requests across the app's entire lifetime share the SAME `_logs` list — memory grows forever, and logs from different users mix together (dangerous!).

## 🚨 The "Captive Dependency" problem (a classic DI bug)

**Never inject a Scoped or Transient service into a Singleton.** The Singleton is created once and holds onto that reference forever — effectively turning the Scoped/Transient service into a de facto Singleton too, causing stale or leaked data across requests.

```csharp
// ❌ DANGEROUS
public class CacheService  // registered as Singleton
{
    private readonly AppDbContext _context; // Scoped — WRONG to inject here!

    public CacheService(AppDbContext context) => _context = context;
    // _context becomes "captured" — same DbContext instance reused forever,
    // even though DbContext is meant to be short-lived per request
}
```

ASP.NET Core will actually **throw an exception at runtime** (`Cannot consume scoped service from singleton`) if this is detected under default validation settings — a helpful safety net.

## 📊 Lifetime compatibility matrix

| Injecting into ↓ / Injected service → | Transient                                      | Scoped                              | Singleton |
| --------------------------------------- | ---------------------------------------------- | ----------------------------------- | --------- |
| **Transient**                     | ✅ OK                                          | ✅ OK                               | ✅ OK     |
| **Scoped**                        | ✅ OK                                          | ✅ OK                               | ✅ OK     |
| **Singleton**                     | ⚠️ OK but wasteful (loses transient benefit) | ❌ Captive dependency — will throw | ✅ OK     |

## 🚨 Common mistakes

- Registering `DbContext`-dependent services as Singleton — `DbContext` is explicitly designed to be Scoped (short-lived, per-request) because it's not thread-safe for concurrent use across requests.
- Assuming Transient means "no memory cost" — creating too many Transient instances of expensive-to-construct objects can hurt performance; use Scoped or Singleton for expensive setup.
- Storing per-request mutable state in a Singleton — causes data leaking between unrelated users' requests (a serious bug, sometimes even a security issue).

## 💡 Best practices

- Default to **Scoped** for most business/application services — it's the safest middle ground for typical web request-response work.
- Use **Singleton** only for genuinely stateless or safely-shared services (configuration, caching infrastructure, logging infrastructure).
- Use **Transient** for lightweight, cheap-to-construct, stateless helper classes.
- Always register `DbContext` as Scoped (`AddDbContext<T>()` does this automatically).

## 🎤 Interview questions

1. Explain the difference between Transient, Scoped, and Singleton lifetimes with an example of when to use each.
2. What is a "captive dependency," and why is injecting a Scoped service into a Singleton dangerous?
3. Why must `DbContext` be registered as Scoped rather than Singleton?
4. If you register a service as Transient but it holds an internal list that grows with every method call, what problem could arise?

## 📝 30-second revision cheat sheet

- **Transient** = new instance every injection. **Scoped** = one instance per HTTP request. **Singleton** = one instance for the app's whole lifetime.
- Never inject Scoped/Transient into Singleton → "captive dependency" bug (framework often throws at runtime).
- `DbContext` must always be Scoped.
- Default choice for most services: **Scoped**

# 02 — Singleton, Scoped, Transient

---

## 🎯 One-Line Definition

> **Service Lifetime controls how long an instance created by the DI container lives and how many instances are created — Singleton = one instance forever, Scoped = one per HTTP request, Transient = new one every time it's asked for.**

---

## 🔷 The Three Lifetimes — Visual

```
APPLICATION STARTS
│
│  SINGLETON created here (once, on first use)
│  ┌──────────────────────────────────────────────────────────┐
│  │  ConfigService instance                                   │  ← lives here
│  └──────────────────────────────────────────────────────────┘
│
│  REQUEST 1 arrives ─────────────────────────────────────────┐
│  │  SCOPED created (new for this request)                    │
│  │  ┌────────────────────────────────────┐                   │
│  │  │  EmployeeBAL instance             │  ← lives here     │
│  │  └────────────────────────────────────┘                   │
│  │                                                           │
│  │  TRANSIENT #1 created (new on first inject)              │
│  │  ┌──────────────┐                                        │
│  │  │ EmailService │  ← used, then gone                     │
│  │  └──────────────┘                                        │
│  │  TRANSIENT #2 created (new on second inject)             │
│  │  ┌──────────────┐                                        │
│  │  │ EmailService │  ← different instance                  │
│  │  └──────────────┘                                        │
│  │                                                          │
│  │  REQUEST 1 ends → Scoped disposed, Transients gone       │
│  └──────────────────────────────────────────────────────────┘
│
│  REQUEST 2 arrives ─────────────────────────────────────────┐
│  │  SCOPED created (brand new for this request)              │
│  │  ┌────────────────────────────────────┐                   │
│  │  │  EmployeeBAL instance  (NEW)       │                   │
│  │  └────────────────────────────────────┘                   │
│  │                                                           │
│  │  SINGLETON — SAME instance as Request 1                  │
│  │  ┌──────────────────────────────────────────────────────┐│
│  │  │  ConfigService instance (still alive)                ││
│  │  └──────────────────────────────────────────────────────┘│
│  └──────────────────────────────────────────────────────────┘
│
APPLICATION ENDS → Singleton disposed
```

---

## 🔷 Singleton

### What It Is

```csharp
builder.Services.AddSingleton<IAppConfig, AppConfig>();
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

```
SINGLETON:
  Created: once, on first request (lazy) OR at startup (eager)
  Shared:  across ALL requests, ALL users, entire app lifetime
  Disposed: when the application shuts down
  Count:   ONE instance total — ever
```

### When to Use

```
✅ GOOD for Singleton:
  Configuration readers    → read appsettings once, share everywhere
  In-memory cache          → shared state intentional
  HttpClient factory       → expensive to create, meant to be reused
  Counters / accumulators  → need shared state across requests
  App-level constants      → computed once, read many times

❌ BAD for Singleton:
  Anything that touches the database per-user
  Anything that holds per-user state
  Anything that holds per-request state
  Classes with non-thread-safe code
    → Singleton shared across threads simultaneously
    → Thread-safety is YOUR responsibility
```

### The Thread-Safety Requirement

```csharp
// ❌ DANGEROUS Singleton — not thread-safe
public class CounterService
{
    private int _count = 0;  // shared across ALL requests simultaneously

    public int Increment()
    {
        return _count++;   // READ then WRITE — race condition!
        // Request 1 reads 5, Request 2 reads 5, both write 6
        // Expected: 7. Got: 6. Data corruption.
    }
}

// ✅ SAFE Singleton — thread-safe with Interlocked
public class CounterService
{
    private int _count = 0;

    public int Increment()
    {
        return Interlocked.Increment(ref _count);  // atomic — thread safe
    }
}

// ✅ SAFE Singleton — read-only after initialization
public class AppConfig
{
    public string ConnectionString { get; }  // set once in constructor
    public int    PageSize         { get; }  // then read-only forever

    public AppConfig(IConfiguration config)
    {
        ConnectionString = config.GetConnectionString("Default");
        PageSize         = config.GetValue<int>("AppSettings:PageSize");
    }
    // Pure reads → always thread-safe
}
```

### Proof — Same Instance

```csharp
public class SingletonDemo
{
    public Guid Id { get; } = Guid.NewGuid();  // unique per instance
}

builder.Services.AddSingleton<SingletonDemo>();

// Request 1: controller logs Id = "3fa85f64-..."
// Request 2: controller logs Id = "3fa85f64-..."  ← SAME
// Request 3: controller logs Id = "3fa85f64-..."  ← SAME
// Same GUID every time → same instance
```

---

## 🔷 Scoped

### What It Is

```csharp
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();
builder.Services.AddScoped<IEmployeeDAL, EmployeeDAL>();
```

```
SCOPED:
  Created: once per HTTP request (at the start of the request)
  Shared:  across ALL injections WITHIN the same request
  Disposed: when the HTTP request ends
  Count:   one per request → N requests = N instances (not concurrent)
```

### Why Scoped Is Right for DAL/BAL

```csharp
// Imagine one request calls two services that both need EmployeeBAL:
public class ReportController : Controller
{
    private readonly IEmployeeBAL  _empBal;
    private readonly IDepartmentBAL _deptBal;

    // Both injected → both resolved from the DI container for THIS request
}

public class DepartmentBAL : IDepartmentBAL
{
    private readonly IEmployeeBAL _empBal;  // also needs EmployeeBAL

    // If Scoped: DepartmentBAL gets the SAME EmployeeBAL instance
    //            that ReportController received.
    //            One instance shared for the whole request. ✅
    //            Consistent state within the request. ✅

    // If Transient: DepartmentBAL gets a DIFFERENT EmployeeBAL instance.
    //               Two separate instances doing the same work.
    //               If EmployeeBAL held a transaction or cache → inconsistent.
}
```

### When to Use

```
✅ GOOD for Scoped:
  Database repositories (DAL)       → one connection context per request
  Business logic classes (BAL)      → one per request is sufficient
  Unit of Work pattern              → one transaction scope per request
  User-specific context             → user resolved once, shared in request
  EF Core DbContext                 → must be scoped (it's stateful per request)

❌ BAD for Scoped:
  Injecting into a Singleton
    → CAPTURED DEPENDENCY problem ← important gotcha
    → Singleton lives forever; Scoped lives one request
    → Scoped instance captured inside Singleton outlives its request
    → Use IServiceScopeFactory if you really need this
```

### Proof — Same Instance Within a Request

```csharp
public class ScopedDemo
{
    public Guid Id { get; } = Guid.NewGuid();
}

builder.Services.AddScoped<ScopedDemo>();

// Request 1: ControllerA gets Id = "aaaa", ControllerB in same req gets Id = "aaaa"
// Request 2: ControllerA gets Id = "bbbb", ControllerB in same req gets Id = "bbbb"
// Same within a request. Different across requests.
```

---

## 🔷 Transient

### What It Is

```csharp
builder.Services.AddTransient<IEmailService, SmtpEmailService>();
builder.Services.AddTransient<IReportBuilder, PdfReportBuilder>();
```

```
TRANSIENT:
  Created: every single time it is requested from the container
  Shared:  never — each injection gets its own fresh instance
  Disposed: immediately after use (if IDisposable)
  Count:   N injections = N instances (even in same request)
```

### When to Use

```
✅ GOOD for Transient:
  Stateless operations          → email sender, PDF builder, validator
  Lightweight services          → cheap to create, no shared state
  Helper utilities              → formatters, calculators, mappers
  Services that MUST be fresh   → random number generators, timestamps

❌ BAD for Transient:
  Heavy objects               → database connections, HttpClient
  Objects that hold state     → counter would reset every injection
  Objects with IDisposable    → disposed after each use → resource waste
                                if injected many times in one request
```

### Proof — Different Instances

```csharp
public class TransientDemo
{
    public Guid Id { get; } = Guid.NewGuid();
}

builder.Services.AddTransient<TransientDemo>();

// Single request — injected twice (controller + another service):
// Controller gets:     Id = "aaaa-1111"
// OtherService gets:   Id = "bbbb-2222"  ← DIFFERENT instance, same request
```

---

## 🔷 Side-by-Side Comparison

```
┌──────────────────┬─────────────────┬─────────────────┬─────────────────┐
│                  │   SINGLETON     │    SCOPED       │   TRANSIENT     │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ How many         │ One for the     │ One per HTTP    │ New one every   │
│ instances?       │ entire app life │ request         │ injection       │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Shared?          │ All requests,   │ Within one      │ Never shared    │
│                  │ all users       │ request only    │                 │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Disposed?        │ App shutdown    │ Request end     │ After each use  │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Thread safety    │ YOUR job        │ Safe (one       │ Safe (never     │
│                  │ Required!       │ thread at a     │ shared)         │
│                  │                 │ time per req)   │                 │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Good for         │ Config readers  │ DAL, BAL        │ Email senders   │
│                  │ Cache services  │ DB contexts     │ Formatters      │
│                  │ HttpClient      │ Unit of Work    │ Validators      │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Bad for          │ DB connections  │ Injecting into  │ Heavy objects   │
│                  │ Per-user state  │ Singleton       │ Stateful svc    │
├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Register with    │ AddSingleton<>  │ AddScoped<>     │ AddTransient<>  │
└──────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

---

## 🔷 The Captured Dependency Problem

```
THIS IS THE MOST COMMON DI LIFETIME BUG.
Singleton captures a Scoped dependency → disaster.
```

```csharp
// ❌ WRONG — Singleton holds a reference to a Scoped service

public class ReportCache  // registered as Singleton
{
    private readonly IEmployeeBAL _bal;  // registered as Scoped ← PROBLEM

    public ReportCache(IEmployeeBAL bal)  // injected at startup
    {
        _bal = bal;
        // At startup: Scoped IEmployeeBAL is created for the ROOT scope
        // This Scoped instance is captured inside the Singleton.
        // It lives AS LONG AS the Singleton → the entire app lifetime.
        // But it was meant to live only ONE request.
        // The per-request scope is completely broken.
    }
}
```

```
ASP.NET Core detects this:
  InvalidOperationException:
  "Cannot consume scoped service 'IEmployeeBAL' from singleton 'ReportCache'"

  (Only in Development. In Production with ValidateScopes=false — silent corruption)
```

```csharp
// ✅ SOLUTION — use IServiceScopeFactory if Singleton truly needs Scoped

public class ReportCache  // Singleton
{
    private readonly IServiceScopeFactory _scopeFactory;

    public ReportCache(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;  // ScopeFactory itself is Singleton — safe
    }

    public List<Employee> GetCachedReport()
    {
        // Create a scope manually for this operation:
        using var scope = _scopeFactory.CreateScope();
        var bal = scope.ServiceProvider.GetRequiredService<IEmployeeBAL>();
        return bal.GetAll();
        // Scope disposed here → BAL disposed → clean
    }
}
```

---

## 🔷 Decision Flowchart

```
What are you registering?
│
├── Does it hold SHARED state across the whole application?
│   (cache, config, counters that survive requests)
│   └── YES → Singleton
│           ← Make sure it's THREAD-SAFE
│
├── Does it need to live for one COMPLETE request
│   and be shared within that request?
│   (DAL, BAL, Unit of Work, DbContext)
│   └── YES → Scoped  ← your default for most custom classes
│
└── Is it STATELESS? Cheap to create? Each use needs a fresh instance?
    (email sender, PDF builder, mapper, formatter)
    └── YES → Transient

QUICK RULE:
  Custom DAL / BAL classes → Scoped  (almost always)
  Configuration / Cache    → Singleton
  Helper utilities          → Transient
```

---

## 🔷 Your Real Project Registration

```csharp
// Program.cs — what lifetime to use for what

// ── Scoped: one per request ─────────────────────────────────────
// DAL and BAL are scoped — they access the DB, one per request is correct
builder.Services.AddScoped<IEmployeeDAL,    EmployeeDAL>();
builder.Services.AddScoped<IEmployeeBAL,    EmployeeBAL>();
builder.Services.AddScoped<IDepartmentDAL,  DepartmentDAL>();
builder.Services.AddScoped<IDepartmentBAL,  DepartmentBAL>();
builder.Services.AddScoped<ILeaveDAL,       LeaveDAL>();
builder.Services.AddScoped<ILeaveBAL,       LeaveBAL>();

// ── Transient: new every time ───────────────────────────────────
// Email service — stateless, each send is independent
builder.Services.AddTransient<IEmailService, SmtpEmailService>();
// PDF/Excel report builder — stateless, fresh build every time
builder.Services.AddTransient<IReportBuilder, ExcelReportBuilder>();

// ── Singleton: one for the whole app ────────────────────────────
// App settings — read once from config, shared everywhere
builder.Services.AddSingleton<IAppSettings, AppSettingsService>();
// In-memory cache wrapper — shared state is the point
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
// → AddMemoryCache() registers IMemoryCache as Singleton automatically
builder.Services.AddMemoryCache();
```

---

## ⭐ Interview Quick-Fire

| Question                                                          | Answer                                                                                                                            |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| What are the 3 DI lifetimes in ASP.NET Core?                      | Singleton (one forever), Scoped (one per request), Transient (new every injection)                                                |
| Which lifetime should a DAL or BAL use?                           | Scoped — one per HTTP request, disposed after the request ends                                                                   |
| Which lifetime should a configuration service use?                | Singleton — read once, safe to share since it's read-only                                                                        |
| Which lifetime should a stateless email sender use?               | Transient — lightweight, stateless, each send is independent                                                                     |
| What is the Captured Dependency Problem?                          | Injecting a Scoped service into a Singleton — the Scoped instance lives as long as the Singleton, breaking per-request isolation |
| How does ASP.NET Core help catch the captured dependency problem? | Throws`InvalidOperationException`in Development: "Cannot consume scoped service from singleton"                                 |
| Is a Scoped service thread-safe?                                  | ✅ Yes — each request gets its own instance, so no shared state between simultaneous requests                                    |
| Why must a Singleton be thread-safe?                              | It's shared across all simultaneous requests — multiple threads access it at once                                                |
| What is`IServiceScopeFactory`used for?                          | Allows a Singleton to create a temporary Scoped context when it needs Scoped services                                             |
| What happens with two injections of a Transient in one request?   | Two different instances — Transient is never shared, even within the same request                                                |
