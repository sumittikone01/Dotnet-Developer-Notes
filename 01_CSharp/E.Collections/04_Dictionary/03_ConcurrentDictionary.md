# 🔐 ConcurrentDictionary<TKey, TValue>

## 📌 What is it?

`ConcurrentDictionary<TKey, TValue>` is a **thread-safe** version of `Dictionary<TKey,TValue>` — designed to be read from and written to by **multiple threads simultaneously** without corrupting data or requiring you to manually add locks.

```csharp
var dict = new ConcurrentDictionary<string, int>();

Parallel.For(0, 1000, i =>
{
    dict.AddOrUpdate("counter", 1, (key, oldValue) => oldValue + 1);
});

Console.WriteLine(dict["counter"]);   // 1000 — safe, no lost updates
```

## 🤔 Why do we need it?

A regular `Dictionary<TKey,TValue>` is **NOT thread-safe**. If multiple threads read/write it at the same time without external locking, you can get:

- Corrupted internal state
- Lost updates (two threads overwrite each other's changes)
- `InvalidOperationException` from concurrent modification
- In rare cases, an infinite loop / hang due to corrupted internal bucket structure

```csharp
// ❌ DANGEROUS — regular Dictionary with concurrent access, no locking
var dict = new Dictionary<string, int>();
Parallel.For(0, 1000, i => { dict["counter"] = dict.GetValueOrDefault("counter") + 1; });
// Result: unpredictable — often ends up LESS than 1000 due to lost updates, or even crashes
```

`ConcurrentDictionary` solves this by handling the necessary synchronization **internally**, so you don't have to wrap every access in `lock` statements yourself.

## 🌍 Real-world analogy

A regular `Dictionary` is like **one shared notebook with no rules** — if two people try to write in it at the same time, pages get torn or entries get overwritten. `ConcurrentDictionary` is like a **bank teller system with proper queuing** — many people can transact "simultaneously" from the outside, but the underlying system ensures each operation completes safely and correctly, without conflicts.

## ⚙️ Internal Working — Fine-Grained Locking

Rather than locking the **entire** dictionary on every operation (which would kill concurrency), `ConcurrentDictionary` internally divides its data into multiple **segments/buckets**, each with its own lock. This means threads working on **different segments** can proceed truly in parallel, while only threads touching the **same segment** briefly wait on each other.

```
Regular Dictionary + manual lock:
   Thread A ──┐
   Thread B ──┼──▶ [ONE big lock] ──▶ entire dictionary (heavy contention)
   Thread C ──┘

ConcurrentDictionary:
   Thread A ──▶ [Lock: Segment 1] ──▶ Segment 1
   Thread B ──▶ [Lock: Segment 2] ──▶ Segment 2   (parallel, no contention!)
   Thread C ──▶ [Lock: Segment 1] ──▶ waits briefly for Thread A only
```

## 💻 Key Thread-Safe Methods

| Method                                        | Purpose                                                                         |
| --------------------------------------------- | ------------------------------------------------------------------------------- |
| `TryAdd(key, value)`                        | Adds only if the key doesn't already exist — returns`bool` success           |
| `TryUpdate(key, newValue, comparisonValue)` | Updates only if current value matches expected value                            |
| `TryRemove(key, out value)`                 | Safely removes and retrieves the removed value                                  |
| `GetOrAdd(key, valueFactory)`               | Returns existing value, or**adds** it using a factory function if missing |
| `AddOrUpdate(key, addValue, updateFactory)` | Adds if missing, or**updates** using a function if it exists              |

```csharp
var dict = new ConcurrentDictionary<string, int>();

dict.TryAdd("Rohit", 25);                                  // true — added
dict.TryAdd("Rohit", 30);                                  // false — key exists, not added

// GetOrAdd — very common pattern: "get if exists, else compute and cache"
int age = dict.GetOrAdd("Priya", key => ComputeAge(key));

// AddOrUpdate — "insert new value, or update existing based on old value"
dict.AddOrUpdate("counter", 1, (key, oldVal) => oldVal + 1);
```

## 📊 `ConcurrentDictionary` vs `Dictionary` + Manual Lock

| Aspect                            | `Dictionary` + `lock`                | `ConcurrentDictionary`                           |
| --------------------------------- | ---------------------------------------- | -------------------------------------------------- |
| Concurrency granularity           | Whole dictionary locked at once          | Fine-grained (per-segment) locking                 |
| Performance under high contention | Poor — threads queue up                 | Better — parallel access to different segments    |
| Code complexity                   | You manage locks manually — error-prone | Built-in, no manual locking needed                 |
| Read performance                  | Also blocked during any write lock       | Reads are largely**lock-free** in many cases |

## 🚨 Common Mistakes

- ❌ Using a regular `Dictionary<TKey,TValue>` in multi-threaded code **without** any synchronization — leads to unpredictable bugs that may not show up in testing but appear under production load.
- ❌ Assuming `ConcurrentDictionary` makes **compound operations** atomic automatically — e.g. checking `ContainsKey()` then separately calling `Add()` is **still a race condition** between the two calls; use `TryAdd()` or `GetOrAdd()` instead, which are atomic as a single operation.
- ❌ Overusing `ConcurrentDictionary` when there's actually **no real multi-threaded access** — it has more overhead than a plain `Dictionary` for single-threaded scenarios, for no benefit.
- ❌ Passing an expensive `valueFactory` to `GetOrAdd()` — the factory **can be invoked more than once** under contention if multiple threads race for the same missing key (though only one result is actually stored) — don't rely on it running exactly once, and avoid side effects inside it.

## 💡 Best Practices

- Use `ConcurrentDictionary` whenever a dictionary is genuinely **shared and mutated across multiple threads** (e.g. a cache, a shared counter/registry).
- Prefer atomic methods (`TryAdd`, `GetOrAdd`, `AddOrUpdate`) over "check-then-act" patterns (`ContainsKey` + `Add`), which are still race-prone even on a thread-safe collection.
- Keep the `valueFactory` passed to `GetOrAdd`/`AddOrUpdate` **side-effect-free** — it might run more than once under contention.
- Don't reach for it by default in single-threaded code — plain `Dictionary` is faster when there's no real concurrency need.

## 🎤 Interview Questions

1. Why is a regular `Dictionary<TKey,TValue>` unsafe to use across multiple threads without external locking?
2. How does `ConcurrentDictionary` achieve better performance than a single global lock around a regular dictionary?
3. Why is `ContainsKey()` followed by `Add()` still a race condition, even on a `ConcurrentDictionary`?
4. Can the `valueFactory` passed to `GetOrAdd()` be called more than once? Why does that matter?

## 📝 30-second Revision Cheat Sheet

- `ConcurrentDictionary<K,V>` = thread-safe dictionary using fine-grained (segment-level) locking internally.
- Key atomic methods: `TryAdd`, `TryUpdate`, `TryRemove`, `GetOrAdd`, `AddOrUpdate`.
- Avoid "check-then-act" (`ContainsKey` + `Add`) — still a race condition; use atomic methods instead.
- `GetOrAdd`'s factory may run more than once under contention — keep it side-effect-free.
- Only use when there's real multi-threaded access — otherwise plain `Dictionary` is faster.
