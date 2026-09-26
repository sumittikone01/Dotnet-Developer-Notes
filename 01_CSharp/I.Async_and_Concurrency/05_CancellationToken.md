# CancellationToken in C#

## 📌 What is it?

`CancellationToken` is .NET's standard, cooperative mechanism for **requesting cancellation** of an ongoing operation (async or long-running sync). It doesn't forcibly kill anything — it's a signal that the operation itself must check and honor.

## 🤔 Why do we need it?

- Long-running operations (API calls, DB queries, file processing) need a clean way to be stopped early — e.g., user navigates away, request times out, app is shutting down.
- Forcibly killing threads (`Thread.Abort`) is dangerous and removed in modern .NET — cooperative cancellation is the safe, standard alternative.
- Enables timeouts, user-initiated cancel buttons, and graceful shutdown across an entire async call chain.

## 🧠 Intuition

It's like a **shared "stop" flag** passed down through a chain of method calls. Nobody is forced to stop — each method periodically checks "has someone asked me to stop?" and, if so, exits cleanly (usually by throwing `OperationCanceledException`).

## 🌍 Real-world analogy

A **relay race with a whistle**: the coach (caller) can blow a whistle (`Cancel()`) at any time. Each runner (method in the call chain) is trained to check for the whistle at checkpoints and stop running cleanly if they hear it — rather than being tackled to the ground mid-stride (which is what `Thread.Abort` effectively does — dangerous and messy).

## ⚙️ Internal working

1. A `CancellationTokenSource` (CTS) is the "controller" — it creates a `CancellationToken` that gets passed around.
2. Calling `cts.Cancel()` flips the token's internal state to canceled.
3. Code holding the token can either:
   - Poll: `token.IsCancellationRequested` (check periodically in loops).
   - Throw: `token.ThrowIfCancellationRequested()` (throws `OperationCanceledException` if canceled).
   - Register a callback: `token.Register(() => { ... })` (runs when cancellation happens).
4. Many built-in async APIs (`HttpClient.GetAsync`, `Task.Delay`, EF Core queries) accept a `CancellationToken` parameter and honor it internally — cancellation propagates automatically through the whole call chain if you pass the token consistently.
5. `CancellationTokenSource` can also auto-cancel after a timeout: `new CancellationTokenSource(TimeSpan.FromSeconds(5))`.

## 🖼 Diagram — Cancellation Flow

```
CancellationTokenSource (cts)
        │
        │ cts.Token passed down through call chain
        ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  MethodA()  │────▶│  MethodB()  │────▶│  MethodC()  │
│ (token)     │     │ (token)     │     │ (token)     │
└─────────────┘     └─────────────┘     └─────┬───────┘
                                                │ checks token periodically
                                                ▼
                                    token.ThrowIfCancellationRequested()
                                                │
                cts.Cancel() called ───────────┘ (from anywhere holding cts)
                                                │
                                                ▼
                                   OperationCanceledException thrown,
                                   propagates up through await chain
```

## 💻 Code Examples

### Basic — manual cancellation

```csharp
var cts = new CancellationTokenSource();

Task task = Task.Run(async () =>
{
    for (int i = 0; i < 10; i++)
    {
        cts.Token.ThrowIfCancellationRequested();
        Console.WriteLine($"Working... {i}");
        await Task.Delay(500);
    }
}, cts.Token);

// Elsewhere — e.g., user clicks "Cancel"
cts.Cancel();

try
{
    await task;
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operation was canceled.");
}
```

### Intermediate — timeout-based cancellation

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5)); // auto-cancels after 5s

