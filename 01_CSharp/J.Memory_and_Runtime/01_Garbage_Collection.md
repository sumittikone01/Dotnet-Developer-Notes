# Garbage Collection in C#

## 📌 What is it?

Garbage Collection (GC) is .NET's **automatic memory management** system — it tracks objects allocated on the managed heap and reclaims memory occupied by objects that are no longer reachable from your running code, so you don't have to manually `free()` memory like in C/C++.

## 🤔 Why do we need it?

- Eliminates entire classes of bugs: memory leaks from forgotten `free()`, dangling pointers, double-free errors.
- Lets developers focus on logic instead of manual memory bookkeeping.
- Automatically reclaims memory from short-lived objects (very common in typical apps) efficiently via generational collection.

## 🧠 Intuition

The GC periodically asks: *"Starting from my known 'roots' (static fields, local variables on active stacks, CPU registers), what objects can I still reach by following references?"* Anything **unreachable** is garbage — safe to reclaim. This is a **tracing/mark-and-sweep** style collector, not reference counting.

## 🌍 Real-world analogy

Think of a **library with automatic shelf-clearing**: the librarian periodically checks which books are still referenced by an active reading list (roots). Any book nobody has bookmarked or is currently reading gets removed from the shelf to make room for new books — you never have to manually return books yourself.

## ⚙️ Internal working — Generational GC

.NET's GC is **generational**, based on the empirical observation that most objects die young:

| Generation                        | What it holds                                            | Collected how often                                                 |
| --------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------- |
| **Gen 0**                   | Newly allocated, short-lived objects                     | Very frequently (fast, cheap)                                       |
| **Gen 1**                   | Objects that survived one Gen 0 collection (buffer zone) | Less frequently                                                     |
| **Gen 2**                   | Long-lived objects (survived multiple collections)       | Rarely (slow, expensive — scans a lot)                             |
| **LOH** (Large Object Heap) | Objects ≥ 85,000 bytes                                  | Collected with Gen 2, not compacted by default (fragmentation risk) |

**Collection steps (simplified):**

1. **Mark**: Starting from roots, trace all reachable objects.
2. **Sweep**: Anything not marked is garbage.
3. **Compact**: Move surviving objects together to eliminate gaps (reduces fragmentation, speeds up future allocation via a simple "bump the pointer" approach).
4. Objects that survive a collection get **promoted** to the next generation.

## 🖼 Diagram — Generational Heap

```
┌───────────────────────────────────────────────────────────┐
│                     Managed Heap                            │
│                                                               │
│  ┌────────┐   survives   ┌────────┐   survives   ┌────────┐ │
│  │ Gen 0  │ ────────────▶│ Gen 1  │ ────────────▶│ Gen 2  │ │
│  │ (new)  │  collection  │(buffer)│  collection  │ (old)  │ │
│  └────────┘              └────────┘              └────────┘ │
│   collected                collected               collected │
│   very often                sometimes               rarely   │
│                                                               │
│                    ┌─────────────────────┐                  │
│                    │  Large Object Heap  │  ← objects ≥85KB  │
│                    │       (LOH)          │  collected w/Gen2 │
│                    └─────────────────────┘                  │
└───────────────────────────────────────────────────────────────┘
```

## 💻 Code Examples

### Observing generations

```csharp
object obj = new object();
Console.WriteLine(GC.GetGeneration(obj)); // 0 — freshly allocated

GC.Collect(); // force a collection (rarely needed in real code!)
GC.WaitForPendingFinalizers();

Console.WriteLine(GC.GetGeneration(obj)); // may now be 1 if it survived
```

### Finalizers (rarely needed directly — prefer IDisposable)

```csharp
public class UnmanagedResourceHolder
{
    private IntPtr _handle;

    ~UnmanagedResourceHolder() // finalizer — called by GC before reclaiming
    {
        // last-resort cleanup if Dispose() was never called
        ReleaseHandle();
    }

    private void ReleaseHandle() { /* ... */ }
}
```

> ⚠️ Finalizers add overhead: objects with a finalizer survive at least one extra GC cycle (they go into a finalization queue first). Prefer `IDisposable` + `using` (see `02_IDisposable_and_using.md`) and only implement a finalizer as a safety net for unmanaged resources.

### Measuring memory pressure

```csharp
long before = GC.GetTotalMemory(false);
var list = new List<byte[]>();
for (int i = 0; i < 1000; i++)
    list.Add(new byte[10_000]);
long after = GC.GetTotalMemory(false);

Console.WriteLine($"Allocated approx: {(after - before) / 1024} KB");
```

### Avoiding unnecessary allocations (reduces GC pressure)

