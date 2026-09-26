# 02_Microservices

> **Microservices Architecture** = an application built as a collection of **small, independent services**, each owning its own data and business logic, each independently deployable, communicating over the network (via `G.Communication`'s REST/gRPC/Message Queues).

> The direct counterpart to `01_Monolithic.md`. Everything that was a strength of the monolith (in-process calls, one deployment, one database) becomes exactly what microservices trade away — in exchange for independent scaling and independent team ownership.

## 📌 What is it?

Instead of one big `Controller → BAL → DAL` application, the system is split into separate services — e.g., `OrderService`, `ProductService`, `InventoryService` — each with its **own codebase, own deployment, and often its own database**. They talk to each other over the network using the patterns from `G.Communication`.

```
Monolith (01_Monolithic.md):              Microservices:

┌─────────────────────────┐               ┌─────────────┐   ┌─────────────┐
│  Controllers → BAL → DAL │               │OrderService │   │ProductService│
│    (ONE deployable)      │               │ + own DB    │   │  + own DB    │
└───────────┬─────────────┘               └──────┬──────┘   └──────┬──────┘
            ▼                                      │  REST/gRPC/     │
      ┌──────────┐                                  │  Message Queue  │
      │SQL Server │                                 └────────┬────────┘
      └──────────┘                              ┌─────────────┐
                                                  │InventoryService│
                                                  │  + own DB      │
                                                  └─────────────┘
```

## 🤔 Why do we need it?

| Monolith limitation (from`01_Monolithic.md`)                         | How Microservices address it                                                              |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Scale the ENTIRE app even if only one feature is hot                   | Scale ONLY the busy service (e.g., just`ProductService` during a flash sale)            |
| One large codebase, hard for many teams to work in without conflict    | Each team owns a separate service, deploys independently                                  |
| One bug can bring down the entire process                              | A crash in one service doesn't directly crash others (ties into`03_Fault_Tolerance.md`) |
| One shared database becomes a bottleneck/coupling point for every team | Each service can own its own database, chosen to fit its own needs                        |

## 🌍 Real-world analogy

A **shopping mall with independent stores**, instead of one giant department store. Each store (service) hires its own staff, manages its own inventory, and can renovate or expand without asking permission from the other stores. If one store closes for renovation, the rest of the mall keeps operating. The trade-off: a customer wanting a shirt and shoes from different stores now has to walk between them (network calls) instead of finding everything under one roof.

## ⚙️ Internal working — a request crossing service boundaries

```
Client places an order
        │
        ▼
  OrderService  (owns Orders data)
        │
        │  1. Save the order in ITS OWN database
        │
        │  2. Needs product price → calls ProductService over the network (REST/gRPC)
        ▼
  ProductService  (owns Products data)
        │
        │  returns price
        ▼
  OrderService continues
        │
        │  3. Publishes "OrderPlaced" event (Pub/Sub, from G.Communication)
        ▼
  InventoryService, EmailService, AnalyticsService — each react independently
```

Notice: what used to be one in-process call chain in the monolith is now **multiple network calls across service boundaries** — this is the fundamental shift microservices introduce.

## 📊 Monolith vs Microservices — Full Comparison

| Aspect                | Monolith                                           | Microservices                                                                                                               |
| --------------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Deployment unit       | One                                                | Many, independent                                                                                                           |
| Inter-component calls | In-process method calls (fast, no network)         | Network calls — REST/gRPC/Message Queues (`G.Communication`)                                                             |
| Database              | Usually one shared DB                              | Often one database PER service                                                                                              |
| Scaling               | Scale the whole app together                       | Scale each service independently                                                                                            |
| Team structure        | One team, or several teams sharing one codebase    | Each team can own and deploy a service independently                                                                        |
| Failure isolation     | A crash can affect the whole process               | A crashing service doesn't directly crash others                                                                            |
| Complexity            | Lower (fewer moving parts operationally)           | Higher — network reliability, service discovery, distributed tracing, data consistency across services                     |
| Transactions          | Simple — one DB, ACID transactions work naturally | Hard — a transaction spanning services needs patterns like Sagas (`H.../04_CQRS.md`, `05_Event_Sourcing.md` territory) |
| Best for              | Small-to-medium apps, most projects starting out   | Large-scale systems with independent, separately-scaling domains and multiple teams                                         |

## 🚨 The hard parts microservices introduce (be honest about these!)

- **Network reliability** — a call that was a guaranteed-fast in-process method call is now a network request that can time out, fail, or arrive out of order (everything from `F.Distributed_Systems` becomes relevant here).
- **Data consistency across services** — if `OrderService` and `InventoryService` have separate databases, you can't wrap "create order + decrement stock" in one simple SQL transaction anymore. This needs distributed transaction patterns (Sagas) — a real added complexity, not a minor detail.
- **Operational overhead** — many services means many deployments, many logs to correlate (`L.Logging_and_Monitoring/03_Distributed_Tracing.md`), and more infrastructure to manage (service discovery, API gateways — next chapter).
- **Debugging a request across services is harder** — a single user action might touch five different services' logs.

## 💻 Code examples

### Basic — a microservice calling another microservice (via HTTP, per `G.Communication`)

```csharp
// Inside OrderService — needs product info from a DIFFERENT service
public class OrderService
{
    private readonly HttpClient _httpClient; // configured to call ProductService's base URL

    public async Task<Order> PlaceOrderAsync(int productId, int quantity, int userId)
    {
        // Network call to a completely separate service/deployment
        var response = await _httpClient.GetAsync($"/api/products/{productId}");
        response.EnsureSuccessStatusCode();

        var product = await response.Content.ReadFromJsonAsync<ProductDto>();

        var order = new Order
        {
            ProductId = productId,
            Quantity = quantity,
            UserId = userId,
            TotalPrice = product!.Price * quantity
        };

        _orderDal.InsertOrder(order); // OrderService's OWN database — not ProductService's

        return order;
    }
}
```

### Intermediate — resilience matters MORE here (ties into `03_Fault_Tolerance.md`)

```csharp
public async Task<ProductDto?> GetProductWithResilienceAsync(int productId)
{
    try
    {
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(2)); // never wait forever
        var response = await _httpClient.GetAsync($"/api/products/{productId}", cts.Token);
        return await response.Content.ReadFromJsonAsync<ProductDto>();
    }
    catch (Exception)
    {
        // ProductService is down/slow — degrade gracefully instead of failing the whole order flow
        return await _cache.GetAsync<ProductDto>($"product:{productId}"); // last-known cached copy
    }
}
```

> Every network call between microservices needs the discipline from `03_Fault_Tolerance.md` (timeouts, retries, fallback) — this is not optional polish, it's a core requirement of the architecture.

## ⚡ Performance considerations

- Network calls between services add real latency (milliseconds, not the near-zero cost of an in-process call) — a request touching 5 services means 5x the network overhead of a monolith's single process.
- Each service CAN be scaled independently — a huge win when load is uneven across features, but only pays off if that unevenness actually exists in your system.
- Distributed tracing (`L.Logging_and_Monitoring/03_Distributed_Tracing.md`) becomes essential, not optional, once a single user action spans multiple services.

## 🚨 Common mistakes

- ❌ Adopting microservices "because it's trendy" without an actual scaling or team-structure problem it solves — this multiplies operational complexity for no real benefit (this exact warning was flagged in `01_Monolithic.md` too — it's worth repeating).
- ❌ Sharing one database across "microservices" — this is sometimes called a "distributed monolith": you get all the network overhead of microservices with NONE of the independence benefits, since services are still coupled through the shared schema.
- ❌ Treating inter-service calls like they're as reliable as in-process calls — skipping timeouts/retries/fallback logic.
- ❌ Splitting services along the wrong boundaries (too fine-grained), causing an explosion of chatty network calls for what should be one simple operation.

## 💡 Best practices

- ✅ Only move to microservices when you have a genuine, specific problem it solves: a team-scaling problem (many teams stepping on each other) or a load-scaling problem (one feature needs vastly more capacity than others).
- ✅ Give each service its **own database** — sharing a DB across services defeats the purpose and creates a "distributed monolith."
- ✅ Apply `03_Fault_Tolerance.md` principles rigorously to every inter-service call — timeouts, retries, circuit breakers, graceful degradation.
- ✅ Invest in distributed tracing and centralized logging from day one — you cannot effectively debug microservices without it.
- ✅ Consider starting as a well-structured monolith (clean Controller → BAL → DAL layering) and only splitting into services later, once real boundaries and scaling needs become clear — this is often called "monolith-first."

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                | Answer                                                                                                                                       |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the fundamental architectural difference from a monolith?                        | Each service is independently deployable, typically owns its own database, and communicates over the network instead of via in-process calls |
| What's the biggest new complexity microservices introduce?                              | Data consistency across services (no single-database ACID transaction) and unreliable network communication between them                     |
| What is a "distributed monolith," and why is it bad?                                    | Multiple services sharing one database — you get microservices' network overhead without their independence benefits                        |
| Why is fault tolerance especially critical in microservices?                            | Every inter-service call is a network call that can fail/time out, unlike a monolith's fast, reliable in-process calls                       |
| What's the commonly recommended approach for greenfield projects regarding this choice? | "Monolith-first" — start as a well-structured monolith, split into microservices later once real scaling/team boundaries emerge             |

## 📝 30-second Revision Cheat Sheet

- Microservices = small, independently deployable services, each owning its own data, talking over the network.
- Solves what monoliths can't: independent scaling per feature, independent team deployment.
- Costs: network unreliability, cross-service data consistency, much higher operational complexity.
- Avoid a "distributed monolith" — each service needs its OWN database.
- Every inter-service call needs Fault Tolerance discipline (timeouts, retries, fallback).
- "Monolith-first" is the commonly recommended default — split later, once real boundaries emerge.
