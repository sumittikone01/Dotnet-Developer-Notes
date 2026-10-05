# 02_Action_Filters

> **Action Filters** = filters that run immediately BEFORE and AFTER an action method executes — giving you a hook to inspect/modify the action's arguments beforehand, and inspect/modify its result afterward.

> Continues **J.Filters**, drilling into the middle stage of `01_Filters_Overview.md`'s five-stage pipeline: Authorization → Resource → **Action** → Exception → Result.

## 📌 What is it?

```csharp
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Runs BEFORE the action method executes
        Console.WriteLine($"About to run: {context.ActionDescriptor.DisplayName}");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // Runs AFTER the action method executes
        Console.WriteLine($"Finished running: {context.ActionDescriptor.DisplayName}");
    }
}
```

## 🤔 Why do we need them?

| Need                                                                     | How an Action Filter helps                                                         |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| Log every action's inputs/outputs consistently                           | One filter, applied everywhere, instead of repeating logging code in every action  |
| Measure how long every action takes                                      | Time between`OnActionExecuting` and `OnActionExecuted`                         |
| Short-circuit an action before it even runs (e.g., a feature flag check) | Set`context.Result` in `OnActionExecuting` — the action method NEVER executes |
| Modify or inspect the action's RESULT before it's returned to the client | `OnActionExecuted` can inspect/replace `context.Result`                        |

## 🌍 Real-world analogy

A **theater usher checking your ticket right before you enter (`OnActionExecuting`), and checking on you again right as you leave (`OnActionExecuted`)** — both checks happen around the actual "show" (the action method), without the usher ever needing to know what happens IN the show itself.

## ⚙️ Internal working — Sync vs Async Action Filters

| Interface              | Methods                                     | When to use                                                                             |
| ---------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- |
| `IActionFilter`      | `OnActionExecuting`, `OnActionExecuted` | Simple, synchronous logic                                                               |
| `IAsyncActionFilter` | `OnActionExecutionAsync(context, next)`   | When you need`await` inside the filter, or want more explicit control over the "wrap" |

```csharp
public class TimingAsyncActionFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();

        var executedContext = await next(); // this line RUNS THE ACTION METHOD (and any later filters)

        sw.Stop();
        Console.WriteLine($"{context.ActionDescriptor.DisplayName} took {sw.ElapsedMilliseconds}ms");
    }
}
```

```
IAsyncActionFilter's single method WRAPS the entire action execution:

OnActionExecutionAsync {
    ... code here runs BEFORE the action (like OnActionExecuting) ...

    await next();   ◄── THIS is where the action method (and inner filters) actually run

    ... code here runs AFTER the action (like OnActionExecuted) ...
}
```

## 🖼 Short-circuiting an action — a genuinely powerful capability

```csharp
public class MaintenanceModeFilter : IActionFilter
{
    private readonly IConfiguration _config;
    public MaintenanceModeFilter(IConfiguration config) => _config = config;

    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (_config.GetValue<bool>("MaintenanceMode"))
        {
            // Setting context.Result SHORT-CIRCUITS the pipeline —
            // the actual action method NEVER runs at all!
            context.Result = new ObjectResult(new { message = "Site under maintenance" })
            {
                StatusCode = 503
            };
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

```
Normal flow:            OnActionExecuting → [Action Method Runs] → OnActionExecuted

