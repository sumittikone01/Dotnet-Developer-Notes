
# 🆕 Interface Default Methods (C# 8+)

## 📌 What is it?

> Since C# 8, interfaces CAN provide a **default implementation** for a method — implementing classes get this behavior automatically unless they choose to override it.

This was a major shift from the "interfaces = zero implementation" rule of earlier C# versions.

---

## 🤔 Why do we need it?

**Versioning problem it solves:** Before C# 8, adding a new method to an existing interface would **break every class** that implements it (they'd all need to implement the new method immediately). Default interface methods let you extend an interface WITHOUT breaking existing implementers.

---

## 💻 Code Examples

**Basic:**

```csharp
public interface ILogger
{
    void Log(string message);

    // Default implementation (C# 8+) — implementers don't HAVE to override this
    void LogError(string message) => Log($"[ERROR] {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
    // LogError not implemented — uses the interface's default automatically
}

ILogger logger = new ConsoleLogger();
logger.Log("Starting process");        // "Starting process"
logger.LogError("Something failed");   // "[ERROR] Something failed" — uses default
```

**Intermediate — overriding the default when needed:**

```csharp
public class FileLogger : ILogger
{
    public void Log(string message) => File.AppendAllText("log.txt", message + "\n");

    public void LogError(string message) => Log($"[CRITICAL ERROR] {message}");   // custom override
}
```

**Practical — versioning scenario (the real reason this feature exists):**

```csharp
// Original interface, already implemented by many classes across a large codebase
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}

// New requirement: add refund support WITHOUT breaking existing implementers
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);

    void RefundPayment(decimal amount) =>    // default — old implementers keep compiling fine
        throw new NotImplementedException("Refunds not supported by this processor");
}
```

---

## 🚨 Common Mistakes

- ❌ Assuming default interface methods make interfaces work exactly like abstract classes — interfaces STILL cannot have instance fields
- ❌ Calling a default method through a class reference instead of interface reference in some scenarios — default implementations are only accessible via the INTERFACE type, not always via the concrete class directly (subtle nuance, verify with actual project's C# version)
- ❌ Overusing this feature for general business logic — it's primarily meant for API versioning/backward compatibility, not as a replacement for abstract classes

---

## 🎤 Interview Questions

| Question                                                        | Key Point                                                                               |
| --------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| What problem do default interface methods solve?                | Allow adding new members to an interface without breaking existing implementing classes |
| Since which C# version are default interface methods supported? | C# 8.0                                                                                  |
| Do interfaces with default methods now have fields?             | No — still no instance fields, even with default methods                               |

---

## 📝 30-Second Revision Cheat Sheet

- C# 8+ allows interfaces to provide default method implementations
- Solves the "breaking change" problem when extending interfaces used by many classes
- Implementing classes can still override the default if needed
- Interfaces still cannot have instance fields, even with this feature
