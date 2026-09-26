# 📝 Logging Basics

## 📌 What is it?

**Logging** is the practice of recording events that happen inside a system while it runs — errors, warnings, requests, state changes — so humans can **understand what happened**, especially after something goes wrong.

## 🤔 Why do we need it?

- **Debugging**: When something breaks in production, logs are often the *only* window into what actually happened.
- **Auditing**: Track who did what, and when.
- **Monitoring health**: Feed dashboards/alerts (see `02_Monitoring_and_Alerting.md`).
- **Understanding behavior at scale**: You can't attach a debugger to a production server serving millions of requests — logs are your eyes there.

## 🌍 Real-world analogy

An aircraft's **black box flight recorder** ✈️. You hope you never need it, but when something goes wrong, it's the only reliable record of exactly what happened, in what order, right before the failure.

## 📊 Log Levels (from least to most severe)

| Level                    | When to use                                    | Example                                  |
| ------------------------ | ---------------------------------------------- | ---------------------------------------- |
| **Trace**          | Extremely detailed, step-by-step internal flow | "Entering method CalculateTotal()"       |
| **Debug**          | Useful during development/troubleshooting      | "Cache miss for key: user_123"           |
| **Information**    | Normal but significant events                  | "User 123 logged in"                     |
| **Warning**        | Something unexpected, but not breaking         | "API response took 4.5s (threshold: 2s)" |
| **Error**          | An operation failed                            | "Failed to save order: DB timeout"       |
| **Critical/Fatal** | System-level failure, app may be unusable      | "Database connection pool exhausted"     |

> 💡 In production, Trace/Debug are usually **disabled** (too noisy, hurts performance) — typically only Information and above are logged, with levels dynamically raised temporarily when investigating an issue.

## ⚙️ Structured vs Unstructured Logging

| Type                        | Example                                              | Machine-readable?                    |
| --------------------------- | ---------------------------------------------------- | ------------------------------------ |
| **Unstructured**      | `"User 123 logged in at 10:05am"`                  | ❌ Hard to query/filter              |
| **Structured (JSON)** | `{"event":"login","userId":123,"time":"10:05:00"}` | ✅ Easy to search, filter, aggregate |

**Structured logging is the modern standard** — it lets tools like Elasticsearch, Splunk, or Seq query logs like a database (e.g. "show me all `login` events for `userId=123` in the last hour").

## 💻 Code Example (ASP.NET Core with `ILogger` — structured logging)

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void PlaceOrder(int userId, int orderId)
    {
        _logger.LogInformation("Order {OrderId} placed by user {UserId}", orderId, userId);

        try
        {
            // ... business logic
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to place order {OrderId} for user {UserId}", orderId, userId);
            throw;
        }
    }
}
```

Note: `{OrderId}` and `{UserId}` are **structured fields**, not just string-interpolated text — the logging framework stores them as separate searchable properties, not just baked into a sentence.

## 🖼 Centralized Logging Flow (multi-server systems)

```
┌────────────┐  ┌────────────┐  ┌────────────┐
│ Server A   │  │ Server B   │  │ Server C   │
│  logs ──┐  │  │  logs ──┐  │  │  logs ──┐  │
└─────────│──┘  └─────────│──┘  └─────────│──┘
          ▼               ▼               ▼
     ┌───────────────────────────────────────┐
     │     Log Aggregator (e.g. Fluentd)     │
     └───────────────────┬───────────────────┘
                          ▼
              ┌───────────────────────┐
              │  Centralized Store     │
              │ (Elasticsearch/Splunk) │
              └───────────┬───────────┘
                          ▼
                  ┌───────────────┐
                  │   Dashboard    │
                  │   (Kibana)     │
                  └───────────────┘
```

Without centralization, debugging an issue across 50 servers means manually checking 50 different log files — centralized logging lets you search **all servers at once**.

## 🚨 Common Mistakes

- ❌ Logging sensitive data (passwords, credit card numbers, tokens) in plain text.
- ❌ Excessive logging in hot paths — hurts performance and generates noise that buries real signal.
- ❌ Unstructured string-only logs — hard to query at scale.
- ❌ No correlation ID — impossible to trace one request's full journey across multiple services (see `03_Distributed_Tracing.md`).
- ❌ Logging and forgetting — no one ever reviews or alerts on the logs being generated.

## 💡 Best Practices

- Use **structured logging** always in production systems.
- Include a **Correlation/Trace ID** per request so all related log lines can be tied together across services.
- Never log secrets/PII — mask or omit sensitive fields.
- Centralize logs from all instances into one searchable system.
- Set appropriate log levels per environment (verbose in dev, minimal + errors in prod).

## 🎤 Interview Questions

1. Why is structured logging preferred over plain string logging in production systems?
2. What is a Correlation ID and why is it critical in a microservices architecture?
3. What's the difference between `Warning` and `Error` log levels, and how would you decide which to use?
4. How would you avoid logging sensitive user data while still having useful logs?

## 📝 30-second Revision Cheat Sheet

- Logging = recording what happened in the system, for debugging & auditing.
- Log Levels: Trace < Debug < Info < Warning < Error < Critical.
- Prefer **structured (JSON)** logs over plain text.
- Use a **Correlation ID** to trace one request across multiple services.
- Centralize logs (Elasticsearch/Splunk) for multi-server searchability.
