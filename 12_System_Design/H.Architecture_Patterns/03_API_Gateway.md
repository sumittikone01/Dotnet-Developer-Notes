# 03_API_Gateway

> **API Gateway** = a single entry point that sits in front of all your microservices, routing each incoming client request to the correct backend service — so clients never need to know about your internal service topology.

> Solves a very concrete problem left open by `02_Microservices.md`: if `OrderService`, `ProductService`, and `InventoryService` are all separate deployments with separate addresses, **what URL does the client actually call?** An API Gateway is the answer.

## 📌 What is it?

Without a gateway, a client (browser, mobile app) would need to know the network address of every individual microservice and call them directly. An API Gateway removes that burden — the client calls **one** address, and the gateway handles routing internally.

```
WITHOUT a Gateway:                      WITH a Gateway:

Client ──► OrderService                 Client ──► API Gateway ──┬──► OrderService
Client ──► ProductService                                        ├──► ProductService
Client ──► InventoryService                                       └──► InventoryService
(client must know 3 addresses,          (client only knows ONE address —
 handle 3 sets of auth, etc.)            gateway handles the routing)
```

## 🤔 Why do we need it?

| Problem without a Gateway                                                     | How an API Gateway helps                                                       |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Client must know every microservice's address                                 | Client calls one single endpoint; gateway routes internally                    |
| Every service must implement its own auth/rate-limiting/logging               | Gateway centralizes these cross-cutting concerns in one place                  |
| Changing a service's internal address/port breaks clients                     | Gateway abstracts the internal topology — clients are shielded from change    |
| A mobile app might need data combined from 3 services in one call             | Gateway can aggregate multiple backend calls into one client-facing response   |
| Every service exposed directly to the public internet (larger attack surface) | Only the Gateway is public-facing; internal services stay on a private network |

## 🌍 Real-world analogy

A **hotel concierge desk**. Guests don't wander the building looking for housekeeping, room service, or the spa directly — they tell the concierge (API Gateway) what they need, and the concierge routes the request to the right department internally. The guest only ever deals with one point of contact, regardless of how many departments actually exist behind the scenes.

## ⚙️ Internal working — request routing

```
Client Request:  GET /api/orders/501

                        ▼
              ┌───────────────────┐
              │    API Gateway     │
              │  (routing rules)   │
              └─────────┬─────────┘
                        │  "/api/orders/*" → route to OrderService
                        ▼
                 OrderService (internal address, e.g. 10.0.1.5:5001)
                        │
                        ▼
                Response flows back through
                the Gateway to the Client
```

```
Routing table (conceptual):

  /api/orders/*      → OrderService
  /api/products/*    → ProductService
  /api/inventory/*   → InventoryService
```

## 📊 Common API Gateway responsibilities

| Responsibility                         | What it does                                                                                    |
| -------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Routing**                      | Directs each request to the correct backend service based on path/host                          |
| **Authentication/Authorization** | Validates tokens/API keys ONCE, at the edge, instead of in every service                        |
| **Rate Limiting**                | Enforces "max N requests per client" centrally (`J.Reliability_Patterns/04_Rate_Limiting.md`) |
| **Request Aggregation**          | Combines results from multiple backend calls into a single client response                      |
| **Load Balancing**               | Distributes requests across multiple instances of a service (`C.Load_Balancing`)              |
| **Protocol Translation**         | E.g., client speaks REST/JSON, but internally a service uses gRPC — gateway can translate      |
| **Logging & Monitoring**         | Captures a single, centralized log of all incoming traffic                                      |
| **Caching**                      | Can cache common GET responses at the edge (ties into`D.Caching`)                             |

## 🖼 Request Aggregation example (a genuinely powerful gateway capability)

```
Mobile app needs a "Product Detail Page" — requires data from 3 different services:

WITHOUT aggregation:                    WITH Gateway aggregation:

Mobile → ProductService (call 1)        Mobile → API Gateway (ONE call)
Mobile → InventoryService (call 2)              │
Mobile → ReviewService (call 3)                 ├──► ProductService
(3 round trips over a mobile network             ├──► InventoryService
 — slow and battery-draining)                    └──► ReviewService
                                          Gateway combines all 3 responses
                                          into ONE JSON payload, sent back
                                          to the mobile client in ONE round trip
```

## 💻 Code examples

### Basic — a simple gateway route configuration (using YARP, .NET's reverse proxy)

