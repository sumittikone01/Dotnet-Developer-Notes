# Task and Task&lt;T&gt; in C#

## 📌 What is it?

`Task` and `Task<T>` represent an **asynchronous operation** — a promise that some work will complete (possibly on another thread, possibly not) at some point, optionally producing a result. `Task` = "will complete, no result." `Task<T>` = "will complete and return a `T`."

## 🤔 Why do we need it?

- Raw `Thread` gives you no way to know when work finishes, get a return value, or handle exceptions cleanly.
- `Task` wraps all of that: completion state, result, exception propagation, cancellation, and composability (`await`, `WhenAll`, `WhenAny`).
- It's the foundation `async/await` is built on.

## 🧠 Intuition

A `Task` is like a **claim ticket** at a dry cleaner. You drop off your clothes (start the work), get a ticket (`Task` object) immediately, and go do other things. Later you can either wait at the counter (`.Wait()`/`.Result` — blocking) or come back when notified (`await` — non-blocking) to collect your clean clothes (the result).

## 🌍 Real-world analogy

Ordering food delivery:

- Placing the order = starting the `Task`.
- The order confirmation/tracking link = the `Task` object itself.
- `await` = getting a notification when it arrives, freeing you to do other things meanwhile.
- `.Result`/`.Wait()` = standing at the door staring at the street until it arrives (blocking — wastes your time).

## ⚙️ Internal working

- Most `Task`s run on the **ThreadPool** (a pool of reusable worker threads) rather than spinning up dedicated OS threads.
- A `Task` has a state machine: `Created → Running → RanToCompletion / Faulted / Canceled`.
- `Task<T>.Result` holds the return value once completed; accessing it before completion **blocks** the calling thread.
- Exceptions thrown inside a `Task` don't crash the app immediately — they're captured and stored in the `Task`, surfaced when you `await` it or access `.Result` (wrapped in `AggregateException` if accessed via `.Result`/`.Wait()`).
- I/O-bound tasks (`Task.Delay`, file/network calls) often use **no thread at all** while waiting — they register a callback and release the thread back to the pool (this is the magic behind scalable async I/O).

## 🖼 Diagram — Task Lifecycle

```
   Task.Run(...) / async method called
              │
              ▼
       ┌─────────────┐
       │   Created    │
       └──────┬───────┘
              ▼
       ┌─────────────┐
       │   Running    │  ← executing on ThreadPool (or suspended for I/O)
       └──────┬───────┘
        ┌─────┼──────┐
        ▼     ▼       ▼
 RanToCompletion  Faulted  Canceled
   (has Result)  (has Exception)
```

## 📊 Comparison Table — `Task` vs `Task<T>` vs `void`

| Return type      | Meaning                | Awaitable? | Exception handling                                                  |
| ---------------- | ---------------------- | ---------- | ------------------------------------------------------------------- |
| `void` (async) | Fire-and-forget        | ❌ No      | Exceptions can crash the process — avoid except for event handlers |
| `Task`         | Async op, no result    | ✅ Yes     | Captured, observed on`await`                                      |
| `Task<T>`      | Async op, returns`T` | ✅ Yes     | Captured, observed on`await`                                      |

## 💻 Code Examples

### Basic — starting and awaiting a Task

```csharp
Task<int> task = Task.Run(() =>
{
    Thread.Sleep(1000); // simulate work
    return 42;
});

int result = await task; // non-blocking wait
Console.WriteLine(result);
```

### Intermediate — running multiple tasks concurrently

```csharp
Task<int> t1 = Task.Run(() => ComputeSquare(5));
Task<int> t2 = Task.Run(() => ComputeSquare(10));

int[] results = await Task.WhenAll(t1, t2); // waits for BOTH, runs concurrently
Console.WriteLine($"Sum: {results.Sum()}");

int ComputeSquare(int x)
{
    Thread.Sleep(500);
    return x * x;
}
```

### Practical — handling exceptions from a Task

```csharp
Task<int> task = Task.Run(() =>
{
    throw new InvalidOperationException("Something broke");
    return 0;
});

try
{
    int result = await task; // exception rethrown here, original stack trace preserved
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Caught: {ex.Message}");
}
```

