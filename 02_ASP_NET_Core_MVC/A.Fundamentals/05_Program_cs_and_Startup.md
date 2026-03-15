
# 05 — Program.cs and Startup

---

## 🎯 One-Line Definition

> **`Program.cs` is the single entry point of every ASP.NET Core application — it does two jobs: register all services your app needs (DI container), and configure the middleware pipeline (the chain of checkpoints every request passes through).**

---

## 🔷 What Program.cs Actually Is

```
Before .NET 6:                   .NET 6+ (what you use):
──────────────────────────       ──────────────────────────────────
Program.cs                       Program.cs
  └─ CreateHostBuilder()         (everything in one place)
Startup.cs
  ├─ ConfigureServices()
  └─ Configure()

Both Startup.cs methods merged into Program.cs in .NET 6.
You will only ever see ONE file now.
```

---

## 🔷 Program.cs — Full Structure With Every Line Explained

```csharp
// ─────────────────────────────────────────────────────────────
// STEP 1: CREATE THE BUILDER
// WebApplication.CreateBuilder does four things automatically:
//   1. Reads appsettings.json (and appsettings.Development.json)
//   2. Reads environment variables
//   3. Sets up the built-in logging system
//   4. Creates the Dependency Injection (DI) container
// ─────────────────────────────────────────────────────────────
var builder = WebApplication.CreateBuilder(args);


// ─────────────────────────────────────────────────────────────
// STEP 2: REGISTER SERVICES (fill the DI container)
//
// Think of this as filling your toolbox before starting work.
// You're telling ASP.NET Core: "I need these tools available."
// Everything registered here can be injected into constructors.
// ─────────────────────────────────────────────────────────────

// MVC Controllers + Razor Views support:
builder.Services.AddControllersWithViews()
    .AddJsonOptions(options =>
    {
        // C# PascalCase property → JSON camelCase
        // ProductName → "productName" in JSON
        options.JsonSerializerOptions.PropertyNamingPolicy =
            System.Text.Json.JsonNamingPolicy.CamelCase;

        // Don't include null fields in JSON output
        options.JsonSerializerOptions.DefaultIgnoreCondition =
            System.Text.Json.Serialization.JsonIgnoreCondition.WhenWritingNull;
    });

// Your own DAL / BAL classes — register them here
// Scoped = one instance per HTTP request (correct for DB access)
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();

// Session support (if you use sessions)
builder.Services.AddSession(options =>
{
    options.IdleTimeout     = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

// In-memory cache
builder.Services.AddMemoryCache();

// Distributed cache (Redis) — for larger apps
// builder.Services.AddStackExchangeRedisCache(options =>
//     options.Configuration = builder.Configuration["Redis:Connection"]);


// ─────────────────────────────────────────────────────────────
// STEP 3: BUILD THE APP
// Finalizes the DI container. After this line, you cannot
// register new services — the container is locked.
// ─────────────────────────────────────────────────────────────
var app = builder.Build();


// ─────────────────────────────────────────────────────────────
// STEP 4: CONFIGURE MIDDLEWARE PIPELINE
//
// ORDER IS CRITICAL. Wrong order = broken authentication,
// missing CORS headers, exceptions not caught.
// ─────────────────────────────────────────────────────────────

// Different behavior for Development vs Production:
if (app.Environment.IsDevelopment())
{
    // Show detailed exception page with stack trace in browser
    app.UseDeveloperExceptionPage();
}
else
{
    // Show friendly error page, hide stack traces from users
    app.UseExceptionHandler("/Home/Error");

    // HSTS: tell browsers to ONLY use HTTPS for next 365 days
    app.UseHsts();
}

// Redirect all HTTP → HTTPS
app.UseHttpsRedirection();

// Serve files from wwwroot/: CSS, JS, images, Kendo files
app.UseStaticFiles();

// Enable routing system — match URLs to controllers
app.UseRouting();

// Session — must come after UseRouting, before controllers
app.UseSession();

// Auth middleware (if used):
app.UseAuthentication();  // READ JWT / cookie → set HttpContext.User
app.UseAuthorization();   // CHECK if user is allowed for this endpoint


// ─────────────────────────────────────────────────────────────
// STEP 5: MAP ROUTES
// Tell ASP.NET Core how to map URLs to Controllers/Actions
// ─────────────────────────────────────────────────────────────

// Conventional MVC route: /ControllerName/ActionName/id
app.MapControllerRoute(
    name:    "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
//           ↑ default controller   ↑ default action  ↑ optional id

// If you also have API controllers using [Route] attribute:
app.MapControllers();


// ─────────────────────────────────────────────────────────────
// STEP 6: START THE SERVER
// ─────────────────────────────────────────────────────────────
app.Run();
```

