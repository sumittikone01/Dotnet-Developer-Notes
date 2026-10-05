# 03_Exception_Filters

> **Exception Filters** = filters that run ONLY when an unhandled exception occurs inside an action method (or an earlier filter) — MVC-aware error handling, as distinct from the raw, generic `UseExceptionHandler()` middleware from `I.Middleware_and_Filters/02_Built_in_Middleware.md`.

> Continues **J.Filters**. Directly builds on `02_Action_Filters.md`'s pipeline diagram — this is the "if an exception is thrown here" branch that was only briefly mentioned there.

## 📌 What is it?

```csharp
public class ApiExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        // Runs ONLY if an exception propagated up from the action method (or an inner filter)
        context.Result = new ObjectResult(new { error = "Something went wrong." })
        {
            StatusCode = 500
        };
        context.ExceptionHandled = true; // marks the exception as HANDLED — stops it from propagating further
    }
}
```

## 🤔 Why do we need Exception Filters, given `UseExceptionHandler()` middleware already exists?

| Aspect                                     | `UseExceptionHandler()` Middleware            | Exception Filter                                                  |
| ------------------------------------------ | ----------------------------------------------- | ----------------------------------------------------------------- |
| Awareness of WHICH controller/action threw | ❌ None — sees only a raw exception            | ✅ Full MVC context:`ActionDescriptor`, route values, etc.      |
| Scope                                      | Global, app-wide by default                     | Can be applied globally OR scoped to specific controllers/actions |
| Access to`ModelState`/action arguments   | ❌ No                                           | ✅ Yes, via the`ExceptionContext`                               |
| Typical use                                | A single, app-wide fallback error page/response | Different error-handling behavior for DIFFERENT parts of the API  |

> **Rule of thumb:** use the middleware for a single, app-wide safety net; use Exception Filters when different Controllers/Actions need genuinely DIFFERENT exception-handling behavior, or when you need MVC-specific context in the handler.

## 🌍 Real-world analogy

`UseExceptionHandler()` middleware is like a **building-wide fire alarm system** — one system, triggers the same response (evacuate) no matter which room the fire started in. An Exception Filter is like having a **specific safety protocol for the chemistry lab** that's different from the one for the cafeteria — the SAME general concern (an emergency), but the specific department/context shapes exactly how it's handled.

## ⚙️ Internal working — where Exception Filters sit in the pipeline

```
[Action Filters: OnActionExecuting]
        │
        ▼
[ACTION METHOD RUNS]
        │
        ├── Exception thrown? ──► EXCEPTION FILTERS run (in registered order)
        │                              │
        │                    Can a filter HANDLE it?
        │                              │
        │                    ┌─────────┴─────────┐
        │                    ▼                   ▼
        │                   YES                  NO
        │           context.Result is set   Exception CONTINUES
        │           ExceptionHandled = true  propagating UP —
        │                    │                eventually reaches
        │                    ▼                UseExceptionHandler()
        │           [Result Filters run       middleware (or crashes
        │            with the filter's         if nothing catches it)
        │            Result]
        │
        No exception
        │
        ▼
[Action Filters: OnActionExecuted]
        │
        ▼
[Result Filters]
```

> **Critical:** `context.ExceptionHandled = true` is what actually STOPS the exception from propagating further. Forgetting this line means your filter's logic runs, but the exception STILL propagates up to the middleware (or crashes the app) anyway.

## 📊 What Exception Filters DON'T catch

```
🚨 Exception Filters do NOT catch exceptions thrown from:
   - Middleware (I.Middleware_and_Filters — outside the MVC pipeline entirely)
   - Authorization Filters
   - Resource Filters
   - Result Filters (an exception thrown here needs its OWN handling — Exception Filters
     only wrap the ACTION method's execution, not the later Result Filter stage)

They ONLY catch exceptions from:
   - The action method itself
   - Action Filters
```

> This is a very common gap — assuming an Exception Filter is a universal safety net for the ENTIRE request, when it actually only covers a specific slice of the pipeline. `UseExceptionHandler()` middleware remains the true universal fallback.

## 💻 Code examples

### Basic — a global Exception Filter for API error responses

```csharp
public class ApiExceptionFilter : IExceptionFilter
{
    private readonly ILogger<ApiExceptionFilter> _logger;
    public ApiExceptionFilter(ILogger<ApiExceptionFilter> logger) => _logger = logger;

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception in {Action}",
            context.ActionDescriptor.DisplayName);

        context.Result = new ObjectResult(new { error = "An unexpected error occurred." })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };
        context.ExceptionHandled = true;
    }
}

// Program.cs — register globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ApiExceptionFilter>();
});
```