```csharp
// ❌ Allocates a new string every iteration — heavy GC pressure
string result = "";
for (int i = 0; i < 10000; i++)
    result += i.ToString();

// ✅ Single mutable buffer — far fewer allocations
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
    sb.Append(i);
string result2 = sb.ToString();
```

## 📊 Comparison Table — GC vs Manual Memory Management

| Aspect                     | Manual (C/C++)                  | .NET GC                                                         |
| -------------------------- | ------------------------------- | --------------------------------------------------------------- |
| Memory freed by            | Developer (`free`/`delete`) | Automatic, tracing collector                                    |
| Common bugs avoided        | —                              | Dangling pointers, double-free, most leaks                      |
| Control over timing        | Full                            | Indirect (`GC.Collect()` exists but rarely recommended)       |
| Performance predictability | High (deterministic)            | Less predictable (pauses during collection)                     |
| Leaks still possible?      | Yes (forgotten frees)           | Yes — via lingering references (event handlers, static caches) |

## ⚡ Performance considerations

- Gen 0 collections are **very fast** — this is by design, since most objects die young.
- Frequent large allocations (LOH) can cause fragmentation since LOH isn't compacted by default — consider `GCSettings.LargeObjectHeapCompactionMode` if this becomes an issue.
- Minimize allocations in hot paths (tight loops, high-throughput code) — fewer allocations means fewer/cheaper GC cycles. Use `Span<T>`, object pooling (`ArrayPool<T>`), and `StringBuilder` where appropriate.
- Avoid calling `GC.Collect()` manually in normal code — the GC already tunes itself; forcing collection usually **hurts** performance by discarding its generational optimizations.

## 🚨 Common mistakes

- ❌ Calling `GC.Collect()` "just to be safe" — almost always counterproductive.
- ❌ Believing GC prevents ALL memory leaks — objects still referenced (forgotten event subscriptions, static collections that grow forever) are never collected.
- ❌ Overusing finalizers — adds real overhead (extra GC generation survival) when `IDisposable`/`using` would suffice.
- ❌ Allocating heavily in hot loops (e.g., string concatenation with `+`) — increases GC pressure and pause frequency.
- ❌ Not unsubscribing from events — a classic "logical leak" where the GC correctly sees the object as reachable (via the event subscriber list) and never collects it.

## 💡 Best practices

- Minimize allocations in performance-critical code paths.
- Prefer `IDisposable`/`using` for deterministic cleanup of unmanaged resources over relying on finalizers.
- Unsubscribe from events / clear static references when objects should become eligible for collection.
- Use `Span<T>`, `ArrayPool<T>`, pooling patterns for high-throughput, allocation-sensitive code.
- Don't fight the GC — avoid manual `GC.Collect()` calls; trust the generational tuning unless you have measured, specific evidence otherwise.

## 🎤 Interview Questions

1. **What is generational garbage collection, and why does it exist?**
   → It groups objects by age (Gen 0/1/2) because most objects die young; collecting Gen 0 frequently and cheaply, while rarely scanning long-lived Gen 2 objects, is far more efficient than treating the whole heap uniformly.
2. **Can you have a memory leak in a garbage-collected language like C#?**
   → Yes — "logical leaks" happen when objects remain reachable unintentionally (unsubscribed event handlers, growing static collections, cached references never cleared).
3. **Why is `GC.Collect()` generally discouraged?**
   → It forces an out-of-cycle collection, often across generations, discarding the GC's own tuned heuristics — usually making things slower, not safer.
4. **What's the difference between the managed heap and the Large Object Heap (LOH)?**
   → Objects ≥ 85,000 bytes go to the LOH, which is collected alongside Gen 2 but isn't compacted by default, so it can suffer more fragmentation.
5. **Why do finalizers add overhead, and what's the recommended alternative?**
   → Objects with a finalizer must survive an extra GC cycle to be queued for finalization before being reclaimed; `IDisposable` + `using` gives deterministic, immediate cleanup without that overhead.

## 📝 30-second Revision Cheat Sheet

| Concept              | Key Point                                                    |
| -------------------- | ------------------------------------------------------------ |
| GC strategy          | Generational: Gen 0 (frequent) → Gen 1 → Gen 2 (rare)      |
| Roots                | Static fields, active stack locals, CPU registers            |
| LOH                  | Objects ≥ 85KB, not compacted by default                    |
| Leaks still possible | Via lingering references (events, static caches)             |
| Avoid                | Manual`GC.Collect()`, heavy finalizer use                  |
| Prefer               | `IDisposable`/`using`, minimize allocations in hot paths |
