
# 03 — Registering Services

---

## 🎯 One-Line Definition

> **Registering a service means telling the DI container "when someone asks for THIS type, create THAT implementation with THIS lifetime" — all registrations happen in `Program.cs` before the app starts, and the container uses that map for every request.**

---

## 🔷 The Registration API — Core Syntax

```csharp
// The three methods — lifetime is the only difference:
builder.Services.AddSingleton <IService, Implementation>();
builder.Services.AddScoped    <IService, Implementation>();
builder.Services.AddTransient <IService, Implementation>();

// Generic syntax (most common):
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();
//                          ↑ interface   ↑ concrete class

// Non-generic syntax (same result):
builder.Services.AddScoped(typeof(IEmployeeBAL), typeof(EmployeeBAL));
```

---

## 🔷 Four Registration Styles

### Style 1: Interface → Implementation (Standard — Always Prefer This)

```csharp
// "When IEmployeeBAL is needed → create an EmployeeBAL"
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();

// Injection in constructor:
public EmployeeController(IEmployeeBAL bal) { _bal = bal; }
// ↑ depends on the interface — swap implementation without touching controller

// Why this is best:
// ✅ Loose coupling — controller doesn't know HOW EmployeeBAL is built
// ✅ Testable — inject FakeEmployeeBAL in unit tests
// ✅ Swappable — change implementation in ONE line of Program.cs
```

### Style 2: Concrete Type Only (No Interface)

```csharp
// Register the class directly — no interface involved
builder.Services.AddScoped<EmployeeBAL>();

// Injection:
public EmployeeController(EmployeeBAL bal) { _bal = bal; }
// ↑ depends on concrete class — tighter coupling

// When acceptable:
// ✅ Simple apps where you'll never swap the implementation
// ✅ Quick prototyping / internal helpers
// ❌ Avoid for anything you want to unit test
```

### Style 3: Instance Registration (Pre-built Object)

```csharp
// You create the instance — container just hands it out
var settings = new AppSettings
{
    PageSize = 10,
    AppName  = "Employee Portal"
};
builder.Services.AddSingleton<IAppSettings>(settings);
// ↑ This SPECIFIC instance is always returned — always Singleton lifetime

// When to use:
// ✅ Objects configured before app starts (read from config files)
// ✅ Objects expensive to create and safe to share (read-only)
// ✅ Test setup — inject a known, pre-built fake
// ⚠️  Always Singleton — the instance you provide lives forever
```

### Style 4: Factory Function (Complex / Conditional Construction)

```csharp
// You provide a lambda that builds the service
builder.Services.AddScoped<IEmployeeDAL>(provider =>
{
    // provider = IServiceProvider — get any registered service from it
    var config  = provider.GetRequiredService<IConfiguration>();
    var logger  = provider.GetRequiredService<ILogger<EmployeeDAL>>();
    var connStr = config.GetConnectionString("DefaultConnection");

    return new EmployeeDAL(connStr, logger);
    // Custom construction — not possible with simple generic registration
});

// Conditional factory — choose implementation based on config:
builder.Services.AddScoped<IStorageService>(provider =>
{
    var config     = provider.GetRequiredService<IConfiguration>();
    var storageType = config["Storage:Provider"];

    return storageType switch
    {
        "Azure" => new AzureBlobStorage(config["Azure:ConnectionString"]),
        "Local" => new LocalFileStorage(config["Storage:Path"]),
        _       => throw new InvalidOperationException(
                       $"Unknown storage provider: {storageType}")
    };
});

// Useful when:
// ✅ Constructor needs values not directly in the DI container
// ✅ You need conditional logic to choose which implementation
// ✅ Implementation needs runtime configuration
```

---

## 🔷 All Registration Methods — Reference