### Intermediate — handling DIFFERENT exception types differently

```csharp
public class DomainExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        context.Result = context.Exception switch
        {
            ProductNotFoundException => new NotFoundObjectResult(new { error = context.Exception.Message }),
            InsufficientStockException => new ObjectResult(new { error = context.Exception.Message })
            {
                StatusCode = StatusCodes.Status409Conflict
            },
            ValidationException ve => new BadRequestObjectResult(new { errors = ve.Errors }),
            _ => null // leave 'context.Result' null for anything NOT explicitly handled
        };

        // Only mark as handled if we ACTUALLY set a result — otherwise let it propagate
        // to the middleware's generic UseExceptionHandler() as a true fallback
        context.ExceptionHandled = context.Result != null;
    }
}
```

### Practical — an async exception filter

```csharp
public class AsyncApiExceptionFilter : IAsyncExceptionFilter
{
    private readonly AuditService _auditService;
    public AsyncApiExceptionFilter(AuditService auditService) => _auditService = auditService;

    public async Task OnExceptionAsync(ExceptionContext context)
    {
        // Needs to 'await' — e.g., logging to a database via a Scoped service
        await _auditService.LogExceptionAsync(context.Exception, context.ActionDescriptor.DisplayName!);

        context.Result = new ObjectResult(new { error = "An unexpected error occurred." })
        {
            StatusCode = 500
        };
        context.ExceptionHandled = true;
    }
}
```

## ⚡ Performance considerations

- Exception Filters only run when something ACTUALLY goes wrong — they add zero overhead to the normal, successful request path.
- Logging inside an exception filter should still be reasonably fast — an exception-handling path that itself times out or throws creates a genuinely confusing failure mode.

## 🚨 Common mistakes

- ❌ Forgetting to set `context.ExceptionHandled = true` — the filter's logic runs, but the original exception still propagates, often resulting in a DIFFERENT (and less helpful) error response than intended.
- ❌ Assuming an Exception Filter catches EVERYTHING — it only covers exceptions from the action method and Action Filters, not Middleware, Authorization Filters, or Result Filters.
- ❌ Using an Exception Filter as the ONLY safety net for the app, with no `UseExceptionHandler()` middleware fallback — leaves gaps for exceptions thrown outside the MVC filter pipeline entirely.
- ❌ Swallowing ALL exceptions generically without distinguishing expected domain exceptions (like a custom `ProductNotFoundException`) from truly unexpected ones — loses useful, specific error information for the client.

## 💡 Best practices

- ✅ Always set `context.ExceptionHandled = true` when your filter has actually produced a `context.Result` for the exception.
- ✅ Use Exception Filters for MVC-aware, potentially DIFFERENT handling per Controller/Action; keep `UseExceptionHandler()` middleware as the app-wide fallback for anything NOT caught by a filter.
- ✅ Pattern-match on specific, known exception types (as in the Intermediate example) to return meaningful, differentiated error responses — rather than one generic "something went wrong" for every failure.
- ✅ Log the exception (with full context) before handling/suppressing it — don't let a handled exception disappear from your logs.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                     | Answer                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between an Exception Filter and`UseExceptionHandler()` middleware?   | The filter is MVC-aware (knows the Controller/Action, has access to ModelState) and can be scoped per-controller/action; the middleware is a generic, app-wide fallback with no MVC context |
| What line is required to actually stop an exception from propagating in an Exception Filter? | `context.ExceptionHandled = true;`                                                                                                                                                        |
| Do Exception Filters catch exceptions thrown by middleware?                                  | No — they only catch exceptions from the action method and Action Filters, not from middleware, Authorization Filters, or Result Filters                                                   |
| Why might you leave`context.ExceptionHandled` as false for certain exceptions?             | To let unhandled/unrecognized exception types propagate up to the middleware's generic fallback handler, rather than force-handling everything in the filter                                |
| When should you use`IAsyncExceptionFilter` instead of `IExceptionFilter`?                | When the exception-handling logic itself needs to`await` something, like logging to a database via a Scoped service                                                                       |

## 📝 30-second Revision Cheat Sheet

- Exception Filters run ONLY when an exception is thrown from the action method or an Action Filter — MVC-aware, unlike generic middleware.
- MUST set `context.ExceptionHandled = true` to actually stop the exception from propagating further.
- Do NOT catch exceptions from middleware, Authorization Filters, or Result Filters — those need `UseExceptionHandler()` or their own handling.
- Pattern-match on specific exception types for differentiated, meaningful error responses.
- Keep `UseExceptionHandler()` middleware as the true app-wide fallback alongside any Exception Filters.
