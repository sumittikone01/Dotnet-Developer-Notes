# 01_REST_API_Principles

> **REST (REpresentational State Transfer)** = an architectural style for designing APIs around **resources** (nouns), manipulated through a small, standard set of operations (HTTP verbs) — with no server-side memory of past requests.

> New chapter: **I.API_Design**. `G.Communication/01_REST_vs_gRPC.md` compared REST at a high level against gRPC; this chapter goes deep on **what makes an API actually "RESTful"** — the concrete design rules.

## 📌 What is it?

REST isn't a protocol or a library — it's a set of **architectural constraints**. An API that follows them is called "RESTful." The central idea: model your API around **resources** (things — `products`, `orders`, `users`), not actions (verbs like `getProduct`, `deleteOrderById`).

```
NOT RESTful (action-oriented):        RESTful (resource-oriented):

POST /getProduct?id=5                 GET /products/5
POST /deleteOrder                     DELETE /orders/12
POST /updateUserEmail                 PATCH /users/42
```

## 🤔 Why do we need it?

| Problem with ad-hoc APIs                                                       | How REST principles help                                                                     |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| Every developer invents their own URL/verb conventions                         | A shared, predictable vocabulary (resources + standard HTTP verbs)                           |
| Hard to guess what an endpoint does without reading docs                       | `GET /orders/5` is self-explanatory — resource + intent are baked into the request itself |
| Server-side session state makes scaling harder (ties into`C.Load_Balancing`) | REST's statelessness means any server can handle any request — trivial to load-balance      |
| No standard way to reason about caching                                        | REST leans on standard HTTP caching semantics (`GET` is cacheable by default)              |

## 🌍 Real-world analogy

A **well-organized library's card catalog system**. Every book (resource) has a predictable address (call number). You don't need a different, custom procedure to find each book — the same standard method (look up the call number, go to the shelf) works for every single book in the building. REST does the same for data: one standard set of operations (GET, POST, PUT, DELETE) works uniformly across every resource in your API.

## 📊 The 6 Core REST Constraints

| Constraint                              | Meaning                                                                                                                                                         |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Client-Server**                 | Client (UI) and Server (API) are separate, independently evolvable — your Kendo UI frontend doesn't need to know how the DAL/stored procedures work internally |
| **Statelessness**                 | Every request contains ALL the information needed to process it — the server stores no session/conversation state between requests                             |
| **Cacheability**                  | Responses must explicitly indicate whether they can be cached (ties directly into`D.Caching`)                                                                 |
| **Uniform Interface**             | A consistent, standard way to interact with resources (URLs + HTTP verbs) — the defining REST constraint                                                       |
| **Layered System**                | Client can't tell (and shouldn't need to know) if it's talking directly to the server or through a Gateway/proxy (`H.../03_API_Gateway.md`)                   |
| **Code on Demand** *(optional)* | Server can occasionally send executable code (e.g., JavaScript) to the client — rarely used in practice, the only optional constraint                          |

## ⚙️ Resource Naming — the practical rules

```
✅ GOOD (nouns, plural, hierarchical):
   GET    /products              → list all products
   GET    /products/5            → get product #5
   GET    /products/5/reviews    → get reviews FOR product #5 (nested resource)
   POST   /products               → create a new product

❌ BAD (verbs, actions baked into the URL):
   GET    /getAllProducts
   GET    /fetchProductById/5
   POST   /createNewProduct
   POST   /products/5/deleteReview/9
```

| Rule                                                              | Example                                                                                           |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Use**nouns**, not verbs, in URLs                            | `/products`, not `/getProducts`                                                               |
| Use**plural** nouns for collections                         | `/products` (not `/product`)                                                                  |
| Nest resources to show relationships                              | `/products/5/reviews` (reviews belonging to product 5)                                          |
| Use query parameters for filtering/sorting, not new endpoints     | `/products?category=electronics&sort=price` — full depth in `04_Pagination_and_Filtering.md` |
| Don't put verbs in the URL — the HTTP method already IS the verb | `DELETE /products/5`, not `POST /products/5/delete`                                           |

## 🖼 Statelessness — why it matters for scaling (ties back to `C.Load_Balancing`)

```
STATEFUL (bad for REST/scaling):              STATELESS (RESTful):

Request 1: "login" → Server A remembers      Request 1: "login" → returns a token
           you're logged in (in its                      containing everything needed
           own memory/session)                            (e.g., JWT with user info)

Request 2: MUST go back to Server A          Request 2: includes the token → 
           (it's the only one that                       ANY server (A, B, or C) can
           remembers you)                                 handle it — no "sticky session"
                                                            requirement on the load balancer
```

This directly connects to `C.Load_Balancing` — a stateless API is what allows a load balancer to route each request to *any* server, without needing "sticky sessions" tied to one specific machine.

## 💻 Code examples

