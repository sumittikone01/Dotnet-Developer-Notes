# Try, Catch, Finally in C#

## 📌 What is it?

`try-catch-finally` is C#'s structured mechanism for handling runtime errors (exceptions) gracefully — instead of letting the program crash, you catch the problem, react to it, and keep control of the flow.

- **try** → the "risky" code block you want to watch.
- **catch** → the "what to do if it breaks" block.
- **finally** → the "always run this no matter what" block.

## 🤔 Why do we need it?

Without exception handling, a single runtime error (null reference, divide by zero, file not found) would crash the entire application. `try-catch-finally` lets you:

- Isolate risky operations (I/O, DB calls, parsing, network).
- Fail gracefully with meaningful messages instead of a stack-trace crash.
- Guarantee cleanup (closing connections, releasing locks) even when things go wrong.

## 🧠 Intuition

Exceptions are a **signal thrown up the call stack** until someone catches them. If nobody catches it, the runtime terminates the program. `try-catch` is you saying: *"I know this line might fail — if it does, jump here instead of crashing."*

## 🌍 Real-world analogy

Like a **fire alarm system** in a building:

- `try` = the building operating normally.
- An exception = a fire breaking out.
- `catch` = the evacuation plan — handle the emergency.
- `finally` = the security guard locking doors on the way out — happens **whether or not** there was a fire.

## ⚙️ Internal working

1. CLR executes the `try` block statement by statement.
2. If an exception is thrown, execution stops immediately at that line.
3. CLR searches **up the call stack** for a matching `catch` (matched by exception type, most specific first).
4. Match found → control jumps into that `catch`, exception object passed in.
5. No match anywhere in the stack → **unhandled exception**, process crashes.
6. `finally` **always executes** before the method exits — whether an exception occurred, was caught, or not (exceptions: `Environment.FailFast`, process kill, `StackOverflowException`).

## 🖼 Flow Diagram

```
┌────────────┐
│   try{ }   │  ← risky code runs here
└─────┬──────┘
      │ exception thrown?
      ▼
   ┌──Yes───────────────┐     No
   │ matching catch?    │──────────┐
   └─────┬───────────────┘         │
     Yes │   No                    │
         ▼    └──► propagate up    │
   ┌────────────┐  call stack      │
   │  catch{ }  │                  │
   └─────┬──────┘                  │
         ▼                         ▼
   ┌──────────────────────────────────┐
   │            finally{ }            │  ← ALWAYS runs
   └──────────────────────────────────┘
                  ▼
          method continues / exits
```

## 💻 Code Examples

### Basic

```csharp
try
{
    int[] numbers = { 1, 2, 3 };
    Console.WriteLine(numbers[5]); // throws IndexOutOfRangeException
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    Console.WriteLine("Cleanup done.");
}
```

### Intermediate — multiple catch blocks (specific → general)

```csharp
try
{
    string input = null;
    int result = int.Parse(input);
}
catch (ArgumentNullException ex)
{
    Console.WriteLine($"Null argument: {ex.Message}");
}
catch (FormatException ex)
{
    Console.WriteLine($"Invalid format: {ex.Message}");
}
catch (Exception ex) // generic — must be LAST
{
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

> ⚠️ Order matters: C# checks catch blocks top-to-bottom. A more general type (like `Exception`) placed before a specific one makes the specific one **unreachable** — compiler error (CS0160).

### Practical — DB connection with guaranteed cleanup

```csharp
SqlConnection conn = new SqlConnection(connectionString);
try
{
    conn.Open();
    using var cmd = new SqlCommand("SELECT * FROM Users", conn);
    using var reader = cmd.ExecuteReader();
    while (reader.Read())
    {
        Console.WriteLine(reader["Name"]);
    }
}
catch (SqlException ex)
{
    _logger.LogError(ex, "Database operation failed");
    throw; // re-throw, preserving original stack trace
}
finally
{
    if (conn.State == ConnectionState.Open)
        conn.Close(); // always executes, even if exception happened
}
```

### `throw` vs `throw ex` (important distinction)

```csharp
catch (Exception ex)
{
    throw;      // ✅ preserves original stack trace
    throw ex;   // ❌ resets stack trace — you lose where it actually broke
}
```

## ⚡ Performance considerations

- Exceptions are **expensive** (stack unwinding, stack trace capture). Never use them for normal control flow (e.g., validating expected user input).
- Prefer `TryParse`, `TryGetValue` patterns over try-catch for expected failure paths.
- An empty `try` block with no exception thrown has near-zero cost — the cost is only on the **throw**, not the wrapping.

## 🚨 Common mistakes

- ❌ Catching `Exception` everywhere and swallowing errors silently (`catch { }`).
- ❌ Using `throw ex;` instead of `throw;` — destroys the original stack trace.
- ❌ Using exceptions for expected/normal logic paths (e.g., "user not found" via exception instead of a null check).
- ❌ Forgetting `finally` for unmanaged resource cleanup (though `using` is preferred today — see `02_IDisposable_and_using.md`).
- ❌ Catching a broad exception type just to suppress a compiler warning.

## 💡 Best practices

- Catch the **most specific exception type** you can meaningfully handle.
- Log the full exception (`ex.ToString()` or structured logging), not just `ex.Message`.
- Prefer `using`/`using var` over manual `finally` cleanup for `IDisposable` objects.
- Only catch what you can actually recover from — let everything else propagate.
- Use custom exceptions for domain-specific failure cases (see `03_Custom_Exceptions.md`).

## 🎤 Interview Questions

1. **What is the execution order if an exception occurs inside `try` and there's a `return` in both `catch` and `finally`?**
   → `finally` always runs before the method returns; if `finally` also has a `return`, it **overrides** the one in `catch`.
2. **Does `finally` run if there's a `return` statement inside `try`?**
   → Yes, always, right before the actual return happens.
3. **Difference between `throw` and `throw ex`?**
   → `throw` preserves the original stack trace; `throw ex` resets it to the current line.
4. **Can you have `try` without `catch`?**
   → Yes — `try { } finally { }` is valid, used purely for guaranteed cleanup.
5. **What happens if an exception occurs inside a `finally` block?**
   → It replaces/masks any exception from the `try`/`catch` — the new exception propagates instead, and the original is lost unless explicitly handled.
6. **Is `catch (Exception)` a bad practice?**
   → Not inherently, but blanket-catching without logging or re-throwing hides bugs. Fine at application boundaries (e.g., global error handler) if logged.

## 📝 30-second Revision Cheat Sheet

| Concept       | Key Point                                                      |
| ------------- | -------------------------------------------------------------- |
| `try`       | Wraps risky code                                               |
| `catch`     | Handles matched exception type, most specific first            |
| `finally`   | Always runs — cleanup, even on`return`/unhandled exceptions |
| `throw;`    | Preserves stack trace ✅                                       |
| `throw ex;` | Resets stack trace ❌                                          |
| Cost          | Cheap to wrap, expensive to actually throw                     |
| Rule          | Catch only what you can handle; let the rest propagate         |
