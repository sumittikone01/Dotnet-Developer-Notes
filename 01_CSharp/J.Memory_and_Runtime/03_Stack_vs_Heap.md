# Stack vs Heap in C#

## 📌 What is it?

The **stack** and **heap** are two different memory regions .NET uses to store data at runtime. The stack stores method call frames and (typically) value types/local variables; the heap stores reference type instances and is managed by the Garbage Collector.

## 🤔 Why do we need it?

Understanding the distinction explains:

- Why value types (`int`, `struct`) are fast to allocate/deallocate but get **copied** on assignment.
- Why reference types (`class`) are more flexible but incur GC overhead.
- Why certain performance patterns (avoiding boxing, using `struct` for small hot-path data) matter.

## 🧠 Intuition

The **stack** is like a stack of trays in a cafeteria — strictly Last-In-First-Out (LIFO), extremely fast to push/pop, automatically cleaned up when a method returns. The **heap** is like a big, unordered storage warehouse — flexible, can hold things of any lifetime, but needs an active system (the GC) to figure out what's still needed and clean up the rest.

## 🌍 Real-world analogy

- **Stack** = your desk's inbox tray — items pile up and get removed in strict order as you finish tasks (methods return); when you leave for the day, the tray is instantly cleared.
- **Heap** = a shared storage warehouse — items can be placed and picked up in any order, by anyone with a claim ticket (a reference); a cleaning crew (GC) periodically checks which items nobody holds a ticket to anymore and removes them.

## ⚙️ Internal working

| Aspect        | Stack                                                      | Heap                                                                   |
| ------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------- |
| Allocation    | Extremely fast (just moves a pointer)                      | Slower (GC must track it)                                              |
| Deallocation  | Automatic, instant — when method returns (frame popped)   | Automatic, but delayed — via Garbage Collection                       |
| Stores        | Value types (locals), method call frames, return addresses | Reference type instances (`class`, `string`, arrays, boxed values) |
| Lifetime      | Tied to method scope                                       | Tied to reachability (until nothing references it)                     |
| Thread-safety | Each thread has its own stack                              | Shared across threads within a process                                 |
| Size          | Small, fixed (~1MB default per thread), can overflow       | Large, grows dynamically                                               |

> ⚠️ Important nuance: it's not strictly "value types → stack, reference types → heap." A value type that is a **field of a class instance**, or a **captured variable in a closure/lambda**, lives on the heap because it's part of a heap-allocated object. The rule of thumb is really about *local variables* directly on the stack.

## 🖼 Diagram

```
CALL STACK (per thread)              HEAP (shared, GC-managed)
┌─────────────────────┐             ┌───────────────────────┐
│ Main()               │             │                        │
│  int x = 5;   ◄──────┼── value     │  Person p = new Person()│
│  Person p ───────────┼── reference─┼─▶ { Name = "Alice" }   │
│                       │            │                        │
├─────────────────────┤             │  new int[1000]         │
│ DoWork()              │            │  (large array)          │
│  int y = 10;          │            │                        │
└─────────────────────┘             └───────────────────────┘
    LIFO, auto-cleaned                  GC tracks reachability
    on method return                     via references from stack
```

## 💻 Code Examples

### Value type behavior — copied, independent

```csharp
struct Point { public int X, Y; }

Point p1 = new Point { X = 1, Y = 2 };
Point p2 = p1;      // COPIES the value — p2 is independent
p2.X = 99;

Console.WriteLine(p1.X); // 1 — unaffected by p2's change
Console.WriteLine(p2.X); // 99
```

### Reference type behavior — shared, aliased

```csharp
class PointClass { public int X, Y; }

PointClass p1 = new PointClass { X = 1, Y = 2 };
PointClass p2 = p1;   // COPIES the reference, not the object — both point to the SAME heap object
p2.X = 99;

Console.WriteLine(p1.X); // 99 — p1 and p2 refer to the same object!
Console.WriteLine(p2.X); // 99
```

### A struct captured by a class field goes on the heap

```csharp
class Container
{
    public Point Location; // struct, but lives on the HEAP because it's part of this class instance
}

var c = new Container(); // c itself is heap-allocated; c.Location lives inside that same heap block
```

