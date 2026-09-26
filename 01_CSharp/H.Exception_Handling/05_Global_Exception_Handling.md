
# Global Exception Handling in ASP.NET Core

## 📌 What is it?

Global exception handling is a **single, centralized place** that catches unhandled exceptions across an entire ASP.NET Core application — instead of wrapping every controller action in try-catch, one piece of middleware (or filter) handles it application-wide.

## 🤔 Why do we need it?

- Avoids repeating the same try-catch/logging code in every controller/action.
- Guarantees a **consistent error response format** for API consumers.
- Ensures unhandled exceptions never leak stack traces or internal details to clients.
- Single place to hook logging, alerting, and correlation IDs.

## 🧠 Intuition

It's a **safety net at the very edge of the request pipeline** — if an exception bubbles all the way up without being caught anywhere else, this is the last line of defense before it would otherwise crash the request/response cycle.

## 🌍 Real-world analogy

Like a building's **fire suppression system on the roof**, as opposed to putting a fire extinguisher in every single room. Individual rooms (controllers) can still have their own local handling for specific, expected issues, but anything that escapes gets caught by the one system covering the whole building.

## ⚙️ Internal working (ASP.NET Core pipeline)

1. Middleware is layered — request flows in, response flows back out.
2. Exception-handling middleware is registered **early** in the pipeline (`app.UseExceptionHandler(...)`), so it wraps everything after it.
3. When downstream middleware/controller code throws and nothing else catches it, execution unwinds back up through the pipeline.
4. The exception-handling middleware intercepts it, logs it, and writes a clean, structured response — the client never sees the raw exception.

## 🖼 Pipeline Diagram

```
Request
  │
  ▼
┌─────────────────────────────┐
│  UseExceptionHandler(...)   │ ◄──── catches anything that
│  (registered FIRST)         │       bubbles up from below
└─────────────┬────────────────┘
              ▼
┌─────────────────────────────┐
│      Routing Middleware      │
└─────────────┬────────────────┘
              ▼
┌─────────────────────────────┐
│   Controller / Endpoint      │ ── throws OrderNotFoundException
└─────────────┬────────────────┘
              │  (unhandled — bubbles up)
              ▼
     back to ExceptionHandler
              │
              ▼
     structured JSON error response
```

## 💻 Code Examples

### Option 1 — `UseExceptionHandler` with a handler lambda (.NET 6+ minimal API style)

```csharp
var app = builder.Build();

app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        var exception = feature?.Error;

        context.Response.ContentType = "application/json";
        context.Response.StatusCode = exception switch
        {
            OrderNotFoundException => StatusCodes.Status404NotFound,
            ArgumentException => StatusCodes.Status400BadRequest,
            _ => StatusCodes.Status500InternalServerError
        };

        var problem = new ProblemDetails
        {
            Title = "An error occurred",
            Status = context.Response.StatusCode,
            Detail = exception is OrderNotFoundException or ArgumentException
                ? exception.Message
                : "An unexpected error occurred." // hide internals for 500s
        };

        await context.Response.WriteAsJsonAsync(problem);
    });
});
```

### Option 2 — `IExceptionHandler` interface (.NET 8+ recommended approach)

```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
        => _logger = logger;

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Unhandled exception occurred");

        var (statusCode, title) = exception switch
        {
            OrderNotFoundException => (StatusCodes.Status404NotFound, "Order not found"),
            ArgumentException => (StatusCodes.Status400BadRequest, "Invalid request"),
            _ => (StatusCodes.Status500InternalServerError, "Internal server error")
        };

        httpContext.Response.StatusCode = statusCode;

        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Title = title,
            Status = statusCode,
            Detail = statusCode == 500 ? "Something went wrong." : exception.Message
        }, cancellationToken);

        return true; // exception handled — don't propagate further
    }
}

// Registration in Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
// ...
app.UseExceptionHandler();
```

### Option 3 — custom middleware (manual approach, pre-.NET 8 style)

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionMiddleware> _logger;

    public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context); // proceed down the pipeline
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception for {Path}", context.Request.Path);
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            await context.Response.WriteAsJsonAsync(new { error = "An unexpected error occurred." });
        }
    }
}

// Registration — must be one of the FIRST middlewares
app.UseMiddleware<ExceptionMiddleware>();
```

## 📊 Comparison Table — Approaches

| Approach                         | Since        | Notes                                                                |
| -------------------------------- | ------------ | -------------------------------------------------------------------- |
| Custom middleware                | Any version  | Full control, more boilerplate                                       |
| `UseExceptionHandler` (lambda) | .NET Core 3+ | Simple, built-in, less structured                                    |
| `IExceptionHandler`            | .NET 8+      | Recommended — DI-friendly, testable, composable (multiple handlers) |

## ⚡ Performance considerations

- Global handling adds negligible overhead — it only activates on the (hopefully rare) unhandled-exception path.
- Avoid heavy work (e.g., synchronous I/O) inside the handler itself, since it runs on the request's response path.

## 🚨 Common mistakes

- ❌ Registering exception-handling middleware **too late** in the pipeline — it must wrap everything that could throw.
- ❌ Returning full exception details (`ex.ToString()`) directly in the response — information disclosure risk.
- ❌ Using global handling as an excuse to remove **all** local try-catch — some errors still need local, specific handling (e.g., retry logic).
- ❌ Forgetting to also handle exceptions thrown during **startup** (before the pipeline is even built) — those need separate handling.
- ❌ Not distinguishing status codes — returning `500` for everything instead of mapping specific exceptions to `400`/`404`/etc.

## 💡 Best practices

- Prefer `IExceptionHandler` (.NET 8+) — supports multiple handlers, DI, and is easily unit-testable.
- Map known exception types to appropriate HTTP status codes; only mask details for truly unexpected (500) errors.
- Use `ProblemDetails` (RFC 7807) for consistent, standardized API error responses.
- Log with structured data (correlation ID, request path, user ID) for traceability.
- Keep local try-catch for **recoverable, specific** scenarios (e.g., retry on transient DB failure); let everything else bubble to the global handler.

## 🎤 Interview Questions

1. **Why must exception-handling middleware be registered early in the pipeline?**
   → Because middleware wraps everything registered after it; if registered late, exceptions thrown by earlier middleware won't be caught.
2. **What's the advantage of `IExceptionHandler` over a custom middleware?**
   → Built-in DI support, multiple handlers can be chained (first one that returns `true` "wins"), and it's more testable in isolation.
3. **Should global exception handling replace all local try-catch blocks?**
   → No — local handling is still needed for recoverable scenarios (retries, specific fallback logic); global handling is the last-resort safety net.
4. **What is `ProblemDetails` and why use it?**
   → A standardized (RFC 7807) JSON structure for API error responses, providing consistency across all error types returned by the API.
5. **How do you avoid leaking sensitive information in error responses?**
   → Only include exception messages for known, safe exception types; for unexpected/500 errors, return a generic message and log full details server-side only.

## 📝 30-second Revision Cheat Sheet

| Concept         | Key Point                                            |
| --------------- | ---------------------------------------------------- |
| Purpose         | One place to catch all unhandled exceptions app-wide |
| Registration    | Must be early/first in the middleware pipeline       |
| Modern approach | `IExceptionHandler` (.NET 8+)                      |
| Response format | Use`ProblemDetails` for consistency                |
| Security        | Never leak stack traces/internals in 500 responses   |
| Still need      | Local try-catch for specific, recoverable cases      |
