
# 04_Filter_Execution_Order

> When MULTIPLE filters of the SAME type apply to one action (global + controller-level + action-level, or several attributes stacked), this chapter is the precise reference for exactly what order they run in — and how to override that order when needed.

> Continues **J.Filters**. `01_Filters_Overview.md` established the order BETWEEN filter TYPES (Authorization → Resource → Action → Exception → Result); this chapter covers order WITHIN the same type, when several filters of that type are stacked together.

## 📌 What is it?

It's common to have MORE than one filter of the same kind apply to a single action at once — a global logging filter, a controller-level auditing filter, AND an action-level validation filter, all Action Filters. This chapter answers: **in what order do they run, and can I control it?**

## 🤔 Why does this matter?

| Scenario                                                                                               | Why order matters                                                                   |
| ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| A global "wrap response" filter AND an action-specific "add extra field" filter both modify the result | Whichever runs LAST determines the FINAL shape of the response                      |
| An authorization filter that sets user context, and another that relies on that context                | The dependent filter must run AFTER the one that sets up its data                   |
| A global exception filter and a more specific, controller-level one                                    | You usually want the MORE SPECIFIC one to get first crack at handling the exception |

## 📊 The Default Scope-Based Order

```
For the "before" phase (OnActionExecuting, OnResultExecuting, etc.):
   Global filters  →  Controller-level filters  →  Action-level filters
   (OUTSIDE-IN — broadest scope runs FIRST)

For the "after" phase (OnActionExecuted, OnResultExecuted, etc.):
   Action-level filters  →  Controller-level filters  →  Global filters
   (INSIDE-OUT — narrowest scope runs FIRST, mirrors the "before" order in reverse)
```

```
┌─────────────────────────────────────────────────────────────┐
│  GLOBAL filter: OnActionExecuting                              │
│  ┌───────────────────────────────────────────────────────┐    │
│  │  CONTROLLER-level filter: OnActionExecuting              │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │  ACTION-level filter: OnActionExecuting            │  │    │
│  │  │                                                     │  │    │
│  │  │           [ACTION METHOD RUNS]                      │  │    │
│  │  │                                                     │  │    │
│  │  │  ACTION-level filter: OnActionExecuted             │  │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  │  CONTROLLER-level filter: OnActionExecuted               │    │
│  └───────────────────────────────────────────────────────┘    │
│  GLOBAL filter: OnActionExecuted                                │
└─────────────────────────────────────────────────────────────┘
```

> This "outside-in, then inside-out" nesting is exactly like Russian nesting dolls (matryoshka) — the OUTERMOST doll (Global) opens FIRST, and closes LAST; the INNERMOST doll (Action-level) opens LAST, and closes FIRST.

## 📊 Same-Scope Ordering — the `Order` property

When multiple filters share the SAME scope (e.g., two global filters, or two attributes on the same action), you control their relative order explicitly with the `Order` property.

```csharp
public class FirstFilter : ActionFilterAttribute
{
    public FirstFilter() => Order = 1; // LOWER Order runs FIRST (in the "before" phase)
}

public class SecondFilter : ActionFilterAttribute
{
    public SecondFilter() => Order = 2;
}

[FirstFilter]   // Order = 1
[SecondFilter]  // Order = 2
[HttpGet]
public IActionResult GetProducts() => Ok(_service.GetAllProducts());

// Execution: FirstFilter.OnActionExecuting → SecondFilter.OnActionExecuting →
//            [Action Runs] →
//            SecondFilter.OnActionExecuted → FirstFilter.OnActionExecuted
```

```
Order property rule:

"Before" phase (Executing):   LOWER Order number runs FIRST
"After" phase (Executed):     LOWER Order number runs LAST (mirrors "before," in reverse)

Default Order (if not explicitly set) = 0 for most filters
```

## 🖼 Overriding the default scope-based order with `Order`

```csharp
// Without explicit Order: Global runs before Controller-level, before Action-level (default)

// WITH explicit Order, you can make an ACTION-level filter run BEFORE a GLOBAL one:
public class MyGlobalFilter : ActionFilterAttribute
{
    public MyGlobalFilter() => Order = 10; // a HIGHER number than the action-level filter below
}

public class MyActionFilter : ActionFilterAttribute
{
    public MyActionFilter() => Order = -10; // a LOWER number — runs FIRST despite being action-scoped!
}
```

> **`Order` OVERRIDES the default scope-based ordering entirely** when explicitly set — this is powerful, but can also make execution order genuinely confusing if used inconsistently across a codebase. Use it deliberately and document WHY when you do.

## 📊 Cross-Type Order (recap, tying it together)

```
Authorization Filters  →  Resource Filters  →  Action Filters  →  [Exception Filters, if error]  →  Result Filters

Within EACH of these types, the scope-based (Global → Controller → Action) or explicit Order rules above apply.
```

## 💻 Code examples

### Basic — demonstrating default scope-based order

