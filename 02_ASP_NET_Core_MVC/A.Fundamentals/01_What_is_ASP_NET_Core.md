# 01 — What is ASP.NET Core?

## 📌 What is it?

**ASP.NET Core** is a free, open-source, cross-platform framework built by Microsoft for creating modern, cloud-based, internet-connected applications — web apps, APIs, microservices, and even background services.

It is a **complete rewrite** of the original ASP.NET Framework, designed from the ground up to be:

- **Cross-platform** — runs on Windows, Linux, macOS
- **Modular** — you only include what you need (via NuGet packages)
- **High-performance** — one of the fastest web frameworks in the industry
- **Cloud-ready** — built-in support for configuration, logging, containers (Docker), and dependency injection

## 🤔 Why do we need it?

Before ASP.NET Core, the old **ASP.NET Framework** had real limitations:

| Problem in old ASP.NET Framework                             | How ASP.NET Core solves it                                              |
| ------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Windows-only (IIS dependent)                                 | Runs on Windows, Linux, macOS                                           |
| Monolithic (`System.Web.dll` — heavy, everything bundled) | Modular NuGet packages — pay only for what you use                     |
| Tightly coupled to IIS                                       | Has its own built-in web server (**Kestrel**)                     |
| No built-in Dependency Injection                             | DI is baked into the framework core                                     |
| Slower performance                                           | Consistently ranks near the top in independent web framework benchmarks |
| Config via`web.config` (XML only)                          | Flexible config via JSON, environment variables, Azure Key Vault, etc.  |

## 🧠 Intuition

Think of the old ASP.NET Framework as a **large, all-in-one Swiss Army knife bolted to Windows** — powerful, but heavy and inflexible.

ASP.NET Core is more like a **toolbox where you pick only the tools you need**, and the toolbox itself can be carried to any workshop (OS) you want.

## 🌍 Real-world analogy

Imagine two restaurant kitchens:

- **Old ASP.NET (Framework):** A kitchen that only works in one specific building (Windows/IIS), comes with every appliance pre-installed whether you use it or not, and can't be easily moved.
- **ASP.NET Core:** A modular kitchen you can set up in *any* building (Linux, Windows, macOS, a container), and you only bring in the appliances (NuGet packages) relevant to the menu (your app) you're serving.

## ⚙️ Internal working (high-level)

1. Your app starts from `Program.cs`, which builds a **Host**.
2. The Host wires up configuration, logging, dependency injection (DI container), and the HTTP request pipeline.
3. **Kestrel** (the built-in lightweight web server) listens for HTTP requests.
4. Requests flow through a chain of **middleware** components (routing, auth, exception handling, etc.).
5. Eventually a request reaches your **MVC Controller** / **Razor Page** / **Minimal API endpoint**, which produces a response.
6. The response flows back out through the middleware pipeline to the client.

```
Client Request
      │
      ▼
   Kestrel (web server)
      │
      ▼
 Middleware Pipeline (Routing → Auth → Exception Handling → ...)
      │
      ▼
 Controller / Endpoint
      │
      ▼
   Response
      │
      ▼
Client Response
```

## 📊 ASP.NET Core editions/hosting models (quick orientation)

| Model                                 | Use case                                                      |
| ------------------------------------- | ------------------------------------------------------------- |
| **MVC (Model-View-Controller)** | Full web apps with server-rendered HTML views                 |
| **Razor Pages**                 | Page-focused web apps (simpler than MVC for CRUD-style pages) |
| **Web API**                     | Pure JSON/REST APIs, no views                                 |
| **Minimal APIs**                | Lightweight APIs with less boilerplate (introduced .NET 6+)   |
| **Blazor**                      | Web apps using C# instead of JavaScript for the client        |

> Since you work with ASP.NET (Web Forms/MVC background) + Kendo UI + AJAX, you'll mostly be dealing with the **MVC** and **Web API** models in this series.

## 🚨 Common mistakes

- Confusing **ASP.NET Core** with **ASP.NET Framework** (they're different products — Core is not "just a newer version," it's a re-architecture).
- Assuming Core is missing features — in reality almost everything from Framework has an equivalent, often better, implementation in Core.

## 💡 Best practices

- Always start new projects in **ASP.NET Core** (Framework is in maintenance mode with no new features).
- Understand the **.NET** version numbering: "ASP.NET Core" now just ships as part of **.NET** (e.g., .NET 8), the "Core" branding was dropped after .NET 5.

## 🎤 Interview questions

1. What is the key architectural difference between ASP.NET Framework and ASP.NET Core?
2. Why is ASP.NET Core considered cross-platform? What makes that possible?
3. What is Kestrel, and how does it relate to IIS?
4. Name three built-in features of ASP.NET Core that had to be added manually in ASP.NET Framework.

## 📝 30-second revision cheat sheet

- ASP.NET Core = cross-platform, modular, high-performance rewrite of ASP.NET.
- Has its own server (**Kestrel**), works without IIS.
- Built-in DI, flexible JSON-based configuration, faster than Framework.
- Supports MVC, Razor Pages, Web API, Minimal APIs, Blazor.
- "ASP.NET Core" is now just called **.NET** (post .NET 5).
