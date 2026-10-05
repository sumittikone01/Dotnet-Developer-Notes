# 08_Global_Exception_Handling

> The Web API-specific, end-to-end strategy for handling unhandled exceptions consistently across an ENTIRE API — bringing together `H.Exception_Handling`'s try/catch fundamentals, `I.Middleware_and_Filters`' `UseExceptionHandler()`, and `J.Filters`' Exception Filters into one coherent, practical setup.

> Continues **K.Web_API**. This is the "how do I actually wire all of this together for a real API" chapter — the payoff for everything learned in those three earlier chapters.

## 📌 What is it?

A Web API needs EVERY unhandled exception — no matter where it occurs — to result in a clean, consistent, `ProblemDetails`-shaped JSON response, NEVER a raw stack trace or an unhandled crash reaching the client.

```
Exception thrown ANYWHERE in the pipeline:
   - Middleware         → caught by UseExceptionHandler() middleware
   - Action method       → caught by an Exception Filter OR UseExceptionHandler()
   - Action Filter        → caught by an Exception Filter OR UseExceptionHandler()
   - Result Filter         → caught by UseExceptionHandler() ONLY (Exception Filters don't cover this)

ALL PATHS → end up producing the SAME consistent, ProblemDetails-shaped JSON response
```

## 🤔 Why do we need a GLOBAL strategy, not just scattered try/catch blocks?

| Problem with scattered try/catch                                             | How a global strategy helps                                                    |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Every action needs its own try/catch — repetitive, easy to forget somewhere | ONE central handler covers everything, consistently                            |
| Inconsistent error response shapes across different parts of the API         | A single global handler guarantees the SAME`ProblemDetails` shape everywhere |
| Risk of leaking a raw stack trace to the client in production                | A global handler ensures production responses are ALWAYS sanitized             |
| Hard to guarantee EVERY exception is logged                                  | One central point logs everything, with nothing able to slip through           |

## 🌍 Real-world analogy

Instead of giving every single department in a building its OWN separate fire extinguisher and evacuation plan (scattered try/catch), you have ONE building-wide fire safety system (global exception handling) that responds consistently no matter which room the fire started in — everyone gets the SAME, well-rehearsed, predictable response.

## 📊 Two Modern Approaches (ASP.NET Core 8+ `IExceptionHandler` vs the classic middleware)

| Approach                                                          | How it works                                                                                      | When introduced                    |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------- |
| **Classic: `UseExceptionHandler()` + a handler endpoint** | Middleware catches ANY exception, redirects/forwards to a dedicated error-handling endpoint       | Available since early ASP.NET Core |
| **Modern: `IExceptionHandler` interface**                 | Register one or more classes implementing`IExceptionHandler`; the framework calls them in order | ASP.NET Core 8+                    |

## ⚙️ Internal working — the classic `UseExceptionHandler()` approach, in full

```csharp
// Program.cs
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/problem+json";

        var exceptionHandlerFeature = context.Features.Get<IExceptionHandlerFeature>();
        var exception = exceptionHandlerFeature?.Error;

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "An unexpected error occurred.",
            Detail = app.Environment.IsDevelopment() ? exception?.ToString() : null // ONLY show details in DEV
        };

        await context.Response.WriteAsJsonAsync(problemDetails);
    });
});
```

```
🚨 CRITICAL: exception.ToString() (full stack trace) should ONLY ever be included
   in the response when app.Environment.IsDevelopment() is true.
   In production, this is a serious information-disclosure risk.
```

## ⚙️ Internal working — the modern `IExceptionHandler` approach (ASP.NET Core 8+)

```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) => _logger = logger;

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Unhandled exception occurred");

        var (statusCode, title) = exception switch
        {
            ProductNotFoundException => (StatusCodes.Status404NotFound, "Product not found"),
            ValidationException => (StatusCodes.Status400BadRequest, "Validation failed"),
            _ => (StatusCodes.Status500InternalServerError, "An unexpected error occurred")
        };

        httpContext.Response.StatusCode = statusCode;
        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = exception.Message
        }, cancellationToken);

        return true; // tells the framework: "yes, I handled this exception"
    }
}
```

```csharp
// Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails(); // enables automatic ProblemDetails for a few other built-in cases too

var app = builder.Build();
app.UseExceptionHandler(); // no lambda needed — delegates to the registered IExceptionHandler
```

> The `IExceptionHandler` approach is the cleaner, more testable, DI-friendly option in modern ASP.NET Core (8+) — prefer it for new projects over the classic lambda-based middleware approach.

## 📊 Where Exception Filters (`J.Filters/03_Exception_Filters.md`) still fit in

```
Even WITH a solid global handler in place, Exception Filters are still useful for:

  - Controller/Action-SPECIFIC exception handling that differs from the global default
  - Converting a specific domain exception into a specific, tailored response
    WITHOUT touching the global handler's general-purpose logic

Recommended layering:
  Exception Filters  → handle KNOWN, SPECIFIC domain exceptions with tailored responses
  Global Handler      → the universal, catch-all SAFETY NET for everything else
```