```csharp
// ── Standard ───────────────────────────────────────────────────────
builder.Services.AddSingleton<IService, Impl>();
builder.Services.AddScoped   <IService, Impl>();
builder.Services.AddTransient<IService, Impl>();

// ── Concrete only (no interface) ────────────────────────────────────
builder.Services.AddScoped<EmployeeBAL>();

// ── Pre-built instance (always Singleton) ───────────────────────────
builder.Services.AddSingleton<IAppSettings>(myInstance);

// ── Factory function ────────────────────────────────────────────────
builder.Services.AddScoped<IService>(provider => new MyImpl(
    provider.GetRequiredService<IDependency>()));

// ── TryAdd — register ONLY if not already registered ────────────────
builder.Services.TryAddScoped<IService, DefaultImpl>();
// If IService is already registered → this line is ignored
// Prevents accidentally overwriting a registration made elsewhere

// ── Replace — remove existing, add new ─────────────────────────────
builder.Services.Replace(
    ServiceDescriptor.Scoped<IEmailService, SendGridEmailService>());
// Useful in tests: replace production service with a test double

// ── Register one class for multiple interfaces ──────────────────────
builder.Services.AddScoped<EmployeeService>();
builder.Services.AddScoped<IEmployeeReader>(
    p => p.GetRequiredService<EmployeeService>());
builder.Services.AddScoped<IEmployeeWriter>(
    p => p.GetRequiredService<EmployeeService>());
// Both interfaces → resolved to the SAME Scoped EmployeeService instance

// ── Multiple implementations of same interface ──────────────────────
builder.Services.AddTransient<INotificationChannel, EmailNotification>();
builder.Services.AddTransient<INotificationChannel, SmsNotification>();
builder.Services.AddTransient<INotificationChannel, PushNotification>();

// Inject all of them as IEnumerable:
public class NotificationService
{
    private readonly IEnumerable<INotificationChannel> _channels;

    public NotificationService(IEnumerable<INotificationChannel> channels)
        => _channels = channels;  // contains Email + SMS + Push instances

    public async Task NotifyAll(string message)
    {
        foreach (var ch in _channels)
            await ch.SendAsync(message);
    }
}
```

---

## 🔷 Resolving Services — GetRequiredService vs GetService

```csharp
// ── GetRequiredService<T> — throws if not registered (PREFER THIS) ──
var bal = provider.GetRequiredService<IEmployeeBAL>();
// If IEmployeeBAL not registered:
// → InvalidOperationException: "No service for type 'IEmployeeBAL' registered"
// Clear error message — you know exactly what's missing

// ── GetService<T> — returns null if not registered ──────────────────
var bal = provider.GetService<IEmployeeBAL>();
// Returns null instead of throwing
// Use ONLY when the dependency is genuinely optional

// ── GetServices<T> — all implementations of an interface ────────────
var channels = provider.GetServices<INotificationChannel>();
// Returns IEnumerable — all registered INotificationChannel implementations

// ── [FromServices] in action parameters — for one-action-only deps ──
[HttpGet]
public IActionResult Export([FromServices] IReportBuilder builder)
{
    // builder resolved from DI for THIS action only
    // No need to inject into constructor if only one action needs it
    var file = builder.Build();
    return File(file, "application/xlsx", "report.xlsx");
}

// ── Resolving at startup (after Build) — for seeding, migrations ─────
var app = builder.Build();

using var scope = app.Services.CreateScope();
var dal = scope.ServiceProvider.GetRequiredService<IEmployeeDAL>();
// dal.SeedInitialData();  ← seed DB at startup
// scope disposed here → dal disposed
```

---

## 🔷 Framework Services — Built-In Registrations

