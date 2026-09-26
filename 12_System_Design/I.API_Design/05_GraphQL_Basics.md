# 05_GraphQL_Basics

> **GraphQL** = a query language for APIs where the **client specifies exactly which fields it needs**, in a single request — instead of the server dictating a fixed response shape per endpoint, as REST does.

> Closes the loop on `01_REST_API_Principles.md` and `04_Pagination_and_Filtering.md` — those chapters were entirely about REST's model (fixed endpoints, server-defined response shapes). GraphQL is an alternative approach that solves REST's two most common pain points: **over-fetching** and **under-fetching**.

## 📌 What is it?

In REST, `GET /products/5` always returns the same fixed shape — every field, whether the client needs it or not. In GraphQL, there's **one endpoint** (`/graphql`), and the client sends a **query** describing exactly which fields it wants back.

```
REST:                                    GraphQL:

GET /products/5                          POST /graphql
→ returns EVERYTHING:                    { product(id: 5) { name price } }
  { id, name, price, description,        → returns ONLY what was asked for:
    stock, category, supplier, ... }        { "name": "Widget", "price": 49.99 }
```

## 🤔 Why do we need it? — The two problems it solves

### Problem 1: Over-fetching

```
REST: A mobile app's product list screen only needs { name, price }
      but GET /products returns the FULL object for every product:
      { id, name, price, description, stock, category, supplier, reviews, ... }

      → Wasted bandwidth sending fields the screen never displays
        (especially painful on mobile networks)
```

### Problem 2: Under-fetching

```
REST: A "Product Detail" screen needs product info + reviews + related products
      → requires THREE separate REST calls (unless you build a custom aggregating
        endpoint, exactly like G.../03_API_Gateway.md's "Request Aggregation")

      GET /products/5
      GET /products/5/reviews
      GET /products/5/related

GraphQL: ONE query, ONE request, gets exactly this shape:

      {
        product(id: 5) {
          name
          price
          reviews { rating comment }
          relatedProducts { name price }
        }
      }
```

| Problem                                           | REST's typical fix                                                                      | GraphQL's built-in fix                                      |
| ------------------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Over-fetching (too much data)                     | Create narrower, purpose-built endpoints (`/products/summary`)                        | Client just asks for fewer fields — no new endpoint needed |
| Under-fetching (too little, needs multiple calls) | Build a custom aggregating endpoint, or use an API Gateway (`H.../03_API_Gateway.md`) | Client asks for nested/related data in the SAME query       |

## 🌍 Real-world analogy

Ordering at a **build-your-own sandwich counter** vs a **fixed combo meal menu**. REST is the combo meal — you get exactly what's on the menu, no substitutions (over-fetching: you didn't want the pickles; under-fetching: you also wanted a drink, that's a separate order). GraphQL is build-your-own — you specify precisely which ingredients (fields) you want, all in one order.

## ⚙️ Internal working

```
1. Client sends a QUERY (not a URL path) describing exactly what it wants:

   query {
     product(id: 5) {
       name
       price
       reviews { rating }
     }
   }

2. A SINGLE GraphQL endpoint (typically POST /graphql) receives this

3. A "Resolver" function runs for EACH requested field:

   Resolver for "product"  → calls ProductService.GetById(5)   (your existing BAL/DAL!)
   Resolver for "reviews"  → calls ReviewService.GetByProductId(5)

4. Results are assembled into exactly the shape the client asked for:

   {
     "data": {
       "product": {
         "name": "Widget",
         "price": 49.99,
         "reviews": [ { "rating": 5 }, { "rating": 4 } ]
       }
     }
   }
```

> Key insight: **Resolvers are just thin wrappers around your existing BAL/DAL logic.** GraphQL doesn't replace your Controller → BAL → DAL architecture — it replaces the *outer HTTP/routing layer* that decides what data gets returned and in what shape.

## 📊 REST vs GraphQL — Full Comparison

| Aspect                   | REST                                                 | GraphQL                                                                                            |
| ------------------------ | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Endpoints                | Many (one per resource/action)                       | One (typically`/graphql`)                                                                        |
| Response shape           | Fixed per endpoint, server-decided                   | Client-specified per request                                                                       |
| Over-fetching            | Common problem                                       | Solved — client asks for only needed fields                                                       |
| Under-fetching           | Common problem (multiple round trips)                | Solved — nested queries in one request                                                            |
| Caching                  | Easy — standard HTTP caching on GET works naturally | Harder — typically POST-based, needs custom caching strategy                                      |
| Learning curve / tooling | Simple, universal, huge existing tooling             | Steeper — needs a schema, resolvers, a GraphQL client library                                     |
| Versioning               | Explicit versions needed (`03_API_Versioning.md`)  | Often avoided — add new fields/types without versioning, since clients only request what they use |
| Best for                 | Simple CRUD, public APIs, cache-friendly needs       | Complex, nested data needs, multiple client types (mobile/web) with very different data needs      |

