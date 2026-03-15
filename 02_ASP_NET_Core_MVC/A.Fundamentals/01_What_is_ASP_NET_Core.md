
# 01 — What is ASP.NET Core?

---

## 🎯 One-Line Definition

> **ASP.NET Core is Microsoft's modern, open-source, cross-platform web framework for building web applications, REST APIs, and real-time apps using C# — built from scratch to be fast, lightweight, and run anywhere.**

---

## 🔷 The Problem ASP.NET Core Solves

```
OLD ASP.NET (Framework):
──────────────────────────────────────────────────────────────
  Windows only        → can't deploy on Linux/Mac servers
  Heavy and slow      → System.Web.dll = 30,000+ types loaded
  Tightly coupled     → Web Forms wired to Windows IIS
  Hard to test        → HttpContext not injectable/mockable
  Not cloud-friendly  → wasn't designed for containers or microservices
  Closed source       → Microsoft controlled all changes

ASP.NET CORE:
──────────────────────────────────────────────────────────────
  Cross-platform      → runs on Windows, Linux, macOS
  Fast and lightweight→ benchmarks among fastest web frameworks
  Fully open-source   → github.com/dotnet/aspnetcore
  Cloud native        → built for Docker, Kubernetes, Azure
  Testable            → every piece is injectable and mockable
  Unified             → MVC, Web API, Razor Pages in one framework
```

---

## 🔷 What You Can Build With ASP.NET Core

```
┌────────────────────────────────────────────────────────────────┐
│  ASP.NET Core                                                   │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │  MVC Web App    │  │   Web API        │  │  Razor Pages  │  │
│  │  (your stack)   │  │  (REST/JSON)     │  │  (page-based) │  │
│  │                 │  │                  │  │               │  │
│  │  Views + Razor  │  │  [ApiController] │  │  .cshtml +    │  │
│  │  Controllers    │  │  JSON responses  │  │  PageModel    │  │
│  └─────────────────┘  └──────────────────┘  └───────────────┘  │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐                     │
│  │  Blazor         │  │  SignalR          │                     │
│  │  (C# in browser)│  │  (real-time)     │                     │
│  └─────────────────┘  └──────────────────┘                     │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Key Characteristics

| Feature                        | What It Means                                                |
| ------------------------------ | ------------------------------------------------------------ |
| **Cross-platform**       | Runs on Windows, Linux, macOS — deploy anywhere             |
| **Open source**          | Source code on GitHub — community contributions             |
| **High performance**     | One of the fastest web frameworks in benchmarks              |
| **Modular**              | Only include what you need — no bloat                       |
| **Unified**              | MVC + Web API + Razor Pages in one framework                 |
| **Cloud native**         | Built for Docker, Kubernetes, microservices                  |
| **Dependency Injection** | Built-in DI container — no extra libraries needed           |
| **Testable**             | Every component designed to be unit-testable                 |
| **Modern C#**            | Uses async/await, records, minimal APIs, nullable references |

---

## 🔷 ASP.NET Core Version Timeline

```
ASP.NET Core 1.0  (2016) → First release, complete rewrite
ASP.NET Core 2.0  (2017) → Razor Pages, SignalR
ASP.NET Core 3.0  (2019) → Worker services, gRPC support
ASP.NET Core 5.0  (2020) → Merged with .NET 5 (no more "Core" branding)
ASP.NET Core 6.0  (2021) → Minimal APIs, .NET 6 LTS
ASP.NET Core 7.0  (2022) → Rate limiting, output caching
ASP.NET Core 8.0  (2023) → .NET 8 LTS, Blazor improvements
ASP.NET Core 9.0  (2024) → Latest release

LTS = Long Term Support (3 years of support)
Current LTS = .NET 8
```

---

## 🔷 Where Your Stack Fits

You are building **ASP.NET Core MVC** applications:

```
YOUR APPLICATION STACK:
──────────────────────────────────────────────────────────────
Browser
  ↓ HTTP request
ASP.NET Core (handles routing, DI, middleware)
  ↓
Controller (C# — your code)
  ↓
BAL (Business rules — C#)
  ↓
DAL (ADO.NET — SqlConnection, SqlCommand)
  ↓
SQL Server Database
```

---

## 🔷 ASP.NET Core vs ASP.NET MVC 5 — Quick Snapshot

|                | ASP.NET MVC 5 (old)      | ASP.NET Core MVC            |
| -------------- | ------------------------ | --------------------------- |
| Platform       | Windows only             | Windows, Linux, macOS       |
| Web server     | IIS only                 | Kestrel, IIS, Nginx, Apache |
| Performance    | Slower                   | Significantly faster        |
| DI container   | Not built-in (use NuGet) | Built-in                    |
| `System.Web` | Required (heavy)         | Not used                    |
| Open source    | ❌                       | ✅                          |
| .NET version   | .NET Framework 4.x       | .NET 6/7/8/9                |
| Configuration  | Web.config (XML)         | appsettings.json            |

---

## ⭐ Interview Quick-Fire

| Question                               | Answer                                                                                       |
| -------------------------------------- | -------------------------------------------------------------------------------------------- |
| What is ASP.NET Core?                  | Microsoft's cross-platform, open-source web framework for building web apps and APIs with C# |
| Is ASP.NET Core open source?           | ✅ Yes — github.com/dotnet/aspnetcore                                                       |
| What can you build with it?            | MVC web apps, REST APIs, Razor Pages, Blazor apps, SignalR real-time apps                    |
| What is the current LTS version?       | .NET 8 (ASP.NET Core 8)                                                                      |
| What web server does ASP.NET Core use? | Kestrel (built-in, cross-platform) — can also run behind IIS, Nginx, Apache                 |
| Is DI built into ASP.NET Core?         | ✅ Yes — no NuGet packages needed                                                           |
