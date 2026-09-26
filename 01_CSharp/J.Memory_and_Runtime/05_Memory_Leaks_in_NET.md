
# Memory Leaks in .NET

## 📌 What is it?

A memory leak in .NET is when objects remain **reachable** (still referenced, directly or indirectly, by a GC root) long after they're actually needed — so the Garbage Collector correctly leaves them alone, even though logically your application is "done" with them. It's not a bug in the GC; it's a bug in how references are held.

## 🤔 Why do we need to know this?

- ".NET has a GC" doesn't mean "leaks are impossible" — it's one of the most common misconceptions among developers new to managed languages.
- Leaks cause gradually increasing memory usage, eventually leading to `OutOfMemoryException`, degraded performance, or forced app restarts in production.
- Diagnosing them requires understanding *why* the GC thinks something is still reachable.

## 🧠 Intuition

The GC's rule is simple and unforgiving: **reachable = kept, unreachable = collected.** A "leak" in .NET isn't memory the GC forgot about — it's memory that's still technically reachable through a reference chain you didn't realize was still there (an event subscription, a static cache, a captured closure).

## 🌍 Real-world analogy

Like a library book that never gets returned because it's still technically checked out on someone's card — not because the librarian (GC) is broken, but because nobody remembered to check it back in. The book (object) sits on a shelf in someone's house (still referenced) forever, invisible to the librarian's "what's actually on loan" cleanup sweep.

## ⚙️ Common causes (internal mechanics)

| Cause                                                                 | Why it leaks                                                                                                                            |
| --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Unsubscribed event handlers**                                 | The publisher holds a reference to the subscriber via the event delegate list — subscriber can't be collected while publisher is alive |
| **Static fields/collections**                                   | Static references live for the entire application lifetime — anything added and never removed lives forever                            |
| **Captured variables in closures/lambdas**                      | A lambda that captures`this` or a large object can keep it alive longer than expected if the lambda itself is stored long-term        |
| **Undisposed `IDisposable` objects holding native resources** | Not a "GC memory leak" strictly, but unmanaged resource leaks (file handles, DB connections) behave similarly in effect                 |
| **Caching without eviction**                                    | An ever-growing in-memory cache with no expiration/size limit                                                                           |
| **Timers / background threads holding references**              | A`System.Timers.Timer` or long-running `Task` capturing `this` keeps the object alive as long as the timer/task runs              |

## 🖼 Diagram — Event Handler Leak

```
Publisher (long-lived, e.g., a service singleton)
   │
   │  event OnDataChanged
   │
   ▼
 subscriber list ──references──▶ Subscriber (should be short-lived, e.g., a form/view)

Even after the Subscriber "should" be done and go out of scope,
the Publisher's subscriber list STILL holds a reference to it.
GC sees it as reachable → never collected → LEAK.
```

## 💻 Code Examples

### ❌ Leak — unsubscribed event handler

```csharp
public class Publisher
{
    public event EventHandler? DataChanged;
    public void Raise() => DataChanged?.Invoke(this, EventArgs.Empty);
}

public class Subscriber
{
    public Subscriber(Publisher publisher)
    {
        publisher.DataChanged += OnDataChanged; // subscribes
        // never unsubscribes!
    }

    private void OnDataChanged(object? sender, EventArgs e) { /* ... */ }
}

// Even if `subscriber` goes out of scope in calling code,
// `publisher` still holds a reference via the event → Subscriber leaks
// as long as publisher is alive.
```

### ✅ Fixed — unsubscribe when done

```csharp
public class Subscriber : IDisposable
{
    private readonly Publisher _publisher;

    public Subscriber(Publisher publisher)
    {
        _publisher = publisher;
        _publisher.DataChanged += OnDataChanged;
    }

    private void OnDataChanged(object? sender, EventArgs e) { /* ... */ }

    public void Dispose()
    {
        _publisher.DataChanged -= OnDataChanged; // breaks the reference chain
    }
}
```

### ❌ Leak — unbounded static cache

```csharp
public static class Cache
{
    private static readonly Dictionary<string, byte[]> _store = new();

    public static void Add(string key, byte[] data) => _store[key] = data;
    // Never removes anything — grows forever, all entries live for the app's lifetime
}
```

### ✅ Fixed — bounded cache with eviction (e.g., `MemoryCache`)