## 💻 Code examples

### Basic — a complete, production-ready setup (modern approach)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler(); // must be registered EARLY, per I.Middleware_and_Filters/01_Middleware_Concepts.md

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Intermediate — layering a domain-specific Exception Filter ON TOP of the global handler

```csharp
// Specific, action-scoped handling for a known business exception
public class InsufficientStockExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        if (context.Exception is InsufficientStockException ex)
        {
            context.Result = new ObjectResult(new ProblemDetails
            {
                Title = "Insufficient stock",
                Detail = ex.Message,
                Status = StatusCodes.Status409Conflict
            }) { StatusCode = 409 };
            context.ExceptionHandled = true; // handled HERE — never reaches the global handler
        }
        // anything ELSE falls through, unhandled, to the global IExceptionHandler as a safety net
    }
}

[ServiceFilter(typeof(InsufficientStockExceptionFilter))]
[HttpPost("{id}/order")]
public IActionResult PlaceOrder(int id, int quantity) => Ok(_service.PlaceOrder(id, quantity));
```

### Practical — custom domain exceptions designed to work well with this pipeline

```csharp
public class ProductNotFoundException : Exception
{
    public ProductNotFoundException(int id) : base($"Product with Id {id} was not found.") { }
}

public class InsufficientStockException : Exception
{
    public InsufficientStockException(int id, int requested, int available)
        : base($"Requested {requested} units of product {id}, but only {available} are in stock.") { }
}

// BAL throws these naturally — no try/catch needed in the Controller at all
public Order PlaceOrder(int productId, int quantity)
{
    var product = _dal.GetProductById(productId) ?? throw new ProductNotFoundException(productId);
    if (product.Stock < quantity)
        throw new InsufficientStockException(productId, quantity, product.Stock);

    return _dal.CreateOrder(productId, quantity);
}
```

## ⚡ Performance considerations

- A global exception handler only runs when something actually goes wrong — zero overhead on the successful request path.
- Logging every exception centrally (rather than scattered across many try/catch blocks) makes it far easier to monitor error rates and catch regressions — an operational, not just a code-cleanliness, benefit.

## 🚨 Common mistakes

- ❌ Including full exception details/stack traces in production responses — a serious information-disclosure risk; always gate this behind `IsDevelopment()`.
- ❌ Relying ONLY on scattered try/catch in every Controller action, with no global fallback — guarantees SOME exception, somewhere, will eventually slip through unhandled.
- ❌ Not logging exceptions that get handled/suppressed — losing visibility into real production issues just because they were gracefully converted to a 4xx/5xx response.
- ❌ Forgetting that Exception Filters don't cover exceptions from middleware or Result Filters (`J.Filters/03_Exception_Filters.md`) — the GLOBAL handler is what covers those gaps.

## 💡 Best practices

- ✅ Set up ONE global exception handler (`IExceptionHandler` in ASP.NET Core 8+, or the classic `UseExceptionHandler()` lambda otherwise) as the universal safety net.
- ✅ Layer domain-specific Exception Filters on top for Controller/Action-specific handling of KNOWN exception types, letting everything else fall through to the global handler.
- ✅ Always log exceptions centrally, even ones that are gracefully handled and converted to a client-friendly response.
- ✅ Gate detailed error information (stack traces, exception messages) behind `IsDevelopment()` — production responses should be clean and safe.
- ✅ Standardize on `ProblemDetails` for every error response shape across the API, matching what `[ApiController]`'s automatic validation handling already produces.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                        | Answer                                                                                                                                                 |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| What's the modern (ASP.NET Core 8+) way to implement global exception handling? | Implement`IExceptionHandler`, register it with `AddExceptionHandler<T>()`, and call `app.UseExceptionHandler()` with no lambda                   |
| Why shouldn't stack traces be included in production error responses?           | It's an information-disclosure risk — internal implementation details shouldn't be exposed to end users                                               |
| How do Exception Filters and a global exception handler work together?          | Exception Filters handle KNOWN, specific domain exceptions with tailored responses; the global handler is the universal safety net for everything else |
| What's a key benefit of centralizing exception handling, beyond cleaner code?   | Guarantees consistent logging and a uniform error response shape across the entire API, improving both monitoring and client experience                |
| What must a global exception handler ALWAYS do before returning a response?     | Log the exception (with full context) — even if it's being gracefully converted into a clean client-facing response                                   |

## 📝 30-second Revision Cheat Sheet

- Global exception handling = one central point (IExceptionHandler in .NET 8+, or UseExceptionHandler() lambda) that catches ANYTHING unhandled, anywhere in the pipeline.
- Layer domain-specific Exception Filters (`J.Filters`) on top for known exceptions; let everything else fall through to the global handler.
- NEVER expose stack traces/exception details in production — gate behind `IsDevelopment()`.
- Always log every exception centrally, even gracefully-handled ones.
- Standardize all error responses on `ProblemDetails` for consistency across the whole API.
