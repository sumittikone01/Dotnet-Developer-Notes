# Exception Types & Hierarchy in C#

## 📌 What is it?

Every exception in .NET is an object derived from the base class `System.Exception`. The hierarchy defines **how specific your catch blocks can be** and how the runtime decides which `catch` matches a thrown exception.

## 🤔 Why do we need it?

Understanding the hierarchy lets you:

- Catch the **right level** of exception (specific vs general).
- Know which exceptions are recoverable vs which indicate serious bugs (and shouldn't be caught).
- Build meaningful custom exceptions that slot correctly into the tree.

## 🧠 Intuition

It's a **tree**, not a flat list. `catch (Exception ex)` matches *everything* because every exception type eventually inherits from `Exception`. Catching a base type also catches all its derived types — like a filter that's wide open.

## 🌍 Real-world analogy

Think of it like a **medical triage hierarchy**:

- `Exception` = "patient is unwell" (too vague to act on).
- `SystemException` = "internal/system issue" (like a hospital equipment fault).
- `ArgumentException` = "wrong medicine dose given" (more specific, actionable).
- `ArgumentNullException` = "no medicine given at all" (most specific — exact diagnosis).

The more specific the diagnosis, the more precisely you can treat it.

## ⚙️ Internal working

- `System.Exception` is the root of **all** exceptions (checked and unchecked — C# doesn't distinguish like Java).
- Two major built-in branches:
  - **`SystemException`** → thrown by the CLR/.NET runtime itself (e.g., `NullReferenceException`, `IndexOutOfRangeException`).
  - **`ApplicationException`** → *intended* for user-defined exceptions, but Microsoft itself now recommends **not** using it — inherit directly from `Exception` instead.
- `catch` matching walks the hierarchy: a thrown `ArgumentNullException` will match a `catch (ArgumentException)` block because `ArgumentNullException : ArgumentException : SystemException : Exception`.

## 🖼 Hierarchy Diagram

```
                    System.Exception
                    /              \
         SystemException      ApplicationException
          (CLR-thrown)          (legacy, avoid)
         /      |       \
        /       |        \
NullReference  IndexOutOf  InvalidOperation
Exception      RangeException  Exception
        |
   ArgumentException
        |
   ArgumentNullException / ArgumentOutOfRangeException

IO branch:            Format branch:
IOException            FormatException
 └─ FileNotFoundException

Data/DB branch:
DbException
 └─ SqlException
```

## 📊 Comparison Table — Common Built-in Exceptions

| Exception Type                  | Thrown When                                        | Recoverable?                           |
| ------------------------------- | -------------------------------------------------- | -------------------------------------- |
| `NullReferenceException`      | Accessing a member on a`null` object             | Usually a bug — fix code, don't catch |
| `ArgumentNullException`       | Required argument passed as`null`                | Yes — validate input                  |
| `ArgumentOutOfRangeException` | Argument value outside valid range                 | Yes — validate input                  |
| `IndexOutOfRangeException`    | Array/collection index invalid                     | Usually a bug — fix logic             |
| `InvalidOperationException`   | Method called at wrong time/state                  | Yes — check state first               |
| `FormatException`             | String not in expected format (e.g.,`int.Parse`) | Yes — use`TryParse` instead         |
| `DivideByZeroException`       | Integer division by zero                           | Yes — validate divisor                |
| `FileNotFoundException`       | File path doesn't exist                            | Yes — check`File.Exists`            |
| `SqlException`                | Database operation fails                           | Yes — retry/log                       |
| `StackOverflowException`      | Call stack exceeded (infinite recursion)           | ❌ Cannot be caught — crashes process |
| `OutOfMemoryException`        | Insufficient memory                                | ❌ Rarely recoverable                  |

## 💻 Code Examples

### Basic — catching by hierarchy level

```csharp
try
{
    int x = int.Parse("abc"); // throws FormatException
}
catch (ArgumentException ex) // FormatException does NOT derive from ArgumentException!
{
    Console.WriteLine("Won't catch it here");
}
catch (FormatException ex) // ✅ this one matches
{
    Console.WriteLine($"Format error: {ex.Message}");
}
```

### Intermediate — exploiting hierarchy for grouped handling

```csharp
try
{
    ProcessOrder(orderId: -1);
}
catch (ArgumentException ex) // catches ArgumentNullException, ArgumentOutOfRangeException, etc.
{
    _logger.LogWarning("Invalid argument: {Message}", ex.Message);
}
catch (SystemException ex) // catches any CLR-level exception not caught above
{
    _logger.LogError(ex, "Runtime error");
}
```

### Practical — inspecting the hierarchy at runtime

```csharp
try
{
    throw new ArgumentNullException("id");
}
catch (Exception ex)
{
    Type t = ex.GetType();
    while (t != null)
    {
        Console.WriteLine(t.Name);
        t = t.BaseType;
    }
    // Output:
    // ArgumentNullException
    // ArgumentException
    // SystemException
    // Exception
    // Object
}
```

## ⚡ Performance considerations

- Hierarchy depth doesn't meaningfully affect performance — the cost is in the `throw`, not in how many catch blocks the CLR checks.
- Catching at the right specificity avoids unnecessary rethrows/reprocessing logic.

## 🚨 Common mistakes

- ❌ Assuming `FormatException` derives from `ArgumentException` — it doesn't (it derives directly from `SystemException`).
- ❌ Inheriting custom exceptions from `ApplicationException` — outdated guidance; inherit from `Exception` directly.
- ❌ Catching `NullReferenceException` instead of fixing the null-check bug that caused it.
- ❌ Trying to catch `StackOverflowException` or `OutOfMemoryException` — these terminate the process regardless.
- ❌ Putting a general `catch (Exception)` block **before** a specific one — compiler error, unreachable code.

## 💡 Best practices

- Catch order: **most specific → most general**.
- Use the hierarchy intentionally: catch `ArgumentException` when you want to handle *any* bad-argument scenario in one place.
- Reserve `catch (Exception)` for top-level/global handlers (middleware, `Main`), not business logic.
- When creating custom exceptions, inherit from `Exception` (see `03_Custom_Exceptions.md`), never `ApplicationException`.

## 🎤 Interview Questions

1. **What's the difference between `SystemException` and `ApplicationException`?**
   → `SystemException` is thrown by the CLR/runtime; `ApplicationException` was meant for user code but is now discouraged — use `Exception` directly.
2. **Does `FormatException` inherit from `ArgumentException`?**
   → No — both inherit separately from `SystemException`.
3. **If you catch `Exception` first, can you still add a `catch (ArgumentException)` after it?**
   → No — compiler error (CS0160), since it's unreachable.
4. **Can `StackOverflowException` be caught with try-catch?**
   → No, it immediately terminates the process; cannot be handled in managed code.
5. **Why is checking `ex is ArgumentException` sometimes better than a `catch (ArgumentException)`?**
   → When you need conditional handling within a single generic catch block, e.g., logging differently per type without duplicating catch clauses.

## 📝 30-second Revision Cheat Sheet

| Concept             | Key Point                                                    |
| ------------------- | ------------------------------------------------------------ |
| Root                | Everything inherits from`System.Exception`                 |
| CLR errors          | `SystemException` branch                                   |
| Custom exceptions   | Inherit from`Exception`, NOT `ApplicationException`      |
| Catch order         | Specific → General (compiler enforces this)                 |
| Uncatchable         | `StackOverflowException`, usually `OutOfMemoryException` |
| `FormatException` | Independent branch, NOT under`ArgumentException`           |
