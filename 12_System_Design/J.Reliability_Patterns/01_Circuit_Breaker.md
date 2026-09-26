# 01_Circuit_Breaker

> **Circuit Breaker** = a pattern that **stops calling a dependency that's clearly failing**, immediately returning an error/fallback instead of endlessly waiting or retrying — protecting the caller from wasting resources on a lost cause.

> New chapter: **J.Reliability_Patterns**. This is the deep dive on a pattern that was flagged but not fully explained back in `F.Distributed_Systems/03_Fault_Tolerance.md`'s technique table. It also directly follows on from `H.../02_Microservices.md`'s warning: "every inter-service call is a network call that can fail."

## 📌 What is it?

Named after an **electrical circuit breaker** — a real one trips (opens) when there's a dangerous surge, cutting power to prevent damage, rather than letting the surge keep flowing. The software version does the same thing for a **failing dependency**: after enough failures, it "trips" and stops sending requests to that dependency for a while, rather than letting every new request hang or fail slowly.

## 🤔 Why do we need it?

Recall the **retry-storm** problem hinted at in `03_Fault_Tolerance.md`:

```
WITHOUT a circuit breaker:

  ProductService is down.
  1,000 incoming requests/sec ALL still try to call ProductService.
  Each one waits for a timeout (e.g., 5 seconds) before failing.
  → Threads/connections pile up waiting on a service that's already dead.
  → The CALLING service ALSO starts to slow down/crash from resource exhaustion.
  → One failure CASCADES into a second failure. This is called a "cascading failure."
```

A circuit breaker prevents this: once it detects the dependency is failing, it **stops trying immediately** (fails fast) instead of piling up slow, doomed requests.

## 🌍 Real-world analogy

Exactly the household electrical circuit breaker it's named after: when there's a short circuit or overload, the breaker **trips** and cuts power immediately — protecting your house's wiring from damage — rather than letting electricity keep flowing into a dangerous situation. You don't keep plugging in more appliances hoping it'll work; you fix the underlying problem, then reset the breaker.

## 📊 The Three States

| State               | Behavior                                                                                     | Transition                                                                         |
| ------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Closed**    | Normal operation — requests flow through to the dependency as usual                         | If failures exceed a threshold → moves to**Open**                           |
| **Open**      | Requests immediately fail/fallback WITHOUT even attempting to call the dependency            | After a cooldown period → moves to**Half-Open**                             |
| **Half-Open** | A small number of test requests are allowed through to check if the dependency has recovered | If they succeed → back to**Closed**. If they fail → back to **Open** |

## 🖼 State Machine Diagram

```
                failures exceed threshold
        ┌─────────────────────────────────────┐
        │                                       ▼
   ┌─────────┐                            ┌──────────┐
   │ CLOSED  │                            │   OPEN   │
   │(normal) │                            │(fail fast)│
   └────┬────┘                            └────┬─────┘
        ▲                                       │
        │                              cooldown timer expires
        │                                       ▼
        │                              ┌───────────────┐
        └────── test requests succeed ─│  HALF-OPEN     │
                                        │ (trial requests)│
                                        └───────┬────────┘
                                                 │
                                       test requests fail
                                                 │
                                                 ▼
                                          back to OPEN
```

## ⚙️ Internal working — a request's journey through each state

```
CLOSED state:
  Request → Circuit Breaker → calls ProductService normally
  ProductService fails → failure count++
  Failure count crosses threshold (e.g., 5 failures in 10 seconds) → TRIP to OPEN

OPEN state:
  Request → Circuit Breaker → IMMEDIATELY returns error/fallback
                               (ProductService is NEVER actually called — fails FAST)
  After cooldown (e.g., 30 seconds) → move to HALF-OPEN

HALF-OPEN state:
  Request → Circuit Breaker → allows ONE test request through to ProductService
  If it SUCCEEDS → assume recovered → back to CLOSED (resume normal traffic)
  If it FAILS    → assume still broken → back to OPEN (wait another cooldown)
```

## 📊 Circuit Breaker vs plain Retry (`03_Fault_Tolerance.md`) — how they differ and combine

| Aspect                           | Retry (alone)                                                 | Circuit Breaker                                                  |
| -------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------- |
| Response to a failing dependency | Keeps trying again (with backoff)                             | Stops trying entirely once tripped                               |
| Risk if dependency is TRULY down | Retries pile up, waste resources, can worsen the outage       | Fails fast — protects both caller and the struggling dependency |
| Best used                        | For transient, short-lived failures (a single dropped packet) | For sustained failures (dependency is genuinely down/overloaded) |

