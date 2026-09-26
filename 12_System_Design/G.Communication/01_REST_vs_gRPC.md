# 01_REST_vs_gRPC

> **REST** and **gRPC** are two different styles/protocols for how services **talk to each other over a network** — REST uses HTTP + (usually) JSON in a resource-oriented style; gRPC uses HTTP/2 + Protocol Buffers in a function-call-oriented style.

> New chapter: **G.Communication**. Everything so far has been about scaling and protecting *data* (caching, replication, sharding) and *coordination* (consensus). This chapter shifts to **how services actually exchange messages** — the wire format and calling convention.

## 📌 What is it?

|                | REST                                              | gRPC                                                               |
| -------------- | ------------------------------------------------- | ------------------------------------------------------------------ |
| Full form      | REpresentational State Transfer                   | **g**RPC **R**emote **P**rocedure **C**all |
| Core idea      | Interact with "resources" via standard HTTP verbs | Directly "call a method" on a remote service, as if it were local  |
| Transport      | HTTP/1.1 (usually)                                | HTTP/2 (always)                                                    |
| Payload format | JSON (usually, human-readable text)               | Protocol Buffers ("protobuf" — compact binary)                    |
| Contract       | Often informal (docs, OpenAPI/Swagger)            | Formal`.proto` file — strict, code-generated                    |

## 🤔 Why do we need to know both?

You'll pick one (or mix both) based on **who's calling whom**:

| Scenario                                                                | Better fit                                                                          |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Public API consumed by browsers, mobile apps, third-party developers    | **REST** — universally supported, human-readable, easy to debug in a browser |
| Internal service-to-service calls (microservices talking to each other) | **gRPC** — faster, strongly-typed, lower overhead                            |
| Need simple, cacheable, widely-understood API                           | **REST**                                                                      |
| Need high-throughput, low-latency, or streaming communication           | **gRPC**                                                                      |

## 🌍 Real-world analogy

**REST** is like sending a **formal letter** addressed to a department: "To: /orders/123 — Please tell me your status" (you address a *resource*, using a standard vocabulary of GET/POST/PUT/DELETE). **gRPC** is like **picking up the phone and directly asking a specific person to run a specific task**: "Please run `GetOrderStatus(123)` for me" — direct, structured, and both sides already agree exactly what that function looks like.

## ⚙️ How REST works (recap, resource-oriented)

```
Client                          Server
  │  GET /api/products/5           │
  │ ───────────────────────────►   │
  │                                 │ (Controller → BAL → DAL → Stored Procedure)
  │  ◄─────────────────────────    │
  │  200 OK { "id":5, "name":"..." }│   ← JSON body
```

- URL identifies **what** (a resource: `/products/5`)
- HTTP method identifies **the action** (GET = read, POST = create, PUT = update, DELETE = remove)
- Covered in full depth in `I.API_Design/01_REST_API_Principles.md`

## ⚙️ How gRPC works (function-call-oriented)

```
1. Define the contract in a .proto file (shared by client and server):

   service ProductService {
     rpc GetProduct (ProductRequest) returns (ProductReply);
   }

2. Code generator produces strongly-typed client + server stubs in your language

3. Client calls it like a LOCAL method:

   ProductReply reply = client.GetProduct(new ProductRequest { Id = 5 });

   ...but under the hood, this is actually a network call over HTTP/2,
   with the request/response serialized as compact Protocol Buffers binary.
```

```
Client Stub                    Network                    Server Stub
(generated code)          (HTTP/2 + protobuf)          (generated code)
     │                            │                            │
     │  GetProduct(req) ───────────────────────────────────►   │
     │                            │                            │ → actual method runs
     │  ◄─────────────────────────────────────── ProductReply  │
     │                            │                            │
```

## 📊 Full Comparison Table

| Aspect                       | REST                                                                | gRPC                                                                                |
| ---------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Payload size                 | Larger (JSON is text, verbose)                                      | Smaller (protobuf is compact binary) — often 5-10x smaller                         |
| Speed                        | Slower (text parsing, HTTP/1.1 overhead)                            | Faster (binary, HTTP/2 multiplexing)                                                |
| Human-readability            | High — you can read a JSON response directly                       | Low — binary, needs tooling to inspect                                             |
| Browser support              | Native (any browser/fetch/XHR)                                      | Limited — needs`grpc-web` + a proxy for browsers                                 |
| Streaming support            | Limited (polling, or separate tech like WebSockets/SSE)             | Native — supports client-streaming, server-streaming, and bidirectional streaming  |
| Contract enforcement         | Loose — relies on documentation/OpenAPI discipline                 | Strict —`.proto` file is the single source of truth, enforced by code generation |
| Caching                      | Easy — standard HTTP caching (GET + cache headers) works naturally | Harder — not built around HTTP semantics the same way                              |
| Tooling maturity / ecosystem | Extremely mature, universal                                         | Growing fast, strong in microservices/cloud-native world                            |
| Best for                     | Public APIs, browser clients, simplicity                            | Internal microservice-to-microservice calls, high performance needs                 |

