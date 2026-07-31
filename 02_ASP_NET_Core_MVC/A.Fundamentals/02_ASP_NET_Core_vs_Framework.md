# 02 — ASP.NET Core vs ASP.NET Framework

## 📌 What is it?

This topic clarifies the **concrete differences** between the two frameworks so you know exactly what changes when moving from your existing ASP.NET (Framework/MVC5-style or Web Forms) background into ASP.NET Core.

## 🤔 Why do we need it?

You already have experience with ASP.NET-style development. Knowing precisely *what changed* (and what stayed conceptually the same) will make the transition much faster — you're not learning web development from scratch, you're **remapping known concepts** to new implementations.

## 📊 Side-by-side comparison

| Aspect                         | ASP.NET Framework                             | ASP.NET Core                                                          |
| ------------------------------ | --------------------------------------------- | --------------------------------------------------------------------- |
| **Platform**             | Windows only                                  | Windows, Linux, macOS                                                 |
| **Web server**           | IIS only                                      | Kestrel (built-in) + can sit behind IIS/Nginx/Apache as reverse proxy |
| **Runtime**              | .NET Framework (CLR)                          | .NET (formerly .NET Core) — cross-platform runtime                   |
| **Project file**         | Heavy`.csproj` with explicit file listings  | Lightweight SDK-style`.csproj` (auto-includes files)                |
| **Configuration**        | `web.config` (XML)                          | `appsettings.json` + environment variables + `IConfiguration`     |
| **Dependency Injection** | Not built-in (needed Ninject, Unity, Autofac) | Built-in DI container from day one                                    |
| **Startup**              | `Global.asax`                               | `Program.cs` (unified in .NET 6+, was `Startup.cs` before)        |
| **Middleware pipeline**  | `HttpModules` / `HttpHandlers`            | Explicit middleware pipeline (`app.Use...`)                         |
| **Performance**          | Moderate                                      | Significantly faster (top-tier in TechEmpower benchmarks)             |
| **Packaging**            | Monolithic`System.Web.dll`                  | Modular NuGet packages                                                |
| **Hosting**              | IIS-bound, harder to containerize             | Native Docker/Kubernetes/cloud-native support                         |
| **Razor views**          | Yes (Razor engine)                            | Yes (same Razor syntax, enhanced)                                     |
| **Web Forms**            | Supported (`.aspx`)                         | **Not supported** — no equivalent, migrate to MVC/Razor Pages  |
| **Versioning/lifecycle** | Tied to .NET Framework (slow releases)        | Frequent releases (annual major versions)                             |
| **Open source**          | Partially                                     | Fully open-source on GitHub                                           |

## 🧠 Intuition

If ASP.NET Framework is like a **car with a fixed factory engine you can't swap**, ASP.NET Core is like a **modular car chassis** where you choose the engine (Kestrel or others), the wheels (Windows/Linux/macOS), and only bolt on the parts (NuGet packages) you actually need for the trip.

## ⚙️ What conceptually STAYS the same (good news for you)

Since you already know ASP.NET MVC concepts, these transfer directly:

- **MVC pattern** — Models, Views, Controllers still work the same way
- **Razor syntax** (`@Model`, `@foreach`, `@Html.something`) — nearly identical
- **Routing concepts** — conventional & attribute routing both still exist
- **Action methods, ActionResults** — same mental model
- **ViewBag / ViewData / TempData** — still exist, work the same

## ⚙️ What's genuinely NEW/different (focus your learning here)

- **`Program.cs`** replaces `Global.asax` + `Startup.cs` for app bootstrapping
- **Built-in Dependency Injection** is now central to how you register services
- **Middleware pipeline** replaces `HttpModules`
- **`appsettings.json`** replaces `web.config`
- **Tag Helpers** (new alternative to `@Html.` helpers, HTML-like syntax)
- **Kestrel** as the actual server process

## 🖼 Migration mental map

```
ASP.NET Framework Concept          →  ASP.NET Core Equivalent
─────────────────────────────────────────────────────────────
Global.asax (Application_Start)    →  Program.cs
web.config (<appSettings>)         →  appsettings.json + IConfiguration
HttpModules/HttpHandlers           →  Middleware (app.Use...)
Ninject/Unity (manual DI)          →  Built-in IServiceCollection
System.Web.dll (everything)        →  Individual NuGet packages
IIS-only hosting                   →  Kestrel (+ optional IIS/Nginx proxy)
Web Forms (.aspx)                  →  No equivalent (use MVC/Razor Pages)
```

## 🚨 Common mistakes

- Trying to find a `web.config` in a new ASP.NET Core project — it's `appsettings.json` now (though a minimal `web.config` can still exist for IIS deployment purposes only).
- Assuming Web Forms code can be "ported" — it cannot; it must be **rewritten** using MVC or Razor Pages.
- Forgetting that **DI is core** to ASP.NET Core — services must be explicitly registered in `Program.cs`, unlike Framework where you might instantiate objects directly.

## 💡 Best practices

- When migrating knowledge (not necessarily code) from Framework to Core, map old concepts to the table above rather than relearning from zero.
- Get comfortable with `Program.cs` and the DI container early — nearly everything else builds on top of these two.

## 🎤 Interview questions

1. What replaced `web.config` in ASP.NET Core, and why is that a better approach?
2. Is Web Forms supported in ASP.NET Core? What's the recommended alternative?
3. How does the built-in DI container in ASP.NET Core change the way services are consumed compared to ASP.NET Framework?
4. What's the role of Kestrel, and can it be used standalone in production?

## 📝 30-second revision cheat sheet

- Framework = Windows/IIS-only, monolithic, no built-in DI, `web.config`.
- Core = cross-platform, modular, built-in DI, `appsettings.json`, Kestrel server.
- MVC pattern, Razor syntax, routing concepts **carry over** — don't relearn these.
- Web Forms has **no direct equivalent** in Core.
- `Global.asax` → `Program.cs` is the biggest structural mental shift.