```csharp
// These are provided by ASP.NET Core via extension methods:

// MVC Controllers + Razor Views:
builder.Services.AddControllersWithViews()
    .AddJsonOptions(options =>
        options.JsonSerializerOptions.PropertyNamingPolicy =
            System.Text.Json.JsonNamingPolicy.CamelCase);

// API Controllers only:
builder.Services.AddControllers();

// Session:
builder.Services.AddSession(options =>
{
    options.IdleTimeout        = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly    = true;
    options.Cookie.IsEssential = true;
    options.Cookie.Name        = ".EmpPortal.Session";
});

// In-memory cache:
builder.Services.AddMemoryCache();
// → IMemoryCache registered as Singleton automatically

// HttpClient (managed, avoids socket exhaustion):
builder.Services.AddHttpClient<IExternalApiService, ExternalApiService>(client =>
{
    client.BaseAddress = new Uri("https://api.external.com/");
    client.Timeout     = TimeSpan.FromSeconds(30);
});

// CORS:
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", p =>
        p.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
});

// Authentication (JWT):
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* token validation params */ });

// Authorization:
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
    options.AddPolicy("HROrAdmin", p => p.RequireRole("HR", "Admin"));
});
```

---

## 🔷 Keeping Program.cs Clean — Extension Methods

```
Program.cs grows large fast.
The solution: extension methods that group related registrations.
```

```csharp
// Extensions/EmployeeServiceExtensions.cs
public static class EmployeeServiceExtensions
{
    public static IServiceCollection AddEmployeeServices(
        this IServiceCollection services)
    {
        services.AddScoped<IEmployeeDAL, EmployeeDAL>();
        services.AddScoped<IEmployeeBAL, EmployeeBAL>();
        return services;  // ← return for fluent chaining
    }
}

// Extensions/DepartmentServiceExtensions.cs
public static class DepartmentServiceExtensions
{
    public static IServiceCollection AddDepartmentServices(
        this IServiceCollection services)
    {
        services.AddScoped<IDepartmentDAL, DepartmentDAL>();
        services.AddScoped<IDepartmentBAL, DepartmentBAL>();
        return services;
    }
}

// Extensions/InfrastructureExtensions.cs
public static class InfrastructureExtensions
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services)
    {
        services.AddTransient<IEmailService,   SmtpEmailService>();
        services.AddTransient<IReportBuilder,  ExcelReportBuilder>();
        services.AddSingleton<IAppSettings,    AppSettingsService>();
        return services;
    }
}
```

```csharp
// Program.cs — now clean and readable
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddSession(o =>
{
    o.IdleTimeout     = TimeSpan.FromMinutes(30);
    o.Cookie.HttpOnly = true;
    o.Cookie.IsEssential = true;
});
builder.Services.AddMemoryCache();

builder.Services.AddEmployeeServices();    // ← all employee DAL + BAL
builder.Services.AddDepartmentServices();  // ← all dept DAL + BAL
builder.Services.AddInfrastructure();      // ← email, reports, settings

var app = builder.Build();
// ...
```

---

## 🔷 Startup Validation

```csharp
// Validate all registrations at startup — catch errors before first request
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateOnBuild = true;   // check ALL registrations at build time
    options.ValidateScopes  = true;   // detect Singleton → Scoped bug
});

// In Development, ASP.NET Core enables ValidateScopes automatically.
// ValidateOnBuild must be set explicitly.

// What it catches:
// ❌ Missing registration  → "No service registered for IXxx"
// ❌ Captured dependency   → "Cannot consume Scoped from Singleton"
// Both caught at STARTUP → not on the first unlucky request
```

---

## 🔷 Common Mistakes