---

## 🔷 The Two Sections — Visual Split

```
Program.cs
│
├── SECTION A: builder.Services.*
│   ├── AddControllersWithViews()
│   ├── AddScoped<MyDAL>()
│   ├── AddSession()
│   └── AddMemoryCache()
│   │
│   │   → Fills the DI container
│   │   → "What tools does my app have?"
│   │   → Runs ONCE at startup
│
└── SECTION B: app.Use*  (after builder.Build())
    ├── UseExceptionHandler()
    ├── UseHttpsRedirection()
    ├── UseStaticFiles()
    ├── UseRouting()
    ├── UseAuthentication()
    ├── UseAuthorization()
    └── MapControllerRoute()
        │
        → Configures the middleware pipeline
        → "What happens to every request?"
        → Runs FOR EVERY request
```

---

## 🔷 builder.Services — Service Lifetimes

When you register your own classes, you choose how long an instance lives:

```csharp
// SCOPED — one instance per HTTP request (CORRECT for DAL/BAL)
builder.Services.AddScoped<EmployeeDAL>();
// → When request arrives: new EmployeeDAL created
// → Same instance used everywhere in that request
// → Disposed when request ends

// TRANSIENT — new instance every time injected
builder.Services.AddTransient<EmailService>();
// → New EmailService created every time it's injected
// → Use for lightweight, stateless services

// SINGLETON — one instance for the entire app lifetime
builder.Services.AddSingleton<ConfigReader>();
// → Created once when first needed
// → Same instance for ALL requests
// → MUST be thread-safe (multiple requests use it simultaneously)
```

```
Request 1 ──────────────────────────────────────────────────
  Scoped  │ new EmployeeDAL() created ─── used ─── disposed │
  Transient│ new every injection                             │
  Singleton│ ← SAME instance ──────────────────────────────→│

Request 2 ──────────────────────────────────────────────────
  Scoped  │ new EmployeeDAL() created ─── used ─── disposed │
  Transient│ new every injection                             │
  Singleton│ ← SAME instance (from Request 1) ─────────────→│
```

---

## 🔷 Reading Configuration in Your Classes

`Program.cs` reads `appsettings.json` automatically. Access it anywhere via `IConfiguration`:

```csharp
// In Program.cs itself:
var connStr = builder.Configuration.GetConnectionString("DefaultConnection");

// In a Controller or DAL — inject IConfiguration:
public class EmployeeDAL
{
    private readonly string _conn;

    public EmployeeDAL(IConfiguration config)
    {
        _conn = config.GetConnectionString("DefaultConnection");
        // Reads: appsettings.json → ConnectionStrings → DefaultConnection
    }
}
```

```json
// appsettings.json (read automatically by CreateBuilder):
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB;Integrated Security=True;"
  },
  "AppSettings": {
    "PageSize": 10,
    "AppName": "Employee Portal"
  }
}
```

```csharp
// Reading non-connection-string values:
int pageSize = config.GetValue<int>("AppSettings:PageSize");   // 10
string name  = config["AppSettings:AppName"];                  // "Employee Portal"
```

---

## 🔷 Environment — Dev vs Production Behavior

```csharp
if (app.Environment.IsDevelopment())
{
    // ← This block only runs when ASPNETCORE_ENVIRONMENT = "Development"
    app.UseDeveloperExceptionPage();   // full stack trace in browser
}
else
{
    // ← Production / Staging
    app.UseExceptionHandler("/Home/Error");  // friendly error page
    app.UseHsts();
}
```

```
How environment is set:
──────────────────────────────────────────────────────────────
Local dev:     launchSettings.json → "ASPNETCORE_ENVIRONMENT": "Development"
Staging:       Environment variable ASPNETCORE_ENVIRONMENT=Staging
Production:    Environment variable ASPNETCORE_ENVIRONMENT=Production
```

