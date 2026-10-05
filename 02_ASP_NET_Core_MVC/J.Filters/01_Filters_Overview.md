# 01_Filters_Overview

> **Filters** = MVC-specific hooks that run at particular points in the Controller/Action execution pipeline — unlike Middleware (`I.Middleware_and_Filters`), which runs at the raw HTTP level with NO awareness of Controllers, Actions, or ModelState.

> New chapter: **J.Filters**. Directly answers the question `01_Middleware_Concepts.md` raised: "when do I need MVC-aware logic instead of generic middleware?" — this is that layer.

## 📌 What is it?

Filters plug into the MVC pipeline at five specific stages, each with its own purpose:

```
Middleware pipeline (I.Middleware_and_Filters)
        │
        ▼
   [Routing selects a Controller/Action]
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│  MVC FILTER PIPELINE (this chapter)                        │
│                                                              │
│  1. Authorization Filters   → "Is this caller allowed here?" │
│  2. Resource Filters         → wraps EVERYTHING below         │
│  3. Action Filters           → before/after the action runs   │
│  4. Exception Filters        → catches exceptions FROM the action │
│  5. Result Filters           → before/after the result executes   │
└─────────────────────────────────────────────────────────┘
        │
        ▼
   Response sent back through Middleware pipeline
```

## 🤔 Why do we need filters, given middleware already exists?

| Need                                                            | Middleware                                | Filters                                                          |
| --------------------------------------------------------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| Log every single HTTP request, generically                      | ✅ Perfect fit                            | Overkill — no MVC context needed                                |
| Run logic only for SPECIFIC controllers/actions                 | ❌ Middleware applies globally by default | ✅ Apply a filter to just one Controller/Action via an attribute |
| Access`ModelState`, action arguments, or the `ActionResult` | ❌ Middleware has no concept of these     | ✅ Filters ARE MVC-aware — this is their whole point            |
| Wrap exception handling with knowledge of WHICH action threw    | ❌ Middleware only sees a raw exception   | ✅ Exception filters know the Controller/Action context          |

## 🌍 Real-world analogy

