# Custom Exceptions in C#

## 📌 What is it?

A custom exception is a user-defined class that inherits from `System.Exception` (or one of its descendants), used to represent **domain-specific failure cases** that built-in exceptions don't clearly express — e.g., `InsufficientFundsException`, `OrderAlreadyShippedException`.

## 🤔 Why do we need it?

- Built-in exceptions are generic (`InvalidOperationException`, `ArgumentException`) — they don't tell you *what business rule* was violated.
- Custom exceptions make error handling **self-documenting** and let calling code catch specific business failures without parsing error strings.
- They carry extra structured data relevant to the failure (e.g., `OrderId`, `AttemptedAmount`).

## 🧠 Intuition

Instead of throwing a generic `Exception("Not enough balance")` and having callers `string.Contains()` the message to figure out what happened, you throw a typed `InsufficientFundsException` that callers can catch directly — type-safe, refactor-safe, testable.

## 🌍 Real-world analogy

Generic exceptions are like a doctor's note that just says "patient is sick." A custom exception is a proper diagnosis code — "Type 2 Diabetes" — precise enough that the next doctor (calling code) knows exactly how to respond, without re-investigating.

## ⚙️ Internal working

- Inherit from `Exception` (not `ApplicationException` — deprecated guidance).
- Implement the standard constructors (parameterless, message-only, message+innerException) for compatibility with serialization, logging frameworks, and rethrow patterns.
- Optionally add custom properties to carry structured context.
- The CLR treats it exactly like any built-in exception once thrown — same stack-unwinding, same `catch` matching by type.

## 📊 Comparison Table — Built-in vs Custom

| Aspect            | Built-in Exception                   | Custom Exception            |
| ----------------- | ------------------------------------ | --------------------------- |
| Meaning           | Generic (`ArgumentException`)      | Specific to your domain     |
| Extra data        | Minimal (`Message`, `ParamName`) | Whatever properties you add |
| Catch specificity | Broad                                | Narrow, precise             |
| When to use       | Framework/technical failures         | Business rule violations    |

## 💻 Code Examples

### Basic — minimal custom exception

```csharp
public class InsufficientFundsException : Exception
{
    public InsufficientFundsException() { }
    public InsufficientFundsException(string message) : base(message) { }
    public InsufficientFundsException(string message, Exception inner) : base(message, inner) { }
}
```

### Intermediate — with structured context

```csharp
public class InsufficientFundsException : Exception
{
    public decimal AttemptedAmount { get; }
    public decimal AvailableBalance { get; }

    public InsufficientFundsException(decimal attempted, decimal available)
        : base($"Attempted to withdraw {attempted:C} but only {available:C} available.")
    {
        AttemptedAmount = attempted;
        AvailableBalance = available;
    }
}

// Usage
public void Withdraw(decimal amount)
{
    if (amount > _balance)
        throw new InsufficientFundsException(amount, _balance);

    _balance -= amount;
}
```

### Practical — catching custom exceptions specifically

```csharp
try
{
    account.Withdraw(500m);
}
catch (InsufficientFundsException ex)
{
    _logger.LogWarning(
        "Withdrawal failed: attempted {Attempted}, available {Available}",
        ex.AttemptedAmount, ex.AvailableBalance);

    return BadRequest($"Insufficient funds. Available: {ex.AvailableBalance:C}");
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error during withdrawal");
    return StatusCode(500);
}
```

### Exception hierarchies for your own domain

```csharp
public abstract class OrderException : Exception
{
    public int OrderId { get; }
    protected OrderException(int orderId, string message) : base(message)
        => OrderId = orderId;
}

public class OrderAlreadyShippedException : OrderException
{
    public OrderAlreadyShippedException(int orderId)
        : base(orderId, $"Order {orderId} has already shipped and cannot be modified.") { }
}

public class OrderNotFoundException : OrderException
{
    public OrderNotFoundException(int orderId)
        : base(orderId, $"Order {orderId} was not found.") { }
}

// Caller can catch broadly OR specifically:
catch (OrderException ex)              // catches ANY order-related failure
catch (OrderAlreadyShippedException ex) // catches ONLY this specific case
```

## ⚡ Performance considerations

- Same cost profile as built-in exceptions — the overhead is in throwing, not in the class definition.
- Don't overload custom exceptions with heavy data (large objects, big collections) — they get copied around during stack unwinding/logging.

## 🚨 Common mistakes

- ❌ Inheriting from `ApplicationException` (obsolete guidance — inherit from `Exception`).
- ❌ Not implementing the standard 3 constructors — breaks compatibility with some logging/serialization tooling.
- ❌ Creating a custom exception for every tiny variation instead of using properties to differentiate (exception-class explosion).
- ❌ Putting business logic inside the exception class itself — it should just carry data, not behavior.
- ❌ Using custom exceptions for **expected, frequent** outcomes (e.g., "item not found" in a lookup) — prefer return types like `TryGetValue` or nullable/Result patterns for hot paths.

## 💡 Best practices

- Name custom exceptions ending in `Exception` (convention).
- Keep them immutable — set properties via constructor only.
- Group related custom exceptions under a common abstract base (`OrderException`) for flexible catching.
- Only create a custom exception when callers need to **programmatically react differently** — not just for nicer error text (use `Message` for that).
- Document exceptions a public method can throw (XML doc `<exception>` tag or clear naming).

## 🎤 Interview Questions

1. **Why avoid inheriting from `ApplicationException`?**
   → Microsoft's own guidance deprecated it; there's no real functional benefit over inheriting `Exception` directly, and most BCL exceptions don't use it either.
2. **When should you create a custom exception vs. reuse a built-in one?**
   → When calling code needs to **catch and react differently** based on a specific business rule violation, not just display a different message.
3. **Should you use custom exceptions for control flow (e.g., "user not found")?**
   → Generally no for high-frequency/expected paths — prefer explicit return values (`null`, `Result<T>`, `TryGetX`) since exceptions are costly and semantically meant for *exceptional* cases.
4. **What's the benefit of an abstract base like `OrderException`?**
   → Lets callers choose granularity — catch all order-related failures broadly, or one specific case narrowly.
5. **Do custom exceptions need to be serializable?**
   → For cross-AppDomain/remoting scenarios yes (legacy .NET Framework); for modern .NET (Core/5+), less critical, but still good practice for logging frameworks.

## 📝 30-second Revision Cheat Sheet

| Concept      | Key Point                                                             |
| ------------ | --------------------------------------------------------------------- |
| Inherit from | `Exception` (never `ApplicationException`)                        |
| Constructors | Implement all 3 standard ones                                         |
| Extra data   | Add properties, set via constructor, keep immutable                   |
| Naming       | End with`Exception`                                                 |
| Hierarchy    | Use abstract base for grouped catching                                |
| Don't        | Use for expected/frequent outcomes — use Result/Try patterns instead |