## 💻 Code examples

### REST — a standard ASP.NET Core Web API controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductService _service;

    [HttpGet("{id}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetProductById(id); // Controller → BAL → DAL → Stored Procedure
        if (product == null) return NotFound();

        return Ok(product); // serialized to JSON automatically
    }
}
```

```
Client call (any HTTP client, even a browser):
   GET https://api.example.com/api/products/5
   → 200 OK
     { "id": 5, "name": "Widget", "price": 19.99 }
```

### gRPC — defining the contract and implementing the service

```protobuf
// products.proto
syntax = "proto3";

service ProductService {
  rpc GetProduct (ProductRequest) returns (ProductReply);
}

message ProductRequest {
  int32 id = 1;
}

message ProductReply {
  int32 id = 1;
  string name = 2;
  double price = 3;
}
```

```csharp
// ProductGrpcService.cs — generated base class implemented here
public class ProductGrpcService : ProductService.ProductServiceBase
{
    private readonly ProductDAL _dal;

    public override Task<ProductReply> GetProduct(ProductRequest request, ServerCallContext context)
    {
        var product = _dal.GetProductById(request.Id); // same BAL/DAL underneath!

        return Task.FromResult(new ProductReply
        {
            Id = product.Id,
            Name = product.Name,
            Price = (double)product.Price
        });
    }
}
```

```csharp
// Client side — feels like calling a local method
var client = new ProductService.ProductServiceClient(channel);
ProductReply reply = await client.GetProductAsync(new ProductRequest { Id = 5 });
Console.WriteLine(reply.Name);
```

> Notice: **the underlying BAL/DAL/stored-procedure architecture doesn't change at all** — REST vs gRPC is purely about the *transport/contract layer* on top of your existing business logic.

## ⚡ Performance considerations

- gRPC's binary protobuf + HTTP/2 multiplexing typically gives **significantly lower latency and bandwidth usage** than REST/JSON — a meaningful win for high-volume internal service-to-service traffic.
- REST's JSON is heavier on the wire but far easier to debug, log, and inspect (just read the text) — a real productivity trade-off, especially during development/troubleshooting.
- gRPC's native streaming support is a genuine capability gap for REST — real-time/bidirectional data (e.g., live order tracking) is much more natural in gRPC.

## 🚨 Common mistakes

- ❌ Choosing gRPC for a public API consumed directly by web browsers — browser gRPC support is limited/awkward (`grpc-web` + proxy needed); REST is the natural fit there.
- ❌ Choosing REST for extremely high-throughput internal microservice calls where the JSON/HTTP1.1 overhead becomes a measurable bottleneck.
- ❌ Assuming you must pick only one — many real systems use **REST for public-facing APIs** and **gRPC for internal service-to-service** calls, side by side.

## 💡 Best practices

- ✅ Default to REST for anything public-facing, browser-consumed, or where debuggability/simplicity matters most.
- ✅ Reach for gRPC for internal microservice communication where performance, strong typing, and streaming matter.
- ✅ Keep your BAL/DAL layer transport-agnostic (as shown above) — the same business logic can be exposed via REST AND gRPC without duplication.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                         | Answer                                                                                                |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| What transport protocol does gRPC always use?                                    | HTTP/2                                                                                                |
| What serialization format does gRPC use, and why is it more efficient than JSON? | Protocol Buffers — a compact binary format, smaller and faster to (de)serialize than text-based JSON |
| Why is REST generally preferred for public APIs?                                 | Universal browser/tooling support, human-readable payloads, easier debugging                          |
| What capability does gRPC have that REST doesn't natively support well?          | Native streaming (client, server, and bidirectional streaming)                                        |
| Can REST and gRPC be used together in the same system?                           | Yes — commonly REST for public-facing APIs and gRPC for internal microservice-to-microservice calls  |

## 📝 30-second Revision Cheat Sheet

- REST = resource-oriented, HTTP + JSON, human-readable, best for public APIs.
- gRPC = function-call-oriented, HTTP/2 + Protobuf (binary), best for internal microservices.
- gRPC is faster/smaller on the wire and supports native streaming; REST is easier to debug and universally supported (including browsers).
- Contract: REST is loosely documented (OpenAPI); gRPC is strictly defined via a `.proto` file with code generation.
- Common real-world pattern: REST for external/public, gRPC for internal service-to-service.