```csharp
// ── MISTAKE 1: Forgot to register ─────────────────────────────────
// InvalidOperationException:
// "No service for type 'IEmployeeBAL' has been registered"
// Fix: builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();


// ── MISTAKE 2: Registered concrete, injected interface ─────────────
builder.Services.AddScoped<EmployeeBAL>();          // registered concrete
public EmployeeController(IEmployeeBAL bal) { }     // asked for interface
// Error: No service for 'IEmployeeBAL'
// Fix: builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>();


// ── MISTAKE 3: Registered AFTER Build() ────────────────────────────
var app = builder.Build();                               // ← locked here
builder.Services.AddScoped<IEmployeeBAL, EmployeeBAL>(); // ← ignored/error
// Fix: all registrations MUST happen before builder.Build()


// ── MISTAKE 4: Duplicate registration ─────────────────────────────
builder.Services.AddScoped<IEmailService, GmailService>();
builder.Services.AddScoped<IEmailService, SmtpService>(); // both registered!
// When IEmailService is injected → LAST registration wins (SmtpService)
// If unintentional: use TryAddScoped to protect the first registration
// If intentional:   inject IEnumerable<IEmailService> to get both

builder.Services.TryAddScoped<IEmailService, GmailService>();
builder.Services.TryAddScoped<IEmailService, SmtpService>();  // ignored
// Now only GmailService is registered


// ── MISTAKE 5: Singleton captures Scoped ──────────────────────────
// (Covered in detail in chapter 02)
// Fix: use IServiceScopeFactory inside the Singleton
```

---

## 🔷 Full Program.cs — Real Project

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Framework services
builder.Services.AddControllersWithViews()
    .AddJsonOptions(o =>
        o.JsonSerializerOptions.PropertyNamingPolicy =
            System.Text.Json.JsonNamingPolicy.CamelCase);

builder.Services.AddSession(o =>
{
    o.IdleTimeout        = TimeSpan.FromMinutes(30);
    o.Cookie.HttpOnly    = true;
    o.Cookie.IsEssential = true;
});
builder.Services.AddMemoryCache();

// Data Access Layer — Scoped (one per request)
builder.Services.AddScoped<IEmployeeDAL,    EmployeeDAL>();
builder.Services.AddScoped<IDepartmentDAL,  DepartmentDAL>();
builder.Services.AddScoped<ILeaveDAL,       LeaveDAL>();
builder.Services.AddScoped<IPayrollDAL,     PayrollDAL>();

// Business Access Layer — Scoped (one per request)
builder.Services.AddScoped<IEmployeeBAL,    EmployeeBAL>();
builder.Services.AddScoped<IDepartmentBAL,  DepartmentBAL>();
builder.Services.AddScoped<ILeaveBAL,       LeaveBAL>();
builder.Services.AddScoped<IPayrollBAL,     PayrollBAL>();

// Infrastructure — Transient / Singleton
builder.Services.AddTransient<IEmailService,   SmtpEmailService>();
builder.Services.AddTransient<IReportBuilder,  ExcelReportBuilder>();
builder.Services.AddSingleton<IAppSettings,    AppSettingsService>();

var app = builder.Build();

// Middleware pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseSession();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
app.Run();
```

---

## ⭐ Interview Quick-Fire

| Question                                                                        | Answer                                                                                                         |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Where do you register services?                                                 | `Program.cs`using `builder.Services.Add*<Interface, Implementation>()`— before `builder.Build()`        |
| What is the difference between `AddScoped`,`AddSingleton`,`AddTransient`? | Scoped = per request, Singleton = whole app, Transient = per injection                                         |
| What does `TryAddScoped`do differently?                                       | Only registers if the interface isn't already registered — prevents accidental overwrite                      |
| What is a factory registration?                                                 | `AddScoped<IService>(provider => new MyImpl(...))`— you provide a lambda for custom construction            |
| What happens if you register the same interface twice?                          | Last registration wins for single injection. Use `IEnumerable<T>`to consume all                              |
| What does `GetRequiredService<T>`do vs `GetService<T>`?                     | `GetRequiredService`throws if not registered.`GetService`returns null. Prefer `GetRequiredService`always |
| How do you keep Program.cs clean?                                               | Extract registrations into static extension methods on `IServiceCollection`                                  |
| What does `ValidateOnBuild = true`catch?                                      | Missing service registrations and captured dependency bugs — at startup, not at runtime                       |
| Can you register after `builder.Build()`?                                     | ❌ No — container is locked after `Build()`                                                                 |
| What is `[FromServices]`used for?                                             | Resolves a service from DI directly into an action method parameter without adding it to the constructor       |