Short-circuited flow:   OnActionExecuting (sets context.Result) → [Action Method SKIPPED] → OnActionExecuted
```

## 📊 Accessing and modifying action ARGUMENTS

```csharp
public class TrimStringsFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // context.ActionArguments is a Dictionary<string, object> of the action's parameters
        foreach (var key in context.ActionArguments.Keys.ToList())
        {
            if (context.ActionArguments[key] is string str)
            {
                context.ActionArguments[key] = str.Trim(); // MODIFIES the argument BEFORE the action sees it
            }
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

## 💻 Code examples

### Basic — a simple logging Action Filter, applied via attribute

```csharp
public class LogExecutionTimeAttribute : ActionFilterAttribute // convenient base class — implement only what you need
{
    private Stopwatch _stopwatch = new();

    public override void OnActionExecuting(ActionExecutingContext context)
    {
        _stopwatch.Restart();
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        _stopwatch.Stop();
        Console.WriteLine($"{context.ActionDescriptor.DisplayName} took {_stopwatch.ElapsedMilliseconds}ms");
    }
}

[LogExecutionTime] // apply directly as an attribute — ActionFilterAttribute makes this possible
[HttpGet("{id}")]
public IActionResult GetById(int id) => Ok(_service.GetProductById(id));
```

### Intermediate — a filter with injected dependencies via `ServiceFilter`

```csharp
public class AuditActionFilter : IAsyncActionFilter
{
    private readonly AuditService _auditService; // Scoped service — safe here since filters are
                                                    // instantiated PER-REQUEST when using ServiceFilter
    public AuditActionFilter(AuditService auditService) => _auditService = auditService;

    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var result = await next(); // run the action

        if (result.Exception == null) // only audit successful actions
        {
            await _auditService.LogActionAsync(context.ActionDescriptor.DisplayName!);
        }
    }
}

// Register as a service so DI can construct it
builder.Services.AddScoped<AuditActionFilter>();
```

```csharp
[ServiceFilter(typeof(AuditActionFilter))] // resolves via DI — Scoped lifetime works correctly here
[HttpPost]
public IActionResult CreateProduct(ProductViewModel model) => Ok(_service.CreateProduct(model));
```

### Practical — modifying the RESULT after the action runs

```csharp
public class WrapResponseFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.Result is ObjectResult objectResult && objectResult.StatusCode is >= 200 and < 300)
        {
            // Wrap every successful response in a consistent envelope
            objectResult.Value = new { success = true, data = objectResult.Value };
        }
    }
}
```

## ⚡ Performance considerations

- Action Filters applied GLOBALLY run for every single action in the app — keep global filter logic minimal; scope filters to specific controllers/actions when the logic doesn't universally apply.
- `IAsyncActionFilter` is preferred over `IActionFilter` when the filter needs to `await` anything (a DB call, an external service) — avoid blocking (`.Result`/`.Wait()`) inside a synchronous `IActionFilter`, which can cause thread-pool starvation under load.
- `ServiceFilter` resolves the filter instance through DI per-request (respecting Scoped lifetimes correctly) — prefer it over a plain attribute-based filter when the filter needs Scoped dependencies.

## 🚨 Common mistakes

- ❌ Using a synchronous `IActionFilter` and blocking on async code inside it (`.Result`, `.Wait()`) — can cause deadlocks/thread-pool exhaustion; use `IAsyncActionFilter` instead when async work is needed.
- ❌ Forgetting that setting `context.Result` in `OnActionExecuting` completely skips the action method — a source of "why isn't my action running?!" confusion if done unintentionally.
- ❌ Directly instantiating a filter with Scoped dependencies via a plain attribute (which uses the DI container differently) instead of `[ServiceFilter(typeof(...))]` — can lead to incorrect lifetimes.
- ❌ Putting expensive, per-request logic (heavy DB queries, external API calls) inside a GLOBALLY-applied filter, silently slowing down every single action in the app.

## 💡 Best practices

- ✅ Use `ActionFilterAttribute` as a convenient base class when you want to apply the filter directly as a `[MyFilter]` attribute without extra registration.
- ✅ Use `[ServiceFilter(typeof(MyFilter))]` (with the filter registered in DI) when the filter needs Scoped dependencies like a DAL/service class.
- ✅ Prefer `IAsyncActionFilter` whenever the filter's logic needs to `await` anything.
- ✅ Reserve `context.Result` short-circuiting for deliberate, well-documented cases (maintenance mode, feature flags) — it's powerful but can be confusing if used carelessly.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                     | Answer                                                                                                                     |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| What are the two methods on`IActionFilter`?                                | `OnActionExecuting` (runs before the action) and `OnActionExecuted` (runs after the action)                            |
| How do you short-circuit an action from executing entirely?                  | Set`context.Result` inside `OnActionExecuting` (or before calling `next()` in `IAsyncActionFilter`)                |
| When should you use`IAsyncActionFilter` instead of `IActionFilter`?      | When the filter's logic needs to`await` something (a DB call, an external API)                                           |
| How do you access an action's parameters inside a filter?                    | Via`context.ActionArguments`, a dictionary of parameter name to value, in `OnActionExecuting`                          |
| Why use`[ServiceFilter(typeof(MyFilter))]` instead of just `[MyFilter]`? | It resolves the filter through the DI container, correctly respecting Scoped/Transient lifetimes for injected dependencies |

## 📝 30-second Revision Cheat Sheet

- Action Filters wrap the action method: `OnActionExecuting` (before) and `OnActionExecuted` (after) — or a single `OnActionExecutionAsync` with `await next()` in between.
- Setting `context.Result` in `OnActionExecuting` SKIPS the action method entirely.
- `context.ActionArguments` lets you inspect/modify action parameters before the action runs.
- Use `IAsyncActionFilter` for anything needing `await`; use `[ServiceFilter(typeof(...))]` for filters needing Scoped DI dependencies.
- `ActionFilterAttribute` is a convenient base class for simple, attribute-applied filters.