### Basic — a properly RESTful controller (resource-oriented, using your team's stack)

```csharp
[ApiController]
[Route("api/products")] // plural noun, resource-oriented
public class ProductsController : ControllerBase
{
    private readonly ProductService _service;

    [HttpGet]                          // GET /api/products
    public IActionResult GetAll() => Ok(_service.GetAllProducts());

    [HttpGet("{id}")]                  // GET /api/products/5
    public IActionResult GetById(int id)
    {
        var product = _service.GetProductById(id);
        return product == null ? NotFound() : Ok(product);
    }

    [HttpPost]                         // POST /api/products
    public IActionResult Create(ProductViewModel model)
    {
        var created = _service.CreateProduct(model); // Controller → BAL → DAL → Stored Procedure
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id}")]                  // PUT /api/products/5
    public IActionResult Update(int id, ProductViewModel model)
    {
        _service.UpdateProduct(id, model);
        return NoContent();
    }

    [HttpDelete("{id}")]               // DELETE /api/products/5
    public IActionResult Delete(int id)
    {
        _service.DeleteProduct(id);
        return NoContent();
    }
}
```

### Intermediate — nested resource (reviews belonging to a product)

```csharp
[ApiController]
[Route("api/products/{productId}/reviews")] // clearly shows the relationship in the URL
public class ProductReviewsController : ControllerBase
{
    private readonly ReviewService _service;

    [HttpGet]  // GET /api/products/5/reviews
    public IActionResult GetReviewsForProduct(int productId)
    {
        return Ok(_service.GetReviewsByProductId(productId));
    }
}
```

### Practical — a stateless, token-based request (no server-side session)

```csharp
// Every request carries everything the server needs — no session lookup required
[HttpGet("{id}")]
[Authorize] // reads claims directly from the incoming JWT token, not from server memory
public IActionResult GetById(int id)
{
    int currentUserId = int.Parse(User.FindFirst("userId")!.Value); // from the TOKEN, not a session
    return Ok(_service.GetProductById(id, currentUserId));
}
```

## ⚡ Performance considerations

- Statelessness enables trivial horizontal scaling — any server can handle any request, which is exactly what makes Load Balancing (`C.Load_Balancing`) simple and effective.
- Leaning on standard HTTP caching semantics (proper use of `GET` + cache headers) lets browsers, CDNs, and reverse proxies cache responses without any custom logic — free performance.
- Resource-oriented, predictable URLs make it much easier to apply consistent caching rules (e.g., "cache all `GET /products/*` responses for 5 minutes").

## 🚨 Common mistakes

- ❌ Using `GET` for operations that change data (e.g., `GET /products/5/delete`) — breaks caching assumptions and the fundamental REST contract that GET is safe/idempotent.
- ❌ Baking actions into the URL (`/createOrder`, `/getUserById`) instead of using the HTTP verb to express intent.
- ❌ Storing session state on the server between requests — breaks statelessness and makes horizontal scaling/load balancing much harder.
- ❌ Deeply nesting resources beyond 1-2 levels (`/products/5/reviews/9/comments/3/replies/1`) — becomes unwieldy; consider flattening with query parameters instead.

## 💡 Best practices

- ✅ Model URLs around resources (nouns), and let the HTTP method express the action.
- ✅ Keep the API genuinely stateless — put anything the server needs to know into the request itself (tokens, headers, parameters), not server-side session memory.
- ✅ Use proper HTTP status codes to communicate outcomes (full depth in `02_HTTP_Methods_and_Status_Codes.md`) rather than inventing custom success/failure fields in every response body.
- ✅ Keep resource nesting shallow — 1-2 levels max; use query parameters for anything more complex.

## 🎤 Interview Quick-Fire Q&A

| Question                                                       | Answer                                                                                                                        |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| What does REST stand for?                                      | REpresentational State Transfer                                                                                               |
| What is the "Uniform Interface" constraint?                    | A consistent way of interacting with all resources — a standard vocabulary of URLs (nouns) and HTTP verbs (actions)          |
| Why does statelessness matter for scalability?                 | Since no server remembers anything about a specific client, ANY server can handle ANY request — making load balancing simple |
| What's the correct RESTful way to represent "delete order#12"? | `DELETE /orders/12` — using the HTTP verb, not an action baked into the URL like `/deleteOrder`                          |
| Which REST constraint is optional?                             | Code on Demand (the server occasionally sending executable code to the client)                                                |

## 📝 30-second Revision Cheat Sheet

- REST = resource-oriented API design using nouns (URLs) + standard HTTP verbs (actions).
- Core constraints: Client-Server, Stateless, Cacheable, Uniform Interface, Layered System (+ optional Code on Demand).
- URLs: plural nouns, nested to show relationships, NO verbs in the path.
- Statelessness → any server can handle any request → simplifies load balancing.
- Use the HTTP verb to express intent — never bake actions into the URL.
