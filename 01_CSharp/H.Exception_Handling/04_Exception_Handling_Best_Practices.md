# Exception Handling Best Practices in C#

## 📌 What is it?

A consolidated set of principles for handling exceptions in a way that keeps applications robust, debuggable, and maintainable — going beyond just "wrap it in try-catch."

## 🤔 Why do we need it?

Poor exception handling is one of the most common sources of production incidents: swallowed errors, lost stack traces, misleading logs, and security leaks through overly detailed error messages. Following consistent practices avoids these pitfalls at scale.

## 🧠 Intuition

Exceptions should be **loud in logs, quiet to users**. Internally you want maximum diagnostic detail; externally (API responses, UI) you want safe, minimal, actionable messages.

## 🌍 Real-world analogy

Like an airplane's black box vs. the pilot's announcement to passengers: the black box (logs) captures *everything* in forensic detail; the passenger announcement (user-facing message) stays calm, brief, and non-technical ("We're experiencing a delay") — no need to say "hydraulic actuator #3 failed."

## ⚙️ Core Principles

| Principle                                           | What it means                                                                              |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Catch specific, not generic**               | Only catch exception types you can meaningfully act on                                     |
| **Fail fast**                                 | Validate early (guard clauses) instead of letting bad state propagate deep before throwing |
| **Don't swallow exceptions**                  | Never leave a`catch` block empty — always log or rethrow                                |
| **Preserve stack trace**                      | Use`throw;` not `throw ex;` when rethrowing                                            |
| **Use exceptions for exceptional cases only** | Not for expected control flow (e.g., validation)                                           |
| **Centralize cross-cutting handling**         | Use global handlers/middleware instead of repeating try-catch everywhere                   |
| **Don't leak internals to users**             | Never expose stack traces, SQL, or connection strings in API responses                     |
| **Log with context**                          | Include correlation IDs, user IDs, input parameters (redacted if sensitive)                |

## 💻 Code Examples

### ❌ Anti-pattern — swallowing exceptions

```csharp
try
{
    SaveToDatabase(order);
}
catch (Exception)
{
    // silently ignored — order is lost, nobody knows!
}
```

### ✅ Better — log and decide deliberately

```csharp
try
{
    SaveToDatabase(order);
}
catch (DbUpdateException ex)
{
    _logger.LogError(ex, "Failed to save order {OrderId}", order.Id);
    throw; // let it propagate — caller/global handler decides response
}
```

### Guard clauses — fail fast instead of deep exceptions

```csharp
public void ProcessOrder(Order order)
{
    ArgumentNullException.ThrowIfNull(order);
    if (order.Items.Count == 0)
        throw new InvalidOperationException("Order must have at least one item.");

    // main logic — no need to defensively check order everywhere below
}
```

### Wrapping low-level exceptions with context (exception chaining)

```csharp
try
{
    ParsePaymentFile(path);
}
catch (IOException ex)
{
    // wrap with domain context while preserving the original as InnerException
    throw new PaymentImportException($"Failed to import payment file: {path}", ex);
}
```

### Validation via Result pattern instead of exceptions (hot path)

```csharp
public Result<Order> ValidateOrder(Order order)
{
    if (order.Items.Count == 0)
        return Result<Order>.Fail("Order must contain at least one item.");

    return Result<Order>.Success(order);
}
// Avoids throwing exceptions for expected, frequent validation failures.
```

## ⚡ Performance considerations

- Avoid throwing exceptions in tight loops or high-frequency paths — the stack-trace capture cost adds up fast.
- Guard clauses and `TryX` patterns are far cheaper than catching exceptions for expected failure cases.
- Excessive nested try-catch blocks can obscure control flow and make JIT optimization harder to reason about (though this is a minor concern vs. the throw cost itself).

## 🚨 Common mistakes

- ❌ Empty `catch` blocks — the silent killer of debuggability.
- ❌ Catching `Exception` deep in business logic instead of at a boundary.
- ❌ Returning generic `500` errors with full stack traces exposed to API clients.
- ❌ Using exceptions to signal expected outcomes (e.g., "record not found" via exception instead of nullable return).
- ❌ Re-throwing with `throw ex;`, losing the original stack trace.
- ❌ Not including contextual data in logs (just `ex.Message` instead of `ex.ToString()` + relevant IDs).

## 💡 Best practices

- Validate inputs early with guard clauses — fail before doing expensive work.
- Wrap and rethrow with added context only when it adds diagnostic value (`PaymentImportException` wrapping `IOException`).
- Use a global exception handler (see `05_Global_Exception_Handling.md`) for cross-cutting concerns like logging and generic user-facing responses.
- Log exceptions once, at the point where you decide **not** to rethrow — avoid duplicate logs at every layer.
- Return structured error responses (e.g., `ProblemDetails` in ASP.NET Core) instead of raw exception messages.
- Consider `Result<T>`/`OneOf<T>` patterns for expected failure paths — reserve real exceptions for truly exceptional situations.

## 🎤 Interview Questions

1. **Why is an empty `catch` block considered dangerous?**
   → It silently discards errors, making failures invisible until they cause much bigger downstream problems.
2. **What's the difference between "fail fast" and "defensive programming everywhere"?**
   → Fail fast validates at the boundary/entry point once; defensive programming rechecks the same conditions repeatedly throughout the call chain, which is redundant and harder to maintain.
3. **When should you wrap an exception in a custom one vs. let it propagate as-is?**
   → Wrap when the original exception lacks domain context that would help diagnose/handle it; otherwise let it propagate unchanged to avoid noise.
4. **Why avoid exposing raw exception details in API responses?**
   → Security risk (leaks internal structure, stack traces, connection info) and poor UX — use sanitized, structured error responses instead.
5. **What's the "log once" principle?**
   → Log an exception at the point where you decide not to rethrow it further; logging it at every layer as it propagates creates duplicate, noisy logs.

## 📝 30-second Revision Cheat Sheet

| Concept     | Key Point                                                    |
| ----------- | ------------------------------------------------------------ |
| Never       | Empty catch blocks                                           |
| Always      | `throw;` not `throw ex;` when rethrowing                 |
| Validate    | Fail fast with guard clauses at entry points                 |
| Hot paths   | Use Result/TryX patterns, not exceptions                     |
| User-facing | Sanitized message only; log full detail internally           |
| Centralize  | Global handler for cross-cutting logging/response formatting |
