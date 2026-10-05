# 03_Custom_Middleware

> Writing your OWN middleware, beyond the inline `app.Use(...)` delegates from `01_Middleware_Concepts.md` — as a proper, reusable, testable **class**, following the convention ASP.NET Core itself uses for its built-in middleware.

> Closes out **I.Middleware_and_Filters**. Where `02_Built_in_Middleware.md` toured the ready-made options, this chapter is for when NONE of them fit — a genuinely custom, app-specific cross-cutting concern.

## 📌 What is it?

The inline `app.Use(async (context, next) => { ... })` style from `01_Middleware_Concepts.md` works fine for small, one-off logic. For anything more substantial — logic with dependencies injected, logic you'll unit test, logic reused across projects — a **middleware class** is the standard, more maintainable approach.

```csharp
// Inline style (fine for small/simple cases):
app.Use(async (context, next) => { /* ... */ await next(context); });

// Class-based style (better for anything nontrivial):
app.UseMiddleware<RequestLoggingMiddleware>();
```

## 🤔 Why do we need a class-based approach?

| Limitation of inline`app.Use(...)`                      | How a middleware class helps                                          |
| --------------------------------------------------------- | --------------------------------------------------------------------- |
| Hard to unit test a lambda embedded in`Program.cs`      | A class can be instantiated and tested independently                  |
| Can't easily use Dependency Injection for scoped services | Middleware classes get constructor-injected dependencies naturally    |
| Logic gets messy/long directly inside`Program.cs`       | Keeps`Program.cs` clean — just one `app.UseMiddleware<T>()` line |
| Hard to reuse across multiple projects                    | A middleware class can be packaged/shared like any other class        |

## 🌍 Real-world analogy

The inline `app.Use(...)` lambda is like a **sticky note reminder** taped to your monitor — fine for a quick, one-off thing. A middleware class is like a **proper standard operating procedure document** — formalized, reusable, testable, and something a new team member could pick up and understand on its own.

## ⚙️ Internal working — the conventional middleware class shape

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next; // reference to the NEXT middleware in the pipeline

    // Constructor: the RequestDelegate is injected automatically by the framework
    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // InvokeAsync (or Invoke) is called for EVERY request — this IS the middleware's logic
    public async Task InvokeAsync(HttpContext context)
    {
        var start = DateTime.UtcNow;

        await _next(context); // calls the NEXT middleware — exactly like next() in the inline style

        var duration = DateTime.UtcNow - start;
        Console.WriteLine($"{context.Request.Method} {context.Request.Path} → {context.Response.StatusCode} ({duration.TotalMilliseconds}ms)");
    }
}
```

```
Registration:                          What happens per-request:

app.UseMiddleware<RequestLoggingMiddleware>();

                                        1. ONE instance of RequestLoggingMiddleware is created
                                           at STARTUP (constructor runs ONCE)
                                        2. InvokeAsync() runs on EVERY SINGLE request
                                           (this is why scoped/per-request services CAN'T be
                                            constructor-injected directly — see below)
```

## 🚨 The Singleton Lifetime Trap — a critical detail

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ProductDal _dal; // ⚠️ DANGEROUS if ProductDal is registered as Scoped!

    public MyMiddleware(RequestDelegate next, ProductDal dal) // constructor runs ONCE at startup
    {
        _next = next;
        _dal = dal; // this captures the SAME instance for the LIFETIME of the app — a bug if ProductDal
                     // is meant to be created FRESH per request (e.g., wraps a per-request DB connection)
    }

    public async Task InvokeAsync(HttpContext context)
    {
        await _next(context);
    }
}
```

> **Middleware classes are effectively Singletons** — their CONSTRUCTOR runs once at app startup. If you need a **Scoped** service (like most DbContext-style dependencies) inside your middleware, inject it as a parameter on `InvokeAsync` instead — the framework resolves THAT per-request correctly.

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    public MyMiddleware(RequestDelegate next) => _next = next; // only Singleton-safe dependencies here

    // ✅ CORRECT — Scoped dependencies go on InvokeAsync's parameters, resolved FRESH per request
    public async Task InvokeAsync(HttpContext context, ProductDal dal)
    {
        // 'dal' here is a fresh, correctly-scoped instance for THIS request
        await _next(context);
    }
}
```

## 💻 Code examples

### Basic — a custom middleware class with a standard extension method wrapper

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger; // ILogger<T> is safe as Singleton — it's designed for this
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Incoming: {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context);
        _logger.LogInformation("Outgoing: {StatusCode}", context.Response.StatusCode);
    }
}

// Standard convention: an extension method makes registration read cleanly
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder app)
        => app.UseMiddleware<RequestLoggingMiddleware>();
}
```