## 💻 Code examples

### Basic — defining a GraphQL schema (the contract, similar in spirit to a `.proto` file from gRPC)

```graphql
# schema.graphql
type Product {
  id: ID!
  name: String!
  price: Float!
  reviews: [Review!]!
}

type Review {
  rating: Int!
  comment: String
}

type Query {
  product(id: ID!): Product
}
```

### Intermediate — implementing resolvers in ASP.NET Core (using HotChocolate, the most common .NET GraphQL library)

```csharp
// Program.cs
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();

// Query.cs — resolvers wrap your EXISTING BAL, no rewrite needed
public class Query
{
    public Product? GetProduct(int id, [Service] ProductService productService)
    {
        return productService.GetProductById(id); // same BAL → DAL → Stored Procedure call as REST would use
    }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }

    // A "field resolver" — only runs if the client actually requested "reviews"
    public List<Review> GetReviews([Service] ReviewService reviewService) =>
        reviewService.GetReviewsByProductId(Id);
}
```

### Practical — the client-side query (this is what replaces multiple REST calls)

```graphql
query GetProductDetail {
  product(id: 5) {
    name
    price
    reviews {
      rating
      comment
    }
  }
}
```

```
POST /graphql
Content-Type: application/json

{ "query": "query GetProductDetail { product(id: 5) { name price reviews { rating comment } } }" }

Response — EXACTLY the requested shape, nothing more:
{
  "data": {
    "product": {
      "name": "Widget",
      "price": 49.99,
      "reviews": [ { "rating": 5, "comment": "Great!" } ]
    }
  }
}
```

## ⚡ Performance considerations

- GraphQL reduces network round trips (great for mobile) and payload size (no unused fields) — a genuine win for bandwidth-constrained clients.
- The **"N+1 query problem"** is a classic GraphQL pitfall: naively resolving `reviews` for each of 20 products in a list can trigger 20 separate DB calls instead of 1 batched call — solved with a technique called "DataLoader" batching (worth knowing exists, even if the fix itself is a deeper topic).
- Because most GraphQL APIs use `POST` for everything, standard HTTP `GET` caching (CDNs, browser cache) doesn't apply automatically — caching needs a deliberate, custom strategy.

## 🚨 Common mistakes

- ❌ Adopting GraphQL for a simple CRUD app with no real over/under-fetching problem — adds real complexity (schema design, resolvers, client tooling) without a matching benefit.
- ❌ Hitting the N+1 query problem by writing naive resolvers that each independently query the database per item in a list.
- ❌ Assuming GraphQL automatically gets HTTP caching benefits like REST's `GET` does — it typically needs its own caching approach.
- ❌ Exposing your entire database schema 1:1 as the GraphQL schema — leaks internal structure and can allow overly expensive queries from clients.

## 💡 Best practices

- ✅ Reach for GraphQL when you have genuinely nested/complex data needs and/or multiple client types (web, mobile, third-party) with very different data requirements per screen.
- ✅ Keep resolvers thin — delegate to your existing BAL/DAL, don't duplicate business logic inside resolver functions.
- ✅ Watch for and address the N+1 problem early (batching/DataLoader patterns) — it's the most common GraphQL performance issue in practice.
- ✅ Consider limiting query depth/complexity server-side, so a client can't request an arbitrarily expensive, deeply nested query that overloads the server.

## 🎤 Interview Quick-Fire Q&A

| Question                                                           | Answer                                                                                                                                                                             |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What two core problems does GraphQL solve that are common in REST? | Over-fetching (too much unused data returned) and under-fetching (needing multiple round trips for related data)                                                                   |
| How many endpoints does a typical GraphQL API expose?              | Usually just one (e.g.,`POST /graphql`) — the client's query determines what data comes back                                                                                    |
| What is a "resolver" in GraphQL?                                   | A function that fetches the data for a specific field in the schema — typically a thin wrapper around existing business logic (BAL/DAL)                                           |
| What is the "N+1 query problem"?                                   | A naive resolver pattern that triggers one database query per item in a list (N) plus the initial query (1), instead of batching them                                              |
| Why is REST generally easier to cache than GraphQL?                | REST commonly uses`GET` requests, which standard HTTP caching (browsers, CDNs) understands natively; GraphQL typically uses `POST`, which isn't cached the same way by default |

## 📝 30-second Revision Cheat Sheet

- GraphQL = client specifies exactly which fields it wants, in one request, to a single endpoint.
- Solves REST's over-fetching (too much data) and under-fetching (multiple round trips) problems.
- Resolvers are thin wrappers around your existing BAL/DAL — architecture underneath doesn't change.
- Watch for the N+1 query problem — batch/DataLoader patterns fix it.
- Best for complex, nested data needs and multiple client types; overkill for simple CRUD.
- Harder to cache than REST (mostly POST-based) — needs a deliberate caching strategy.

---

✅ **I.API_Design chapter complete** (01–05). Next up: **J.Reliability_Patterns**.
