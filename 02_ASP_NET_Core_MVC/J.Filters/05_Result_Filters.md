
# 05_Result_Filters

> **Result Filters** = filters that run immediately BEFORE and AFTER the `ActionResult` actually EXECUTES — i.e., before/after the response is actually written to the client. The final stage in `01_Filters_Overview.md`'s five-stage pipeline.

> Closes out **J.Filters** (01–05). Distinct from `02_Action_Filters.md`: Action Filters wrap the ACTION METHOD's execution (getting the DATA ready); Result Filters wrap the RESULT's execution (getting the RESPONSE actually sent).

## 📌 What is it?

```csharp
public class AddHeaderResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        // Runs BEFORE the result (e.g., JSON serialization) actually executes
        context.HttpContext.Response.Headers.Append("X-App-Version", "1.0");
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // Runs AFTER the result has been written to the response
        Console.WriteLine($"Result executed: {context.HttpContext.Response.StatusCode}");
    }
}
```

## 🤔 Why do we need Result Filters, distinct from Action Filters?

| Aspect                                                              | Action Filter (`02_Action_Filters.md`)                                             | Result Filter                                                                                 |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| Wraps                                                               | The ACTION METHOD's execution (business logic, returning an`IActionResult` OBJECT) | The RESULT's EXECUTION (actually serializing/writing that object to the response)             |
| Can it modify the action's RETURN VALUE before it becomes a result? | ✅ Yes, in`OnActionExecuted` (via `context.Result`)                              | ❌ Too late — the`IActionResult` object already exists by this point                       |
| Can it modify response HEADERS right before writing?                | Awkward — response may already be starting                                          | ✅ This is exactly its purpose — runs right before the result WRITES the response            |
| Typical use                                                         | Timing, logging, argument/result inspection                                          | Adding response headers, wrapping/formatting the FINAL written output, response caching logic |

## 🌍 Real-world analogy

If the Action Filter is the **chef finishing a dish and putting it on a plate** (`OnActionExecuted` — the `IActionResult` object now exists), the Result Filter is the **waiter doing a final check and garnish right before it leaves the kitchen** (`OnResultExecuting`) and **confirming the plate actually reached the table** (`OnResultExecuted`) — a distinct, LATER stage than the cooking itself.

## ⚙️ Internal working — where Result Filters sit (recap from `01_Filters_Overview.md`)

```
[Action Filters: OnActionExecuted]
        │
        ▼
   (the ActionResult object now exists — e.g., an ObjectResult wrapping a Product)
        │
        ▼
1. RESULT FILTERS run: OnResultExecuting
        │
        ▼
   [The ActionResult ACTUALLY EXECUTES — e.g., serializes the Product to JSON and writes it]
        │
        ▼
2. RESULT FILTERS run: OnResultExecuted
        │
        ▼
   (Response continues back out through the Middleware pipeline)
```

## 📊 Sync vs Async Result Filters

| Interface              | Methods                                                                            |
| ---------------------- | ---------------------------------------------------------------------------------- |
| `IResultFilter`      | `OnResultExecuting`, `OnResultExecuted`                                        |
| `IAsyncResultFilter` | `OnResultExecutionAsync(context, next)` — needed if you must `await` anything |

```csharp
public class TimingAsyncResultFilter : IAsyncResultFilter
{
    public async Task OnResultExecutionAsync(ResultExecutingContext context, ResultExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();
        await next(); // this is where the result ACTUALLY executes (writes the response)
        sw.Stop();
        Console.WriteLine($"Result execution took {sw.ElapsedMilliseconds}ms");
    }
}
```

## 🖼 Short-circuiting a Result Filter — replacing the result entirely

```csharp
public class CachedResponseResultFilter : IResultFilter
{
    private readonly IMemoryCache _cache;
    public CachedResponseResultFilter(IMemoryCache cache) => _cache = cache;

    public void OnResultExecuting(ResultExecutingContext context)
    {
        var cacheKey = context.HttpContext.Request.Path.ToString();

        if (_cache.TryGetValue(cacheKey, out string? cachedJson))
        {
            // REPLACE the result entirely with a cached response — the ORIGINAL
            // result (from the action) never actually executes/serializes at all!
            context.Result = new ContentResult
            {
                Content = cachedJson,
                ContentType = "application/json"
            };
            context.Cancel = true; // skips the REST of the result-filter pipeline for THIS result
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}
```

> `context.Cancel = true` in a Result Filter is the equivalent of setting `context.Result` in an Action Filter's `OnActionExecuting` — it short-circuits the remaining pipeline stage.

## 💻 Code examples

### Basic — a Result Filter adding a consistent response header