```csharp
private readonly MemoryCache _cache = new(new MemoryCacheOptions
{
    SizeLimit = 1000
});

_cache.Set(key, data, new MemoryCacheEntryOptions
{
    Size = 1,
    SlidingExpiration = TimeSpan.FromMinutes(10) // auto-evicts unused entries
});
```

### ❌ Leak — closure capturing `this` in a long-lived timer

```csharp
public class ReportGenerator
{
    private Timer _timer;
    private byte[] _largeBuffer = new byte[100_000_000]; // 100MB

    public ReportGenerator()
    {
        _timer = new Timer(_ => DoWork(), null, 0, 5000);
        // the timer's callback captures `this` — ReportGenerator (and its 100MB buffer)
        // stays alive as long as the timer is running, even if nothing else references it
    }

    private void DoWork() { /* ... */ }
}
```

## 📊 Comparison Table — "Real" Leak vs. .NET Logical Leak

| Aspect         | C/C++ style leak          | .NET logical leak                                                         |
| -------------- | ------------------------- | ------------------------------------------------------------------------- |
| Cause          | Forgot to`free()`       | Object still reachable via forgotten reference                            |
| GC involvement | None (manual memory)      | GC works correctly — the "bug" is in reference management                |
| Fix            | Add the missing`free()` | Remove/break the unintended reference (unsubscribe, clear cache, dispose) |

## ⚡ Performance considerations

- Leaks cause the working set to grow over time, increasing GC scan cost (more Gen 2 objects to trace) even before running out of memory entirely.
- Long-running services (web servers, background workers) are especially vulnerable since small leaks compound over days/weeks of uptime — a leak invisible in short test runs can crash production after days.

## 🚨 Common mistakes

- ❌ Assuming ".NET has a GC, so I can't have memory leaks."
- ❌ Subscribing to events on long-lived publishers without ever unsubscribing.
- ❌ Using `static` collections as ad-hoc caches with no eviction strategy.
- ❌ Capturing large objects or `this` in long-lived closures (timers, background tasks, cached delegates).
- ❌ Not disposing `IDisposable` objects that hold unmanaged resources, assuming "the GC will handle it eventually" (it will, but too late and inconsistently).

## 💡 Best practices

- Always unsubscribe from events (`Dispose` pattern, weak event patterns, or `WeakReference` where appropriate).
- Use bounded caches with eviction policies (`MemoryCache`, sliding/absolute expiration) instead of raw static dictionaries.
- Be deliberate about what closures capture — avoid capturing `this` or large objects in long-lived delegates when not necessary.
- Dispose `IDisposable` resources promptly via `using`.
- Use memory profiling tools (dotMemory, Visual Studio Diagnostic Tools, `dotnet-counters`, `dotnet-gcdump`) to catch leaks during development, not just in production.

## 🎤 Interview Questions

1. **Can a garbage-collected language like C# have memory leaks? How?**
   → Yes — "logical leaks" occur when objects remain reachable through unintended references (events, statics, closures) even though the application logically no longer needs them.
2. **Why do event subscriptions commonly cause leaks?**
   → The publisher's event delegate list holds a reference to the subscriber; if the subscriber never unsubscribes, it stays reachable — and alive — for as long as the publisher exists.
3. **How would you diagnose a suspected memory leak in a running .NET application?**
   → Use memory profiling tools (dotMemory, `dotnet-gcdump`, Visual Studio Diagnostic Tools) to take heap snapshots over time and identify objects with unexpectedly growing instance counts and their reference chains (GC root paths).
4. **What's a "weak event pattern" and why does it help?**
   → It uses `WeakReference` so the publisher doesn't keep the subscriber alive just by holding a subscription — allows the subscriber to be collected even if it never explicitly unsubscribed.
5. **Why are static collections a common source of leaks?**
   → Static fields live for the entire application lifetime; anything added to a static collection and never removed is effectively "pinned" in memory forever.

## 📝 30-second Revision Cheat Sheet

| Concept         | Key Point                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------- |
| Root cause      | Object still reachable via an unintended reference                                              |
| Top culprits    | Unsubscribed events, static caches, closures capturing`this`/large objects, long-lived timers |
| GC's role       | Works correctly — the bug is in your reference graph, not the GC                               |
| Diagnosis tools | dotMemory,`dotnet-gcdump`, VS Diagnostic Tools                                                |
| Fix pattern     | Unsubscribe, evict, dispose, avoid unnecessary captures                                         |