```json
// appsettings.json (YARP configuration — Microsoft's official .NET API Gateway toolkit)
{
  "ReverseProxy": {
    "Routes": {
      "orders-route": {
        "ClusterId": "orders-cluster",
        "Match": { "Path": "/api/orders/{**catch-all}" }
      },
      "products-route": {
        "ClusterId": "products-cluster",
        "Match": { "Path": "/api/products/{**catch-all}" }
      }
    },
    "Clusters": {
      "orders-cluster": {
        "Destinations": {
          "destination1": { "Address": "http://order-service:5001/" }
        }
      },
      "products-cluster": {
        "Destinations": {
          "destination1": { "Address": "http://product-service:5002/" }
        }
      }
    }
  }
}
```

```csharp
// Program.cs — the Gateway's OWN codebase, separate from any business service
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();
app.MapReverseProxy();
app.Run();
```

### Intermediate — centralizing authentication at the gateway

```csharp
// Program.cs (Gateway)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-auth-provider.com";
        options.Audience = "your-api";
    });

app.UseAuthentication(); // validated ONCE, here, at the edge
app.UseAuthorization();
app.MapReverseProxy();

// Individual services (OrderService, ProductService, etc.) can now trust
// that any request reaching them has ALREADY been authenticated by the Gateway —
// they don't need to re-implement JWT validation themselves.
```

### Practical — a simple request-aggregation endpoint

```csharp
// A custom Gateway endpoint that fans out to multiple services and merges results
app.MapGet("/api/product-detail/{id}", async (int id, HttpClient client) =>
{
    var productTask = client.GetFromJsonAsync<ProductDto>($"http://product-service/api/products/{id}");
    var inventoryTask = client.GetFromJsonAsync<InventoryDto>($"http://inventory-service/api/inventory/{id}");
    var reviewsTask = client.GetFromJsonAsync<List<ReviewDto>>($"http://review-service/api/reviews/{id}");

    await Task.WhenAll(productTask, inventoryTask, reviewsTask); // parallel calls, not sequential

    return Results.Ok(new
    {
        Product = productTask.Result,
        Inventory = inventoryTask.Result,
        Reviews = reviewsTask.Result
    });
});
```

## ⚡ Performance considerations

- The Gateway itself must be **highly available and fast** — every single request passes through it, so it can become a new bottleneck or Single Point of Failure (`03_Fault_Tolerance.md`) if not properly scaled/replicated.
- Request aggregation reduces round trips for clients (especially valuable for mobile), but adds a bit of processing/latency at the gateway itself doing the fan-out and merging.
- Centralized auth/rate-limiting at the gateway avoids redundant work in every downstream service — a real efficiency win, not just a convenience.

## 🚨 Common mistakes

- ❌ Treating the Gateway as a single point of failure by running only one instance — it needs the same redundancy/load-balancing discipline as any other critical service.
- ❌ Putting heavy business logic inside the Gateway — it should stay focused on routing/cross-cutting concerns, not become a second monolith in disguise.
- ❌ Forgetting that the Gateway adds a network hop of its own — every request now goes Client → Gateway → Service, not directly Client → Service.

## 💡 Best practices

- ✅ Keep the Gateway focused on cross-cutting concerns (routing, auth, rate limiting, logging) — leave business logic to the actual services.
- ✅ Run multiple Gateway instances behind a load balancer (`C.Load_Balancing`) — don't let it become a new Single Point of Failure.
- ✅ Use request aggregation deliberately for client types that benefit most (e.g., mobile apps on slow networks) rather than everywhere by default.
- ✅ Centralize authentication at the gateway so individual services can trust incoming requests instead of re-validating tokens everywhere.

## 🎤 Interview Quick-Fire Q&A

| Question                                                  | Answer                                                                                                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| What problem does an API Gateway solve for microservices? | It gives clients a single entry point, so they don't need to know about or call each individual microservice directly                      |
| Name three common responsibilities of an API Gateway.     | Routing, Authentication, Rate Limiting (also valid: Request Aggregation, Load Balancing, Logging, Caching)                                 |
| What is "request aggregation"?                            | The gateway calling multiple backend services and combining their responses into a single response for the client                          |
| Why must the Gateway itself be highly available?          | Every client request passes through it — if it goes down, the entire system becomes unreachable, even if all backend services are healthy |
| What should NOT be put inside an API Gateway?             | Heavy business logic — it should stay focused on cross-cutting concerns like routing, auth, and rate limiting                             |

## 📝 30-second Revision Cheat Sheet

- API Gateway = single entry point that routes client requests to the correct microservice.
- Centralizes cross-cutting concerns: auth, rate limiting, logging, routing, load balancing.
- Request aggregation = combining multiple backend calls into one client-facing response.
- Must be highly available and horizontally scaled itself — it's now a critical path for EVERY request.
- Keep business logic OUT of the gateway — it's a routing/cross-cutting layer, not a service.

---

✅ Progress note: **H.Architecture_Patterns** — 01–03 done. Next up: **04_CQRS.md** and **05_Event_Sourcing.md**.