If Middleware is the **airport-wide security process** every single passenger goes through (`I.Middleware_and_Filters`'s analogy), Filters are like **gate-specific checks** that only apply to passengers boarding a SPECIFIC flight — a particular gate might require an extra document check (Authorization Filter) or have staff specifically monitoring boarding order (Action Filter) — checks that only make sense in the context of THIS particular flight, not every passenger in the whole airport.

## 📊 The Five Filter Types — Full Reference

| Filter Type                     | Runs                                                                                                         | Typical use                                                                                    | Detailed in                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- | ---------------------------------------- |
| **Authorization Filters** | FIRST — before anything else in the MVC pipeline                                                            | `[Authorize]` — deciding if the caller can access this action AT ALL                        | `L.Security`                           |
| **Resource Filters**      | Right after Authorization, wraps everything below (including model binding)                                  | Caching a full response before model binding even happens; short-circuiting expensive work     | (advanced — less commonly custom-built) |
| **Action Filters**        | Immediately before AND after the action method executes                                                      | Logging action parameters, modifying arguments, measuring execution time, modifying the result | `02_Action_Filters.md`                 |
| **Exception Filters**     | ONLY if an unhandled exception occurs inside the action or a later filter                                    | MVC-aware error handling/logging — knows which Controller/Action threw                        | `03_Exception_Filters.md`              |
| **Result Filters**        | Immediately before AND after the ActionResult executes (i.e., before/after the response is actually written) | Modifying response headers, wrapping/formatting the final output                               | `05_Result_Filters.md`                 |

## ⚙️ Internal working — the filter pipeline in detail

```
Request reaches MVC (after Middleware, after Routing selects an action)
        │
        ▼
1. AUTHORIZATION FILTERS run
   → if UNAUTHORIZED, short-circuits HERE — action never runs at all
        │
        ▼
2. RESOURCE FILTERS run (before model binding)
        │
        ▼
   [Model Binding happens — request data → action parameters]
        │
        ▼
3. ACTION FILTERS run (OnActionExecuting)
        │
        ▼
   [THE ACTION METHOD ITSELF RUNS]
        │
        ├── If an EXCEPTION is thrown here ──► 4. EXCEPTION FILTERS run
        │                                          (can suppress the exception and provide a result)
        ▼
3. ACTION FILTERS run again (OnActionExecuted)
        │
        ▼
5. RESULT FILTERS run (OnResultExecuting)
        │
        ▼
   [The ActionResult actually EXECUTES — e.g., writes JSON to the response]
        │
        ▼
5. RESULT FILTERS run again (OnResultExecuted)
        │
        ▼
   [Response continues back OUT through the Middleware pipeline]
```

## 📊 Filter Scopes — where you can apply them

| Scope            | How                                                                                 | Applies to                          |
| ---------------- | ----------------------------------------------------------------------------------- | ----------------------------------- |
| Global           | Registered in`Program.cs`/`AddControllers(options => options.Filters.Add(...))` | EVERY controller/action in the app  |
| Controller-level | `[MyFilter]` attribute on the Controller class                                    | Every action within that Controller |
| Action-level     | `[MyFilter]` attribute on a specific action method                                | Just that one action                |

```csharp
[MyFilter] // applies to ALL actions in this controller
public class ProductsController : ControllerBase
{
    [MyOtherFilter] // applies ONLY to this specific action
    [HttpGet("{id}")]
    public IActionResult GetById(int id) { ... }
}
```

## 💻 Code examples

### Basic — registering a filter globally

```csharp
// Program.cs
builder.Services.AddControllers(options =>
{
    options.Filters.Add<RequestTimingActionFilter>(); // applies to EVERY controller/action
});
```

### Intermediate — applying a filter at the Controller vs Action level

```csharp
[ServiceFilter(typeof(AuditLogFilter))]  // applies to ALL actions in this controller
public class OrdersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(_service.GetAllOrders());

    [ValidateAntiForgeryToken]  // applies ONLY to this action (see L.Security/13_Anti_Forgery_Tokens_CSRF.md)
    [HttpPost]
    public IActionResult Create(OrderViewModel model) => Ok(_service.CreateOrder(model));
}
```

### Practical — recognizing filters you already use every day

```csharp
[ApiController]                          // itself enables several built-in filter-like behaviors
[Authorize]                               // an AUTHORIZATION filter
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    [ValidateAntiForgeryToken]            // an AUTHORIZATION-adjacent filter (CSRF protection)
    public IActionResult Create(ProductViewModel model)
    {
        if (!ModelState.IsValid)          // NOTE: [ApiController] itself triggers automatic
            return BadRequest(ModelState); // model validation via a built-in filter — this check
                                            // is often even done FOR you automatically with [ApiController]!
        return Ok(_service.CreateProduct(model));
    }
}
```

## ⚡ Performance considerations

- Global filters run for EVERY controller/action in the app — same discipline as middleware: keep global filter logic lightweight.
- Filters scoped to specific controllers/actions (via attributes) only add overhead where actually applied — prefer this scoping over a global filter when the logic genuinely doesn't apply everywhere.
- Filters are somewhat more expensive to invoke than raw middleware (there's more MVC machinery involved) — for truly generic, framework-level concerns with NO need for MVC context, middleware remains the more efficient choice.

## 🚨 Common mistakes

- ❌ Reaching for a Filter when Middleware would do — if the logic has no need for Controller/Action/ModelState awareness, Middleware is simpler and more efficient.
- ❌ Reaching for Middleware when a Filter is what's actually needed — e.g., trying to inspect `ModelState` or action arguments from middleware, which has no concept of them at all.
- ❌ Applying an expensive filter globally when it's only relevant to a handful of specific actions — unnecessarily slows down every request in the app.
- ❌ Confusing Action Filters with Exception Filters — an Action Filter's `OnActionExecuted` does NOT reliably catch exceptions the way a dedicated Exception Filter does (see `03_Exception_Filters.md`).

## 💡 Best practices

- ✅ Choose Middleware for framework-wide, HTTP-level concerns; choose Filters for logic that specifically needs MVC context (Controller, Action, ModelState, the ActionResult).
- ✅ Scope filters as narrowly as makes sense — action-level or controller-level over global, unless the logic truly applies everywhere.
- ✅ Learn to recognize filters you're ALREADY using — `[Authorize]`, `[ValidateAntiForgeryToken]`, and `[ApiController]`'s automatic model validation are all filter-based mechanisms under the hood.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                        | Answer                                                                                                                                                     |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the key difference between Middleware and Filters?                       | Middleware operates at the raw HTTP level with no MVC awareness; Filters run within the MVC pipeline and ARE aware of Controllers, Actions, and ModelState |
| Name the five types of filters, in execution order.                             | Authorization, Resource, Action, Exception (only on error), Result                                                                                         |
| Which filter type runs FIRST, and what can it do?                               | Authorization filters — they can short-circuit the pipeline entirely if the caller isn't allowed to proceed                                               |
| At what three scopes can a filter be applied?                                   | Global (all controllers/actions), Controller-level (all actions in that controller), Action-level (a single action)                                        |
| Give an example of a filter you're probably already using without realizing it. | `[Authorize]` (an authorization filter) or `[ApiController]`'s automatic model validation                                                              |

## 📝 30-second Revision Cheat Sheet

- Filters = MVC-aware hooks into the Controller/Action pipeline; unlike Middleware, they know about ModelState, action arguments, and results.
- Five types, in order: Authorization → Resource → Action → Exception (on error) → Result.
- Scoped globally, per-controller, or per-action — prefer the narrowest scope that fits.
- Choose Middleware for generic HTTP concerns; choose Filters when MVC context is genuinely needed.
- `[Authorize]`, `[ValidateAntiForgeryToken]`, and `[ApiController]`'s auto-validation are all filters you likely already use.
