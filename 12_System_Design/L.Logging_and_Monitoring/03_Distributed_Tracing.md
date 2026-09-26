# 🔍 Distributed Tracing

## 📌 What is it?

**Distributed Tracing** tracks a **single request's full journey** as it flows through multiple services in a distributed/microservices system, recording how long each step took and where it went.

## 🤔 Why do we need it?

In a monolith, if a request is slow, you check one app's logs. In **microservices**, one user request might touch 10+ services:

```
User Request → API Gateway → Auth Service → Order Service → Inventory Service → Payment Service → DB
```

If this request takes 3 seconds, **which service** caused the delay? Logs alone won't tell you — each service only knows about itself. Distributed Tracing stitches together the **entire path** into one visual timeline.

## 🌍 Real-world analogy

Tracking a **package delivery** 📦 across multiple couriers — local pickup, regional hub, international flight, customs, local delivery. A tracking number lets you see the exact timestamp at each stage, and instantly spot **where** it got stuck (e.g. sat at customs for 2 days).

## ⚙️ Core Concepts

| Term                         | Meaning                                                                    |
| ---------------------------- | -------------------------------------------------------------------------- |
| **Trace**              | The full journey of one request across all services                        |
| **Span**               | One unit of work within that journey (e.g. "Auth Service validated token") |
| **Trace ID**           | A unique ID shared by all spans belonging to the same request              |
| **Parent/Child Spans** | Spans can be nested — a span can trigger other spans                      |

## 🖼 Trace Timeline Example

```
Trace ID: abc-123
├─ Span: API Gateway              [0ms ─────────────── 3000ms]
│   ├─ Span: Auth Service          [0ms ── 200ms]
│   ├─ Span: Order Service         [200ms ──────── 2700ms]
│   │   ├─ Span: Inventory Check   [200ms ── 400ms]
│   │   └─ Span: Payment Service   [400ms ─────────── 2700ms] 🐌 ← bottleneck found!
│   └─ Span: Response formatting   [2700ms ── 3000ms]
```

Instantly visible: **Payment Service** is where 2.3 of the 3 seconds went — that's where to investigate.

## 📊 How Trace Context Propagates

Each service passes the **Trace ID** (and parent span info) to the next service via HTTP headers, so all spans can later be reassembled into one trace.

```
Header example (W3C Trace Context standard):
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
              │  └──────── Trace ID ──────────┘ └─ Span ID ──┘
           version
```

## 💻 Code Example (ASP.NET Core with OpenTelemetry)

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSource("OrderService")
        .AddOtlpExporter()  // sends traces to Jaeger/Zipkin/etc.
    );

// Custom span example:
using var activity = MyActivitySource.StartActivity("ProcessPayment");
activity?.SetTag("orderId", orderId);
// ... payment logic
```

## 📊 Popular Tracing Tools

| Tool                              | Type                                                |
| --------------------------------- | --------------------------------------------------- |
| **Jaeger**                  | Open-source, CNCF project                           |
| **Zipkin**                  | Open-source, one of the earliest tracing tools      |
| **OpenTelemetry**           | Vendor-neutral standard/SDK (industry standard now) |
| **Datadog APM / New Relic** | Commercial, full observability suites               |

## 🚨 Common Mistakes

- ❌ Not propagating the Trace ID across service boundaries (async messaging, background jobs) — breaks the trace chain.
- ❌ Tracing every single request in high-traffic systems without **sampling** — huge storage/performance cost.
- ❌ Treating tracing as separate from logging/metrics instead of correlating them together (a good setup links a log line to its Trace ID).

## 💡 Best Practices

- Use **OpenTelemetry** — the vendor-neutral standard — so you're not locked into one tracing backend.
- Apply **sampling** (e.g. trace 1% of requests, or 100% of error requests) to control cost at scale.
- Always propagate trace context across **all** boundaries — HTTP calls, message queues, background jobs.
- Correlate traces with logs by including the Trace ID in log lines.

## 🎤 Interview Questions

1. What problem does distributed tracing solve that centralized logging alone cannot?
2. What is a "span" and how does it relate to a "trace"?
3. How does trace context get passed between services that don't share memory?
4. Why would you use sampling in a distributed tracing system, and how would you choose a sampling rate?

## 📝 30-second Revision Cheat Sheet

- Distributed Tracing = tracks one request's full path across multiple services.
- Trace = whole journey; Span = one step within it.
- Trace ID propagates via headers (e.g. `traceparent`) across service calls.
- OpenTelemetry = industry-standard tracing framework.
- Use sampling at scale; always correlate traces with logs.