### Stack overflow from excessive recursion

```csharp
void RecurseForever()
{
    RecurseForever(); // each call pushes a new frame onto the stack
}
// Eventually throws StackOverflowException — stack has a small fixed size (~1MB by default)
```

## 📊 Comparison Table — Value Type vs Reference Type

| Aspect                         | Value type (`struct`, `int`, `bool`, etc.) | Reference type (`class`, `string`, arrays) |
| ------------------------------ | ------------------------------------------------ | ---------------------------------------------- |
| Assignment                     | Copies the actual data                           | Copies the reference (pointer)                 |
| Default location               | Stack (if a local variable)                      | Always heap                                    |
| Passed to methods (by default) | By value (copy)                                  | By reference (copy of the reference)           |
| `null`able by default?       | No (unless`Nullable<T>`/`T?`)                | Yes                                            |
| Equality (`==`) default      | Value comparison (bitwise for structs)           | Reference comparison (unless overridden)       |

## ⚡ Performance considerations

- Stack allocation/deallocation is essentially free (pointer bump) — favor value types for small, short-lived, frequently-created data in hot paths.
- Large `struct`s get expensive to copy repeatedly (every assignment/parameter pass copies the whole thing) — prefer `class` or `in`/`ref` parameters for large value types.
- Excessive heap allocations increase GC pressure — see `01_Garbage_Collection.md`.
- Deep/unbounded recursion can exhaust the stack (`StackOverflowException`), which — unlike most exceptions — **cannot be caught** and crashes the process immediately.

## 🚨 Common mistakes

- ❌ Assuming all value types are always on the stack — they're on the heap when they're fields of a class or captured in a closure.
- ❌ Passing large structs by value repeatedly without realizing the copy cost — use `in` parameter modifier for read-only large structs.
- ❌ Confusing "copying a reference" with "copying an object" — assigning a class instance to a new variable copies the *reference*, not the underlying data (mutations are shared).
- ❌ Writing deeply recursive algorithms without a base case or with excessive depth — `StackOverflowException` crashes immediately and can't be handled.
- ❌ Assuming struct equality (`==`) works the same as class equality without checking — structs default to bitwise/field comparison, classes default to reference comparison.

## 💡 Best practices

- Use `struct` for small, immutable, short-lived data (e.g., `Point`, `Money`) where copy semantics are actually desired.
- Use `class` for anything with identity, larger data, or that needs to be shared/mutated across references.
- Pass large structs with `in` (read-only reference) to avoid copy overhead while keeping value semantics.
- Be deliberate about mutable structs — mutating a copy is a classic, confusing bug source; prefer immutable structs.
- Watch recursion depth in algorithms working on user-supplied/unbounded input — consider iterative alternatives for deep recursion.

## 🎤 Interview Questions

1. **Are value types always stored on the stack?**
   → No — only when they're local variables/parameters directly on the stack. If a value type is a field of a class instance or captured by a closure, it lives on the heap as part of that object.
2. **What happens when you assign one class instance to another variable?**
   → The reference (pointer to the heap object) is copied, not the object itself — both variables point to the same underlying object.
3. **Why is passing large structs by value potentially expensive?**
   → Every assignment/parameter pass makes a full bitwise copy of all its fields — for large structs, this can be more expensive than passing a reference.
4. **Why can't you catch `StackOverflowException`?**
   → Because the stack is already exhausted when it's thrown — there's no safe way to unwind or execute a catch handler; the CLR terminates the process immediately.
5. **What's the default equality behavior difference between structs and classes?**
   → Structs default to value/bitwise field comparison; classes default to reference comparison (same object identity) unless `Equals`/`==` is overridden.

## 📝 30-second Revision Cheat Sheet

| Concept        | Key Point                                                    |
| -------------- | ------------------------------------------------------------ |
| Stack          | Fast, LIFO, auto-cleaned on method return, small fixed size  |
| Heap           | Flexible, GC-managed, shared across threads                  |
| Value type     | Copied on assignment; stack if local, heap if inside a class |
| Reference type | Reference copied; data always on heap                        |
| Big structs    | Expensive to copy — use`in` parameter                     |
| Deep recursion | Risks uncatchable`StackOverflowException`                  |