### `Task.WhenAny` — race multiple tasks, take the first

```csharp
Task<string> fastApi = CallApiAsync("fast-service");
Task<string> slowApi = CallApiAsync("slow-service");

Task<string> firstDone = await Task.WhenAny(fastApi, slowApi);
Console.WriteLine($"First result: {await firstDone}");
```

### ❌ Anti-pattern — blocking on async code

```csharp
// DON'T do this in async code — risks deadlocks (esp. in ASP.NET/UI apps)
int result = SomeAsyncMethod().Result;
SomeAsyncMethod().Wait();
```

## ⚡ Performance considerations

- `Task.Run` for CPU-bound work uses a ThreadPool thread — fine for parallelizable compute, wasteful for I/O-bound work (which doesn't need a thread at all while waiting).
- Excessive `Task.Run` calls for trivial work adds scheduling overhead without benefit.
- `Task<T>.Result`/`.Wait()` block the calling thread — in UI or ASP.NET (classic) contexts this can cause **deadlocks** due to `SynchronizationContext` capture. Always prefer `await`.

## 🚨 Common mistakes

- ❌ Using `.Result` or `.Wait()` instead of `await` — deadlock risk, defeats the purpose of async.
- ❌ Wrapping already-async I/O calls in `Task.Run` (e.g., `Task.Run(() => httpClient.GetAsync(url))`) — pointless, `GetAsync` is already non-blocking.
- ❌ Forgetting to `await` a `Task` — the method continues without waiting, silently losing exceptions ("fire and forget" bugs).
- ❌ Using `async void` for anything except top-level event handlers — exceptions can't be caught by the caller.
- ❌ Not disposing `CancellationTokenSource` when done.

## 💡 Best practices

- Use `Task.Run` only for **CPU-bound** work; use native async APIs (`HttpClient.GetAsync`, `File.ReadAllTextAsync`) directly for I/O-bound work — no `Task.Run` wrapper needed.
- Always `await` tasks; never leave them "dangling" unless truly fire-and-forget (and even then, handle exceptions explicitly).
- Use `Task.WhenAll` to run independent tasks concurrently instead of awaiting them one by one sequentially.
- Prefer `Task<T>` return types on your own async methods over `async void`.
- Pass and honor `CancellationToken` for long-running tasks (see `05_CancellationToken.md`).

## 🎤 Interview Questions

1. **What's the difference between `Task.Run` and just calling an async method directly?**
   → `Task.Run` queues CPU-bound work onto the ThreadPool; calling an already-async method (like `HttpClient.GetAsync`) directly doesn't need a dedicated thread — it uses I/O completion ports/callbacks instead.
2. **Why is `.Result`/`.Wait()` dangerous in ASP.NET (classic) or WinForms/WPF apps?**
   → It can deadlock: the `SynchronizationContext` tries to resume the continuation on the original thread, which is itself blocked waiting on `.Result`.
3. **What happens to an exception thrown inside a `Task` that's never awaited?**
   → It becomes an "unobserved task exception" — may go unnoticed, or in some configurations crash the process during garbage collection via `TaskScheduler.UnobservedTaskException`.
4. **Difference between `Task.WhenAll` and `Task.WhenAny`?**
   → `WhenAll` completes when **all** given tasks complete (aggregates exceptions); `WhenAny` completes as soon as the **first** one finishes.
5. **Why avoid `async void`?**
   → Exceptions thrown inside can't be caught by the caller via try-catch — they propagate directly to the `SynchronizationContext`, often crashing the app.

## 📝 30-second Revision Cheat Sheet

| Concept                    | Key Point                                                        |
| -------------------------- | ---------------------------------------------------------------- |
| `Task`                   | Async operation, no return value                                 |
| `Task<T>`                | Async operation, returns`T`                                    |
| Use`Task.Run` for        | CPU-bound work only                                              |
| Use native async for       | I/O-bound work (no`Task.Run` needed)                           |
| Never                      | `.Result` / `.Wait()` in async code — use `await`         |
| `WhenAll` vs `WhenAny` | Wait for all vs wait for first                                   |
| `async void`             | Only for event handlers — exceptions escape try-catch otherwise |