```csharp
// Program.cs — registration reads like a built-in middleware now
app.UseRequestLogging();
```

### Intermediate — a custom middleware that reads a Scoped service per-request

```csharp
public class AuditingMiddleware
{
    private readonly RequestDelegate _next;
    public AuditingMiddleware(RequestDelegate next) => _next = next;

    // AuditService is Scoped — injected on InvokeAsync, NOT the constructor
    public async Task InvokeAsync(HttpContext context, AuditService auditService)
    {
        await _next(context);

        if (context.Response.StatusCode >= 400)
        {
            await auditService.LogFailedRequestAsync(context.Request.Path, context.Response.StatusCode);
        }
    }
}
```

### Practical — a custom middleware that adds a security header to every response

```csharp
public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;
    public SecurityHeadersMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.OnStarting(() => // ensures headers are set BEFORE the response actually starts sending
        {
            context.Response.Headers.Append("X-Content-Type-Options", "nosniff");
            context.Response.Headers.Append("X-Frame-Options", "DENY");
            return Task.CompletedTask;
        });

        await _next(context);
    }
}
```

## ⚡ Performance considerations

- Because a middleware class instance is created ONCE (Singleton-like) and `InvokeAsync` runs on every request, avoid doing expensive setup work inside `InvokeAsync` that could instead be done once in the constructor (for genuinely Singleton-safe dependencies).
- Keep the logic inside `InvokeAsync` fast — like all middleware, this runs for every matching request.

## 🚨 Common mistakes

- ❌ Injecting a Scoped service (like most `DbContext`/DAL classes) into the middleware CONSTRUCTOR — captures a single instance for the app's entire lifetime, causing subtle, hard-to-diagnose bugs (stale data, cross-request state leakage). Inject Scoped dependencies as `InvokeAsync` parameters instead.
- ❌ Forgetting to call `await _next(context)` — silently breaks the entire pipeline for every request that hits this middleware.
- ❌ Writing directly to `context.Response` after the response has already started being sent (e.g., after some content was already written) — causes runtime exceptions; use `context.Response.OnStarting(...)` for header modifications when in doubt.
- ❌ Not providing a clean extension method wrapper (`UseMyMiddleware()`) — while not required, it's the established convention that makes registration in `Program.cs` read consistently with the built-in middleware.

## 💡 Best practices

- ✅ Use a middleware CLASS (not an inline lambda) for anything beyond a few simple lines — improves testability and readability.
- ✅ Inject Singleton-safe dependencies (like `ILogger<T>`) via the constructor; inject Scoped dependencies via `InvokeAsync`'s parameters instead.
- ✅ Provide an `IApplicationBuilder` extension method (`UseYourMiddlewareName()`) for clean, conventional registration in `Program.cs`.
- ✅ Use `context.Response.OnStarting(...)` when modifying response headers, to guarantee it happens at the right point in the response lifecycle.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                     | Answer                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What two members does a conventional middleware class need?                  | A constructor accepting a`RequestDelegate next` parameter, and an `InvokeAsync(HttpContext context)` method                                                                                                |
| Why is injecting a Scoped service into a middleware's constructor dangerous? | The middleware instance (and its constructor-injected dependencies) is effectively a Singleton — a Scoped service captured there would incorrectly persist across requests instead of being fresh per request |
| Where should Scoped dependencies be injected in a custom middleware instead? | As parameters on the`InvokeAsync` method — the framework resolves them correctly, per request                                                                                                               |
| What is the standard convention for registering a custom middleware cleanly? | An`IApplicationBuilder` extension method (e.g., `UseRequestLogging()`) that wraps `app.UseMiddleware<T>()`                                                                                               |
| What happens if a custom middleware forgets to call`await _next(context)`? | The pipeline stops there entirely — no later middleware or the Controller ever runs for that request                                                                                                          |

## 📝 30-second Revision Cheat Sheet

- Custom middleware = a class with a `RequestDelegate next` constructor parameter and an `InvokeAsync(HttpContext)` method.
- The middleware instance is effectively a Singleton — constructor runs ONCE at startup.
- Inject Scoped dependencies (like most DAL/DbContext classes) via `InvokeAsync` parameters, NOT the constructor.
- Always call `await _next(context)` unless deliberately short-circuiting.
- Wrap registration in an `IApplicationBuilder` extension method for a clean `app.UseYourMiddleware()` call in `Program.cs`.

---

✅ **I.Middleware_and_Filters chapter complete** (01–03). Next up: **J.Filters**.
