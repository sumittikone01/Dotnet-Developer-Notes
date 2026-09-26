# Async / Await in C#

## 📌 What is it?

`async`/`await` is syntactic sugar built on top of `Task`/`Task<T>` that lets you write asynchronous code that **reads like sequential code**, while the compiler transforms it under the hood into a non-blocking state machine.

- `async` — marks a method as containing asynchronous operations, enabling `await` inside it.
- `await` — suspends the method at that point until the awaited `Task` completes, **without blocking the calling thread**.

## 🤔 Why do we need it?

- Without it, async code (raw callbacks, `ContinueWith` chains) becomes deeply nested and hard to follow ("callback hell").
- Lets I/O-bound operations (DB calls, HTTP requests, file access) run without tying up a thread for the whole duration — critical for scalability (e.g., web servers handling thousands of concurrent requests with a small thread pool).
- Keeps UI apps responsive by not blocking the UI thread during long operations.

## 🧠 Intuition

`await` doesn't mean "wait and block." It means: **"pause here, release the thread back to do other work, and resume this method later (possibly on a different thread) when the awaited task completes."**

## 🌍 Real-world analogy

Ordering coffee at a busy café:

- **Synchronous**: You order, then stand at the counter doing nothing until your coffee is ready — nobody else can be served by that barista meanwhile.
- **Async/await**: You order, get given a buzzer (the `Task`), and go sit down / help other customers. When the buzzer goes off (`await` resumes), you come back and continue. The barista (thread) is free to serve others in between.

## ⚙️ Internal working

1. The compiler transforms an `async` method into a **state machine** (a generated class implementing `IAsyncStateMachine`).
2. When `await` hits an incomplete `Task`, the method **returns control to its caller immediately** — the calling thread is freed.
3. A continuation is registered on the `Task`; when it completes, execution resumes from where it left off (potentially on a different ThreadPool thread, unless `SynchronizationContext` dictates otherwise — e.g., UI thread).
4. This is why async methods are non-blocking: the thread isn't "parked" waiting — it goes back to the pool to do other work.
5. `async Task` methods propagate exceptions through the returned `Task`; `await`ing rethrows them naturally with the original stack trace.

## 🖼 Diagram — Sync vs Async Execution

```
SYNCHRONOUS (blocking):
Thread: [ Call API ]---[ BLOCKED waiting ]---[ Continue ]
         └── thread stuck doing nothing for the whole wait ──┘

ASYNCHRONOUS (await):
Thread: [ Call API ]---[ freed, does other work ]---[ Resume when ready ]
                              │
                     (thread serves other requests)
```

## 📊 Comparison Table — Sync vs Async

| Aspect                    | Synchronous                              | Async/Await                             |
| ------------------------- | ---------------------------------------- | --------------------------------------- |
| Thread during I/O wait    | Blocked, idle                            | Released, reusable                      |
| Scalability (server apps) | Poor — thread-per-request exhausts pool | High — few threads serve many requests |
| Code readability          | Simple, linear                           | Simple, linear (thanks to`await`!)    |
| UI responsiveness         | Freezes during long ops                  | Stays responsive                        |

## 💻 Code Examples

### Basic

```csharp
public async Task<string> GetUserNameAsync(int userId)
{
    HttpClient client = new HttpClient();
    string json = await client.GetStringAsync($"https://api.example.com/users/{userId}");
    return ParseName(json);
}
```

### Intermediate — sequential vs concurrent awaits

```csharp
// ❌ Sequential — 3x the wait time (each awaits before the next starts)
var user = await GetUserAsync(id);
var orders = await GetOrdersAsync(id);
var invoices = await GetInvoicesAsync(id);

// ✅ Concurrent — all three run in parallel, total time = the slowest one
var userTask = GetUserAsync(id);
var ordersTask = GetOrdersAsync(id);
var invoicesTask = GetInvoicesAsync(id);

await Task.WhenAll(userTask, ordersTask, invoicesTask);

var user = userTask.Result;      // safe — already completed
var orders = ordersTask.Result;
var invoices = invoicesTask.Result;
```