```csharp
// Check environment anywhere:
if (app.Environment.IsDevelopment())   { /* dev only */ }
if (app.Environment.IsProduction())    { /* prod only */ }
if (app.Environment.IsStaging())       { /* staging only */ }
if (app.Environment.IsEnvironment("Testing")) { /* custom name */ }
```

---

## 🔷 appsettings.json + appsettings.Development.json

```
ASP.NET Core loads config in this order (later overrides earlier):
──────────────────────────────────────────────────────────────
1. appsettings.json                   ← base settings (all environments)
2. appsettings.{Environment}.json     ← environment-specific overrides
3. Environment variables              ← server/container overrides
4. Command-line arguments             ← runtime overrides
```

```json
// appsettings.json — shared base:
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=PROD-SERVER;Database=EmployeeDB;..."
  },
  "Logging": { "LogLevel": { "Default": "Warning" } }
}

// appsettings.Development.json — overrides for dev machine:
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB_Dev;Integrated Security=True;"
  },
  "Logging": { "LogLevel": { "Default": "Debug" } }
}
```

On your dev machine → uses `appsettings.Development.json` connection string.
On production server → uses `appsettings.json` connection string.
Same code, different behavior. No hardcoding.

---

## 🔷 Old Way vs New Way — Side by Side

```
.NET 5 and earlier:                  .NET 6+ (current):
──────────────────────────────────   ──────────────────────────────────
// Program.cs                        // Program.cs (everything here)
public class Program                 var builder = WebApplication
{                                        .CreateBuilder(args);
    public static void Main(string[] args)
    {                                builder.Services
        CreateHostBuilder(args)          .AddControllersWithViews();
            .Build()
            .Run();                  var app = builder.Build();
    }
    public static IHostBuilder       app.UseStaticFiles();
        CreateHostBuilder(            app.UseRouting();
            string[] args) =>         app.UseAuthorization();
        Host.CreateDefaultBuilder()   app.MapControllerRoute(
            .ConfigureWebHostDefaults     "default",
            (webBuilder =>                "{controller=Home}/{action=Index}/{id?}");
            {
                webBuilder               app.Run();
                    .UseStartup<Startup>();
            });
}

// Startup.cs
public class Startup
{
    public void ConfigureServices(IServiceCollection s)
    {
        s.AddControllersWithViews();
    }
    public void Configure(IApplicationBuilder app)
    {
        app.UseStaticFiles();
        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
            endpoints.MapControllerRoute(...));
    }
}
```

---

## 🔷 Minimal Program.cs vs Full Program.cs

```csharp
// ── MINIMAL (new project default) ─────────────────────────────────
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
app.Run();

// ── YOUR REAL PROJECT (session + DI + JSON config) ─────────────────
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddJsonOptions(o => o.JsonSerializerOptions.PropertyNamingPolicy =
        System.Text.Json.JsonNamingPolicy.CamelCase);

builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();
builder.Services.AddSession(o => o.IdleTimeout = TimeSpan.FromMinutes(30));
builder.Services.AddMemoryCache();

var app = builder.Build();

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

| Question                                                                        | Answer                                                                                        |
| ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| What is `Program.cs`in ASP.NET Core?                                          | The entry point — registers services (DI) and configures the middleware pipeline             |
| What did `Startup.cs`do and where is it now?                                  | Had `ConfigureServices`and `Configure`methods — both merged into `Program.cs`in .NET 6 |
| What does `builder.Services.AddScoped<T>()`do?                                | Registers a class in the DI container with scoped lifetime (one instance per request)         |
| What is the difference between `AddScoped`,`AddTransient`,`AddSingleton`? | Scoped = per request, Transient = per injection, Singleton = once for entire app              |
| When does `builder.Build()`get called?                                        | After all services are registered — locks the DI container                                   |
| Can you register services after `builder.Build()`?                            | ❌ No — container is finalized after Build()                                                 |
| What is `IsDevelopment()`used for?                                            | To show detailed error pages and verbose logging only in development, not production          |
| How does ASP.NET Core read `appsettings.json`?                                | Automatically via `WebApplication.CreateBuilder(args)`— no extra code needed               |
| What overrides `appsettings.json`?                                            | `appsettings.{Environment}.json`→ Environment variables → Command-line args               |
