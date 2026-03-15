
# 02 — ASP.NET Core vs ASP.NET Framework

---

## 🎯 One-Line Definition

> **ASP.NET Framework runs only on Windows using the old .NET Framework — ASP.NET Core is the modern rewrite that runs everywhere, starts faster, uses less memory, and is the only one still actively developed.**

---

## 🔷 The Two Worlds

```
┌─────────────────────────────────────────────────────────────────┐
│  ASP.NET FRAMEWORK (Legacy)          ASP.NET CORE (Modern)      │
│  ─────────────────────────           ──────────────────────     │
│  Born: 2002                          Born: 2016                 │
│  Platform: Windows ONLY              Platform: Win/Linux/Mac    │
│  Runtime: .NET Framework 4.x         Runtime: .NET 6/7/8/9      │
│  Web server: IIS ONLY                Web server: Kestrel + more │
│  Status: Maintenance only            Status: Actively developed │
│  Config: web.config (XML)            Config: appsettings.json   │
│  DI: not built-in                    DI: built-in               │
│  System.Web.dll: required            System.Web.dll: gone       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Performance — The Biggest Difference

```
ASP.NET Framework:
  Loads System.Web.dll → 30,000+ types in memory at startup
  Every request carries the full HttpContext weight
  Not designed for high concurrency

ASP.NET Core:
  Modular — only load what you use
  Kestrel web server built for async I/O
  TechEmpower benchmarks: ASP.NET Core consistently top 10 fastest
  Plaintext requests/sec:
    ASP.NET Core   ~7,000,000 req/s
    ASP.NET 4.x    ~  300,000 req/s

For your Employee Management app → either works fine.
For a platform serving millions → Core is the only choice.
```

---

## 🔷 Feature Comparison — Full Table

| Feature                                | ASP.NET Framework              | ASP.NET Core                        |
| -------------------------------------- | ------------------------------ | ----------------------------------- |
| **OS Support**                   | Windows only                   | Windows, Linux, macOS               |
| **Web Server**                   | IIS only                       | Kestrel, IIS, Nginx, Apache, Docker |
| **DI Container**                 | ❌ External (Autofac, Unity)   | ✅ Built-in                         |
| **Performance**                  | Good                           | Excellent (10–20x faster)          |
| **Configuration**                | `web.config`(XML)            | `appsettings.json`(JSON)          |
| **Async Support**                | Limited                        | Full async/await throughout         |
| **Middleware**                   | HTTP Modules/Handlers          | Clean middleware pipeline           |
| **Open Source**                  | ❌                             | ✅                                  |
| **NuGet packages**               | `System.Web.*`               | `Microsoft.AspNetCore.*`          |
| **Hosting**                      | IIS required                   | Self-host, Docker, cloud            |
| **Startup code**                 | `Global.asax`+`Web.config` | `Program.cs`only                  |
| **Tag Helpers**                  | ❌ (HTML Helpers only)         | ✅                                  |
| **Minimal APIs**                 | ❌                             | ✅                                  |
| **Active development**           | ❌ Bug fixes only              | ✅                                  |
| **Recommended for new projects** | ❌                             | ✅                                  |

---

## 🔷 Startup Code — What Changed

### ASP.NET Framework (old)

```csharp
// Global.asax.cs
protected void Application_Start()
{
    RouteConfig.RegisterRoutes(RouteTable.Routes);
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    BundleConfig.RegisterBundles(BundleTable.Bundles);
}

// Web.config — XML configuration
<connectionStrings>
    <add name="Default" connectionString="Server=.;Database=EmployeeDB;..." />
</connectionStrings>
<system.web>
    <authentication mode="Forms">
      <forms loginUrl="~/Account/Login" timeout="2880" />
    </authentication>
</system.web>
```

### ASP.NET Core (modern)

```csharp
// Program.cs — everything in ONE place
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllersWithViews();
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();

var app = builder.Build();

// Configure middleware pipeline
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

```json
// appsettings.json — clean JSON
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EmployeeDB;..."
  },
  "Logging": { "LogLevel": { "Default": "Information" } }
}
```

---

## 🔷 Hosting — What Changed

```
ASP.NET Framework:
  Must deploy to IIS on Windows Server
  IIS is the only option
  Can't run in Docker easily

ASP.NET Core:
  Built-in Kestrel web server — no IIS needed
  Run as a console app:  dotnet run
  Run in Docker:         docker run
  Run behind IIS:        yes (reverse proxy)
  Run behind Nginx:      yes (Linux server)
  Run on Azure App Service: yes
  Run as a Windows Service: yes
```

---

## 🔷 Dependency Injection — What Changed

```csharp
// ASP.NET Framework — NO built-in DI
// Had to install and configure Autofac, Unity, Ninject etc.
// Example with Unity:
container.RegisterType<EmployeeDAL>(new HierarchicalLifetimeManager());
container.RegisterType<EmployeeBAL>(new HierarchicalLifetimeManager());
DependencyResolver.SetResolver(new UnityDependencyResolver(container));

// ASP.NET Core — DI is BUILT IN
// No extra packages, no setup complexity:
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();
// Done — ASP.NET Core handles injection automatically
```

---

## 🔷 Should You Migrate Old Projects?

```
Keep on Framework if:
  ✅ App works fine, no new features needed
  ✅ Deep Windows Auth / Active Directory integration
  ✅ Third-party dependencies that don't support Core
  ✅ Budget/time not available for migration

Migrate to Core if:
  ✅ New project (always start with Core)
  ✅ Need Linux/Docker deployment
  ✅ Performance is critical
  ✅ Long-term maintenance matters
  ✅ Want modern C# features

New projects: ALWAYS start with ASP.NET Core.
```

---

## ⭐ Interview Quick-Fire

| Question                                                        | Answer                                                                                                                         |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| What is the main difference between ASP.NET Core and Framework? | Core is cross-platform, faster, open-source, and actively developed. Framework is Windows-only, legacy, maintenance mode only. |
| Can ASP.NET Framework run on Linux?                             | ❌ No — Windows only                                                                                                          |
| Is DI built into ASP.NET Framework?                             | ❌ No — needs external libraries                                                                                              |
| What replaced `web.config`?                                   | `appsettings.json`in ASP.NET Core                                                                                            |
| What replaced `Global.asax`?                                  | `Program.cs`in ASP.NET Core                                                                                                  |
| Should new projects use ASP.NET Framework?                      | ❌ No — always use ASP.NET Core for new projects                                                                              |
| What web server does ASP.NET Core use?                          | Kestrel (built-in) — can also run behind IIS, Nginx, Apache                                                                   |