try
{
    HttpResponseMessage response = await httpClient.GetAsync(url, cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Request timed out.");
}
```

### Practical — ASP.NET Core controller honoring client disconnect

```csharp
[HttpGet]
public async Task<IActionResult> GetReport(CancellationToken cancellationToken)
{
    // ASP.NET Core automatically supplies a token tied to the client's request lifetime.
    // If the client disconnects, this token gets canceled automatically.
    var data = await _reportService.GenerateReportAsync(cancellationToken);
    return Ok(data);
}

public async Task<Report> GenerateReportAsync(CancellationToken cancellationToken)
{
    var rawData = await _dbContext.Sales
        .ToListAsync(cancellationToken); // EF Core honors it natively

    cancellationToken.ThrowIfCancellationRequested();
    return BuildReport(rawData);
}
```

### Combining multiple cancellation sources

```csharp
using var userCts = new CancellationTokenSource();
using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
using var combined = CancellationTokenSource.CreateLinkedTokenSource(userCts.Token, timeoutCts.Token);

await DoWorkAsync(combined.Token); // cancels if EITHER source cancels
```

### Registering a cleanup callback

```csharp
cts.Token.Register(() =>
{
    Console.WriteLine("Cancellation requested — cleaning up resources.");
});
```

## 📊 Comparison Table — Cancellation Check Methods

| Method                             | Behavior                             | When to use                                               |
| ---------------------------------- | ------------------------------------ | --------------------------------------------------------- |
| `IsCancellationRequested`        | Returns`bool`, no throw            | Loops where you want a clean, non-exceptional exit        |
| `ThrowIfCancellationRequested()` | Throws`OperationCanceledException` | Standard pattern — lets exception propagate up naturally |
| `Register(callback)`             | Runs a callback on cancellation      | Cleanup, logging, releasing resources                     |

## ⚡ Performance considerations

- Checking `IsCancellationRequested` is a cheap, lock-free read — safe to call frequently in loops.
- Avoid checking cancellation on every single tiny iteration of a very tight loop (millions of iterations) — check periodically (e.g., every N iterations) to reduce overhead.
- Linked token sources (`CreateLinkedTokenSource`) have minor overhead but are the correct way to combine multiple cancellation reasons.

## 🚨 Common mistakes

- ❌ Not passing the token through the **entire** call chain — cancellation silently stops propagating at whichever method forgot it.
- ❌ Catching `OperationCanceledException` and treating it as a generic error (logging as `Error` level) instead of recognizing it as expected/intentional.
- ❌ Forgetting to dispose `CancellationTokenSource` (`using` statement) — it holds timer resources if constructed with a timeout.
- ❌ Using `Thread.Abort()` instead of cooperative cancellation — removed in modern .NET and dangerous even where available.
- ❌ Ignoring the token entirely in a long-running loop — the operation becomes uncancelable regardless of what the caller wants.

## 💡 Best practices

- Accept a `CancellationToken` parameter in any async method that could run for a meaningful duration — make it the **last** parameter by convention.
- Always pass the token down to any inner async calls that accept one (`ToListAsync(token)`, `GetAsync(url, token)`, etc.) — this is how cancellation "flows."
- Use `token.ThrowIfCancellationRequested()` at natural checkpoints in loops/long operations.
- Combine timeout + manual cancellation via `CreateLinkedTokenSource` when both are needed.
- Treat `OperationCanceledException` as an expected outcome, not an error — log at `Information`/`Debug` level, not `Error`.

## 🎤 Interview Questions

1. **What's the difference between `IsCancellationRequested` and `ThrowIfCancellationRequested()`?**
   → The former just returns a boolean for manual handling; the latter throws `OperationCanceledException` automatically, which is the idiomatic pattern for propagating cancellation up an async call chain.
2. **Is cancellation forced or cooperative in .NET?**
   → Cooperative — the operation itself must check the token and choose to stop; there's no way to forcibly kill it from outside (safely).
3. **How does ASP.NET Core provide automatic cancellation for HTTP requests?**
   → It injects a `CancellationToken` tied to the request's lifetime into controller actions; if the client disconnects, the token is automatically canceled.
4. **How do you combine a timeout with a user-triggered cancellation?**
   → `CancellationTokenSource.CreateLinkedTokenSource(userToken, timeoutToken)` — the resulting linked token cancels if either source cancels.
5. **Why shouldn't `OperationCanceledException` be logged as an error?**
   → It represents an intentional, expected outcome (user canceled, timeout hit) rather than a bug or failure — logging it as an error creates noise and false alarms.

## 📝 30-second Revision Cheat Sheet

| Concept                            | Key Point                                                      |
| ---------------------------------- | -------------------------------------------------------------- |
| `CancellationTokenSource`        | The controller — calls`.Cancel()`                           |
| `CancellationToken`              | The read-only signal passed down the call chain                |
| `ThrowIfCancellationRequested()` | Idiomatic way to react — throws`OperationCanceledException` |
| Timeout                            | `new CancellationTokenSource(TimeSpan...)`                   |
| Combine sources                    | `CreateLinkedTokenSource(...)`                               |
| Golden rule                        | Pass the token through the ENTIRE async chain                  |