> **In practice, these are combined**: retry a failed call a couple of times (for genuinely transient blips), but wrap the whole thing in a circuit breaker so that if failures are sustained, the system stops retrying altogether and fails fast instead.

## 💻 Code examples

### Basic — using Polly (the standard .NET resilience library) for a circuit breaker

```csharp
// Program.cs — registering an HttpClient with a circuit breaker policy
builder.Services.AddHttpClient<ProductServiceClient>()
    .AddTransientHttpErrorPolicy(policyBuilder =>
        policyBuilder.CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 5,       // trip after 5 consecutive failures
            durationOfBreak: TimeSpan.FromSeconds(30)));  // stay OPEN for 30 seconds before Half-Open
```

### Intermediate — combining Retry + Circuit Breaker (the standard real-world pairing)

```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, attempt => TimeSpan.FromMilliseconds(200 * attempt)); // exponential-ish backoff

var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        exceptionsAllowedBeforeBreaking: 5,
        durationOfBreak: TimeSpan.FromSeconds(30));

// Wrap retry INSIDE circuit breaker — retries happen only while the circuit is CLOSED
var combinedPolicy = Policy.WrapAsync(retryPolicy, circuitBreakerPolicy);

builder.Services.AddHttpClient<ProductServiceClient>()
    .AddPolicyHandler(combinedPolicy);
```

### Practical — handling the "circuit is open" case gracefully

```csharp
public async Task<Product?> GetProductWithFallbackAsync(int id)
{
    try
    {
        return await _productServiceClient.GetProductByIdAsync(id);
    }
    catch (BrokenCircuitException)
    {
        // The circuit is OPEN — ProductService wasn't even called this time.
        // Fall back to cached data instead of failing the whole request (ties into D.Caching)
        return await _cache.GetAsync<Product>($"product:{id}");
    }
}
```

## ⚡ Performance considerations

- Failing fast (Open state) is dramatically cheaper than waiting for timeouts on every request during an outage — frees up threads/connections for the calling service to keep serving other traffic.
- Choosing the right thresholds matters: too sensitive (trips on 1-2 failures) causes unnecessary outages during brief blips; too lenient (trips only after 100 failures) delays protection during a real outage. Tune based on observed traffic and failure patterns.

## 🚨 Common mistakes

- ❌ Using retries alone, with no circuit breaker, against a dependency that's genuinely down — causes exactly the retry-storm/cascading-failure scenario this pattern exists to prevent.
- ❌ Setting the cooldown (Open → Half-Open) too short — repeatedly hammering a still-recovering service with test requests can prevent it from ever stabilizing.
- ❌ Not having a sensible fallback for the Open state — failing fast is only half the win; pair it with graceful degradation (`03_Fault_Tolerance.md`) rather than just surfacing a raw error to the end user.

## 💡 Best practices

- ✅ Wrap every external/inter-service call (especially in a microservices architecture, `H.../02_Microservices.md`) with a circuit breaker, not just a plain try/catch.
- ✅ Combine circuit breakers with retries (retry for brief transient issues, circuit breaker for sustained ones) — this is the standard, recommended pairing.
- ✅ Always define a meaningful fallback for the Open state (cached data, a default value, a friendly degraded response) rather than just propagating a raw exception.
- ✅ Monitor circuit breaker state transitions (Closed → Open events) as a first-class metric — a circuit tripping is a valuable early warning signal of a dependency's health.

## 🎤 Interview Quick-Fire Q&A

| Question                                                  | Answer                                                                                                                                                   |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What problem does a Circuit Breaker solve?                | Prevents a caller from repeatedly hitting a failing dependency, avoiding wasted resources and cascading failures                                         |
| Name the three states of a Circuit Breaker.               | Closed (normal), Open (fail fast), Half-Open (testing recovery)                                                                                          |
| What triggers the transition from Closed to Open?         | The number of failures exceeding a configured threshold within a given window                                                                            |
| How does a circuit breaker recover from the Open state?   | After a cooldown period, it moves to Half-Open and allows a few test requests through; success returns it to Closed, failure sends it back to Open       |
| How do Retry and Circuit Breaker typically work together? | Retry handles brief, transient failures with a few attempts; the circuit breaker wraps around it to stop all attempts entirely if failures are sustained |

## 📝 30-second Revision Cheat Sheet

- Circuit Breaker = stop calling a failing dependency instead of endlessly retrying/waiting — "fail fast."
- Three states: Closed (normal) → Open (fail fast, after too many failures) → Half-Open (test recovery) → back to Closed or Open.
- Prevents cascading failures / retry storms during a real outage.
- Typically combined WITH retries: retry for brief blips, circuit breaker for sustained failures.
- Always pair the Open state with a graceful fallback (cache, default value) — not just a raw error.