```csharp
// Program.cs
builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalLoggingFilter>(); // GLOBAL scope
});

[ControllerAuditFilter]   // CONTROLLER scope
public class ProductsController : ControllerBase
{
    [ActionValidationFilter] // ACTION scope
    [HttpPost]
    public IActionResult Create(ProductViewModel model) => Ok(_service.CreateProduct(model));
}

// Execution order for OnActionExecuting:
//   GlobalLoggingFilter → ControllerAuditFilter → ActionValidationFilter → [Action Runs]
// Execution order for OnActionExecuted (reversed):
//   [Action Runs] → ActionValidationFilter → ControllerAuditFilter → GlobalLoggingFilter
```

### Intermediate — explicitly controlling order with the `Order` property

```csharp
public class ValidateInputFilter : ActionFilterAttribute
{
    public ValidateInputFilter() => Order = 1; // wants to run BEFORE logging
}

public class LogRequestFilter : ActionFilterAttribute
{
    public LogRequestFilter() => Order = 2; // runs AFTER validation — only logs VALID requests
}

[ValidateInputFilter] // Order = 1 — runs FIRST
[LogRequestFilter]    // Order = 2 — runs SECOND
[HttpPost]
public IActionResult Create(ProductViewModel model) => Ok(_service.CreateProduct(model));
```

### Practical — a real-world case where order genuinely matters: Exception Filters

```csharp
public class SpecificDomainExceptionFilter : IExceptionFilter
{
    public int Order => 1; // runs FIRST — gets first chance to handle KNOWN exception types

    public void OnException(ExceptionContext context)
    {
        if (context.Exception is ProductNotFoundException)
        {
            context.Result = new NotFoundObjectResult(new { error = context.Exception.Message });
            context.ExceptionHandled = true;
        }
        // otherwise, leaves ExceptionHandled = false, letting it fall through
    }
}

public class GenericFallbackExceptionFilter : IExceptionFilter
{
    public int Order => 100; // runs LAST — the final, catch-all safety net

    public void OnException(ExceptionContext context)
    {
        if (!context.ExceptionHandled)
        {
            context.Result = new ObjectResult(new { error = "An unexpected error occurred." }) { StatusCode = 500 };
            context.ExceptionHandled = true;
        }
    }
}
```

## ⚡ Performance considerations

- Filter execution order has no meaningful performance cost on its own — the concern here is CORRECTNESS (getting the right behavior), not speed.
- Be mindful that a HIGH-Order (late-running) filter that does expensive work still runs AFTER every earlier filter has already executed — if a filter can determine "this request should be rejected" cheaply, giving it a LOW Order (running early) avoids wasted work in later, more expensive filters.

## 🚨 Common mistakes

- ❌ Assuming filters run in the ORDER THEY'RE WRITTEN in the source file (as attributes) rather than by scope/Order — attribute declaration order does NOT reliably determine execution order; explicit `Order` does.
- ❌ Two filters both assuming they'll run "first" without an explicit `Order` set — leads to fragile, hard-to-predict behavior that can change with framework updates or refactoring.
- ❌ Setting `Order` inconsistently across a codebase (some filters use it, most don't) — makes the overall execution order confusing and hard to reason about for the whole team.
- ❌ Forgetting that "before" and "after" phases run in OPPOSITE relative order (outside-in, then inside-out) — assuming the "after" methods run in the same order as "before."

## 💡 Best practices

- ✅ Rely on the default scope-based order (Global → Controller → Action) when it naturally matches your intent — it's predictable and requires no extra code.
- ✅ Use the explicit `Order` property only when you have a genuine, specific reason two same-scope filters need a particular relative order — and comment WHY.
- ✅ For Exception Filters specifically, give more specific/domain-aware filters a LOWER `Order` (run first) and generic fallback filters a HIGHER `Order` (run last).
- ✅ Document any non-default ordering clearly, since it's one of the harder things for a new team member to intuit by reading the code alone.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                        | Answer                                                                                                                              |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| What's the default execution order for filters of the same type, by scope?                      | Global → Controller-level → Action-level, for the "before" phase; reversed (Action → Controller → Global) for the "after" phase |
| How do you control execution order between filters of the SAME scope?                           | Set the`Order` property explicitly — lower values run first in the "before" phase                                                |
| Does the order filters are written as attributes in the source code determine execution order?  | No — scope and the explicit`Order` property determine execution order, not source code declaration order                         |
| Why might you give a specific Exception Filter a LOWER`Order` than a generic fallback filter? | So the specific filter gets first chance to handle known exception types, leaving the generic filter as the final catch-all         |
| Does the "before" phase and "after" phase run filters in the same relative order?               | No — they're reversed: outside-in for "before," inside-out for "after," like nested dolls opening and closing                      |

## 📝 30-second Revision Cheat Sheet

- Default order (same filter type): Global → Controller → Action for "before" phases; reversed for "after" phases (nested-doll pattern).
- The `Order` property overrides scope-based ordering — lower values run first in "before" phases.
- Attribute declaration order in source code does NOT determine execution order — scope/`Order` does.
- For Exception Filters, give specific handlers a lower `Order` and generic fallbacks a higher `Order`.
- Use explicit `Order` deliberately and document it — inconsistent use makes execution order hard to reason about.
