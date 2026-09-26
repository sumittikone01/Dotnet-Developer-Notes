# Threads Overview in C#

## 📌 What is it?

A **thread** is the smallest unit of execution within a process. A .NET process starts with one thread (the main thread) but can create more via `System.Threading.Thread` to run code **concurrently** — multiple sequences of instructions progressing at the same time.

## 🤔 Why do we need it?

- Keep a UI responsive while doing heavy work in the background.
- Utilize multi-core CPUs — run independent work in parallel for speed.
- Handle multiple simultaneous requests (e.g., a web server handling many clients).

## 🧠 Intuition

A process is like an office building; threads are like employees inside it, all sharing the same resources (memory, files) but each working on their own task independently. They can step on each other's work if not coordinated (race conditions) — hence the need for synchronization.

## 🌍 Real-world analogy

Think of a **kitchen with one chef vs. multiple chefs**:

- Single-threaded = one chef doing everything sequentially: chop, then cook, then plate.
- Multi-threaded = multiple chefs working simultaneously — one chops, one cooks, one plates — but they share the same countertop (memory), so they need to coordinate to avoid collisions.

## ⚙️ Internal working

1. Each `Thread` object maps to an OS-level thread (with its own stack, ~1MB by default).
2. The OS scheduler decides which thread runs on which CPU core, and for how long (time-slicing/context switching) — this is **preemptive multitasking**.
3. Threads within the same process **share the heap** (objects, static fields) but have **separate stacks** (local variables).
4. Creating raw OS threads is relatively expensive (memory + context-switch overhead) — this is why `Task`/`ThreadPool` (see `02_Task_and_Task_T.md`) is usually preferred over manually creating `Thread` objects.

## 🖼 Diagram — Process, Threads, Shared Memory

```
┌─────────────────────────── Process ───────────────────────────┐
│                                                                 │
│   ┌────────────┐   ┌────────────┐   ┌────────────┐             │
│   │  Thread 1  │   │  Thread 2  │   │  Thread 3  │             │
│   │  own stack │   │  own stack │   │  own stack │             │
│   └─────┬──────┘   └─────┬──────┘   └─────┬──────┘             │
│         │                │                │                    │
│         └────────────────┴────────────────┘                    │
│                           │                                     │
│                 ┌─────────▼─────────┐                          │
│                 │   Shared Heap      │  ← objects, statics      │
│                 │  (race condition   │                          │
│                 │   risk here!)      │                          │
│                 └────────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 Comparison Table — Thread vs Task vs ThreadPool Thread

| Aspect             | `Thread` (manual)                      | `ThreadPool` thread  | `Task`                           |
| ------------------ | ---------------------------------------- | ---------------------- | ---------------------------------- |
| Creation cost      | High (~1MB stack, OS call)               | Low (reused from pool) | Low (uses pool internally)         |
| Best for           | Long-running, dedicated work             | Short background work  | Almost everything (modern default) |
| Return value       | No built-in mechanism                    | No built-in mechanism  | ✅`Task<T>` supports results     |
| Cancellation       | Manual (`Abort` — obsolete/dangerous) | Manual                 | ✅`CancellationToken` support    |
| Exception handling | Must handle inside thread                | Must handle inside     | ✅ Propagates via`Task`          |
| Recommended today  | Rarely (special cases only)              | Rarely directly        | ✅ Yes, default choice             |

## 💻 Code Examples

### Basic — creating a raw thread

```csharp
Thread t = new Thread(() =>
{
    Console.WriteLine($"Running on thread {Thread.CurrentThread.ManagedThreadId}");
});
t.Start();
t.Join(); // wait for it to finish
```

### Intermediate — passing data & background threads

```csharp
Thread worker = new Thread(DoWork)
{
    IsBackground = true // won't keep the process alive on its own
};
worker.Start("some parameter");

void DoWork(object? data)
{
    Console.WriteLine($"Processing: {data}");
}
```

### Practical — demonstrating a race condition (and why we need sync)

```csharp
int counter = 0;

void Increment()
{
    for (int i = 0; i < 100000; i++)
        counter++; // NOT thread-safe — read-modify-write isn't atomic
}

Thread t1 = new Thread(Increment);
Thread t2 = new Thread(Increment);
t1.Start(); t2.Start();
t1.Join(); t2.Join();

Console.WriteLine(counter); // often NOT 200000 — race condition!
```

```csharp
// Fixed with a lock
private readonly object _lock = new();
int counter = 0;

void Increment()
{
    for (int i = 0; i < 100000; i++)
    {
        lock (_lock)
        {
            counter++;
        }
    }
}
```

## ⚡ Performance considerations

- Each OS thread reserves ~1MB stack by default — spawning hundreds manually is wasteful.
- Context switching between threads has real CPU cost — too many threads can actually **slow down** throughput (thread thrashing).
- The `ThreadPool` reuses threads to avoid repeated creation/teardown cost — this is why `Task` (built on `ThreadPool`) is preferred for short-lived work.

## 🚨 Common mistakes

- ❌ Creating a new raw `Thread` for every small unit of work — expensive; use `Task.Run` instead.
- ❌ Accessing shared mutable state from multiple threads without synchronization (`lock`, `Monitor`, `Interlocked`).
- ❌ Using `Thread.Abort()` — deprecated and dangerous (can leave shared state corrupted); doesn't even exist in .NET Core/5+.
- ❌ Assuming `IsBackground = false` (default) won't matter — foreground threads **keep the process alive** even after `Main` returns.
- ❌ Confusing "concurrency" (tasks interleaving) with "parallelism" (tasks truly running simultaneously on multiple cores) — concurrency doesn't require multiple cores.

## 💡 Best practices

- Prefer `Task`/`async-await` over raw `Thread` for almost everything in modern C# (see `03_Async_Await.md`).
- Reserve raw `Thread` for genuinely long-running, dedicated background work (e.g., a dedicated polling thread) where pooling doesn't fit.
- Always protect shared mutable state with proper synchronization primitives.
- Set `IsBackground = true` for threads that shouldn't block app shutdown.
- Never manually kill threads — let them exit cleanly via cooperative cancellation.

## 🎤 Interview Questions

1. **What's the difference between concurrency and parallelism?**
   → Concurrency = multiple tasks making progress (possibly interleaved on one core); Parallelism = multiple tasks executing literally at the same time on multiple cores.
2. **Why is manually creating `Thread` objects discouraged in modern C#?**
   → High creation cost, no built-in result/cancellation/exception handling — `Task` (via `ThreadPool`) does all this more efficiently.
3. **What is a race condition? Give an example.**
   → When multiple threads access/modify shared state concurrently without synchronization, producing unpredictable results — e.g., two threads incrementing a shared counter simultaneously.
4. **Foreground vs background thread — what's the difference?**
   → A foreground thread keeps the process alive until it finishes; a background thread is terminated automatically when all foreground threads end.
5. **What does `Thread.Join()` do?**
   → Blocks the calling thread until the target thread finishes execution.

## 📝 30-second Revision Cheat Sheet

| Concept                  | Key Point                                                        |
| ------------------------ | ---------------------------------------------------------------- |
| Thread                   | Smallest unit of execution; own stack, shared heap               |
| Cost                     | ~1MB stack + OS overhead per thread — expensive to spawn freely |
| Race condition           | Unsynchronized shared-state access → unpredictable results      |
| Fix                      | `lock`, `Monitor`, `Interlocked`                           |
| Modern default           | Use`Task`/`async-await`, not raw `Thread`                  |
| Foreground vs background | Foreground keeps process alive; background doesn't               |