### Practical — async all the way (avoid mixing sync/async)

```csharp
// ✅ Async all the way up the call chain
public async Task<IActionResult> GetOrder(int id)
{
    var order = await _orderService.GetOrderAsync(id);
    return order is null ? NotFound() : Ok(order);
}

public async Task<Order?> GetOrderAsync(int id)
{
    return await _dbContext.Orders.FindAsync(id);
}
```

### Exception handling with async/await

```csharp
public async Task ProcessAsync()
{
    try
    {
        await RiskyOperationAsync();
    }
    catch (HttpRequestException ex)
    {
        _logger.LogError(ex, "API call failed");
    }
}
```

## ⚡ Performance considerations

- Async doesn't make individual operations *faster* — it makes the **thread available for other work** while waiting, improving overall system throughput/scalability, not single-request latency.
- `Task.WhenAll` for independent async calls avoids unnecessary sequential waiting — a common, high-impact optimization.
- Excessive `async`/`await` for trivial, fast synchronous work adds small overhead (state machine allocation) without benefit.

## 🚨 Common mistakes

- ❌ Awaiting tasks sequentially when they're independent — wastes time; use `Task.WhenAll`.
- ❌ Mixing sync and async (`.Result`/`.Wait()` inside an `async` context) — deadlock risk.
- ❌ Forgetting `await` on a call — the method returns before the operation is done (silent bugs, "fire and forget").
- ❌ Marking a method `async` but never actually awaiting anything inside — compiler warning (CS4014-adjacent); no real async benefit, just overhead.
- ❌ Not using `ConfigureAwait(false)` in library code where continuing on the original context isn't needed (see `04_ConfigureAwait.md`).

## 💡 Best practices

- Go "async all the way" — don't mix blocking calls into an async call chain.
- Use `Task.WhenAll` for independent concurrent operations.
- Always name async methods with an `Async` suffix (convention).
- Return `Task`/`Task<T>` from async methods, never `async void` (except event handlers).
- Pass `CancellationToken` through async chains for long-running/cancelable operations.

## 🎤 Interview Questions

1. **Does `await` block the calling thread?**
   → No — it suspends the async method and releases the thread back to do other work; execution resumes via a continuation when the awaited task completes.
2. **What does the compiler actually generate for an `async` method?**
   → A state machine class implementing `IAsyncStateMachine`, with `MoveNext()` tracking which "step" the method is currently at.
3. **Why does awaiting tasks sequentially hurt performance compared to `Task.WhenAll`?**
   → Sequential awaits force each operation to fully complete before the next starts, summing their durations; `WhenAll` starts them concurrently, so total time ≈ the slowest one.
4. **What's the danger of calling `.Result` inside an `async` method in a UI/ASP.NET (classic) app?**
   → Deadlock — the `SynchronizationContext` tries to resume the continuation on the same (blocked) thread that's waiting on `.Result`.
5. **Is async/await about parallelism or concurrency?**
   → Concurrency (specifically for I/O-bound work) — it's about not blocking threads while waiting, not necessarily running things on multiple cores simultaneously (that's more `Task.Run`/`Parallel` territory for CPU-bound work).

## 📝 30-second Revision Cheat Sheet

| Concept                 | Key Point                                                  |
| ----------------------- | ---------------------------------------------------------- |
| `await`               | Suspends method, frees thread — does NOT block            |
| Compiler magic          | Generates a state machine behind the scenes                |
| Independent async calls | Use`Task.WhenAll`, not sequential awaits                 |
| Golden rule             | "Async all the way" — never mix`.Result`/`.Wait()` in |
| Naming                  | Suffix async methods with`Async`                         |
| Benefit                 | Scalability/responsiveness, not raw speed per call         |