```csharp
public class ApiVersionHeaderFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers.Append("X-Api-Version", "2.0");
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}

// Program.cs — apply globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ApiVersionHeaderFilter>();
});
```

### Intermediate — wrapping every successful JSON response in a consistent envelope

```csharp
public class ResponseEnvelopeFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.Result is ObjectResult objectResult &&
            objectResult.StatusCode is null or >= 200 and < 300)
        {
            objectResult.Value = new
            {
                success = true,
                timestamp = DateTime.UtcNow,
                data = objectResult.Value
            };
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}
```

### Practical — using `IAsyncResultFilter` for post-response logging with a Scoped service

```csharp
public class ResponseAuditResultFilter : IAsyncResultFilter
{
    private readonly AuditService _auditService; // Scoped — safe via ServiceFilter/DI registration
    public ResponseAuditResultFilter(AuditService auditService) => _auditService = auditService;

    public async Task OnResultExecutionAsync(ResultExecutingContext context, ResultExecutionDelegate next)
    {
        await next(); // let the result actually execute (write the response) FIRST

        // Now log AFTER the response was actually sent — status code is finalized
        await _auditService.LogResponseAsync(
            context.HttpContext.Request.Path,
            context.HttpContext.Response.StatusCode);
    }
}
```

## ⚡ Performance considerations

- Result Filters run on every request whose action produces a matching result — keep logic here fast, since it directly delays the response actually being sent to the client.
- Short-circuiting a Result Filter (like the caching example) can be a genuine performance win — skipping the actual serialization work entirely for a cached response.
- Modifying headers in `OnResultExecuting` is safe; attempting to modify headers in `OnResultExecuted` (AFTER the response has started sending) can throw, since headers must be set before the response body begins.

## 🚨 Common mistakes

- ❌ Trying to modify response HEADERS in `OnResultExecuted` instead of `OnResultExecuting` — by that point, the response may have already started sending, and headers can no longer be changed (throws an exception).
- ❌ Confusing Result Filters with Action Filters — trying to inspect/modify the action's RETURN VALUE (before it became an `IActionResult`) inside a Result Filter, when that data is only available in Action Filters.
- ❌ Forgetting `context.Cancel = true` when replacing `context.Result` inside `OnResultExecuting` — without it, the ORIGINAL result may still attempt to execute as well.
- ❌ Doing expensive work in `OnResultExecuting` that delays every response being sent — a Result Filter runs squarely in the critical path of returning data to the client.

## 💡 Best practices

- ✅ Use Result Filters specifically for response-shaping concerns: headers, response envelopes, output caching logic — things that need to happen right around the ACTUAL writing of the response.
- ✅ Set response headers in `OnResultExecuting`, never in `OnResultExecuted`.
- ✅ Use `context.Cancel = true` alongside replacing `context.Result`, to properly short-circuit the rest of the result pipeline.
- ✅ Prefer `IAsyncResultFilter` when the filter needs to `await` anything, especially logging AFTER the response has been sent (post-response auditing).

## 🎤 Interview Quick-Fire Q&A

| Question                                                                            | Answer                                                                                                                                                            |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the key difference between an Action Filter and a Result Filter?             | Action Filters wrap the action METHOD's execution (business logic); Result Filters wrap the RESULT's execution (actually writing the response)                    |
| Why must response headers be set in`OnResultExecuting`, not `OnResultExecuted`? | By`OnResultExecuted`, the response may have already started sending, and headers can no longer be modified without throwing an exception                        |
| How do you short-circuit a Result Filter and replace the result entirely?           | Set`context.Result` to a new result AND set `context.Cancel = true` in `OnResultExecuting`                                                                  |
| When would you use`IAsyncResultFilter` over `IResultFilter`?                    | When the filter needs to`await` something — e.g., logging AFTER the response has been sent, using a Scoped service                                             |
| Give a genuine real-world use case for a Result Filter.                             | Adding a consistent response header, wrapping successful JSON responses in an envelope, or serving a cached response without executing the original result at all |

## 📝 30-second Revision Cheat Sheet

- Result Filters wrap the RESULT's execution (writing the response) — the final stage after Action Filters.
- `OnResultExecuting` (before the response is written) vs `OnResultExecuted` (after) — or `OnResultExecutionAsync` with `await next()`.
- Set response headers ONLY in `OnResultExecuting` — too late by `OnResultExecuted`.
- Short-circuit with `context.Result = ...` AND `context.Cancel = true` together.
- Typical uses: response headers, envelope wrapping, output caching, post-response auditing (via `IAsyncResultFilter`).

---

✅ **J.Filters chapter complete** (01–05). Next up: **K.Web_API**.
