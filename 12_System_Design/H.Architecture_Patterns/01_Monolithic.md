# 01_Monolithic

> **Monolithic Architecture** = an entire application — UI, business logic, and data access — built and deployed as **one single unit**.

> New chapter: **H.Architecture_Patterns**. Everything so far (caching, scaling, communication) has been about techniques you can apply *within* an architecture. This chapter is about the **shape of the application itself** — starting with the simplest shape there is.

## 📌 What is it?

A monolith is exactly what you already know as **your team's current setup**: one ASP.NET Core MVC project containing Controllers, BAL, DAL, and everything else, deployed as a single unit, talking to one SQL Server database.

```
┌───────────────────────────────────────────┐
│           ONE Deployable Application        │
│                                               │
│  Controllers  →  BAL  →  DAL  →  Stored Procs │
│  (Kendo UI views also live here)             │
│                                               │
└───────────────────────────────────────────┘
                     │
                     ▼
              ┌─────────────┐
              │ SQL Server  │
              └─────────────┘
```

## 🤔 Why do we (still) need to understand it?

Monoliths get a bad reputation in modern System Design content, but they are **absolutely the right choice** for a huge number of real applications — including most internal business tools like the ones you build day-to-day.

| Monolith strength                      | Why it matters                                                               |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| Simple to develop                      | One codebase, one solution to open, one debugger session                     |
| Simple to deploy                       | One build, one deployment artifact/pipeline                                  |
| Simple to test                         | End-to-end testing doesn't require spinning up multiple services             |
| No network overhead between components | Controller → BAL → DAL are just in-process method calls, not network calls |
| Easier to reason about transactions    | One database, standard ACID transactions work naturally                      |

## 🌍 Real-world analogy

A **single, self-contained food truck**. One person (or small team) handles taking orders, cooking, and serving — all in one small space. It's fast to set up, easy for one team to run and understand fully, and works great at small-to-medium scale. It only becomes a problem when you need to serve an entire city at once from that one truck.

## ⚙️ Internal structure (your actual team stack, as a monolith)

```
                     HTTP Request
                          │
                          ▼
                 ┌─────────────────┐
                 │   Controller     │   (ASP.NET Core MVC)
                 └────────┬────────┘
                          ▼
                 ┌─────────────────┐
                 │       BAL        │   (Business logic / validation)
                 └────────┬────────┘
                          ▼
                 ┌─────────────────┐
                 │       DAL        │   (ADO.NET, SqlConnection/SqlCommand)
                 └────────┬────────┘
                          ▼
                 ┌─────────────────┐
                 │ Stored Procedure │
                 └────────┬────────┘
                          ▼
                 ┌─────────────────┐
                 │   SQL Server     │
                 └─────────────────┘

ALL of this lives in ONE process, ONE deployment, ONE codebase.
```

## 📊 Monolith vs Microservices (full comparison lands in `02_Microservices.md`) — quick preview

| Aspect         | Monolith                                                                   | Microservices                                                         |
| -------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Deployment     | One unit, deployed together                                                | Many independent services, deployed separately                        |
| Scaling        | Scale the whole app together                                               | Scale only the specific service that needs it                         |
| Team structure | Works well for one team / small teams                                      | Suited to many independent teams owning separate services             |
| Complexity     | Low operational complexity                                                 | High operational complexity (networking, service discovery, etc.)     |
| Best for       | Small-to-medium apps, startups, internal tools, most projects starting out | Very large-scale systems with independent, separately-scaling domains |

## 🖼 Where a monolith starts to strain (not "fails" — strains)

```
As the app grows:

  - Codebase gets large → longer build times, harder onboarding
  - One team's change risks breaking an unrelated feature (shared deployment)
  - Scaling: if only the "reporting" feature is under heavy load,
    you still have to scale the ENTIRE app (can't scale just that piece)
  - A bug in one module can, in the worst case, crash the entire process
```

This is where teams *sometimes* start looking at breaking a monolith apart — but that decision has real costs too (see Best Practices below).

## 💻 Code examples

### Basic — a typical monolith's internal layering (this is exactly your team's stack)

```csharp
// Controller — part of the same deployable unit as everything below it
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductBAL _bal;

    public ProductsController(ProductBAL bal) => _bal = bal;

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _bal.GetProductById(id); // in-process call, no network hop
        return product == null ? NotFound() : Ok(product);
    }
}

// BAL — same process, same deployment
public class ProductBAL
{
    private readonly ProductDAL _dal;
    public ProductBAL(ProductDAL dal) => _dal = dal;

    public Product GetProductById(int id)
    {
        if (id <= 0) throw new ArgumentException("Invalid product id");
        return _dal.GetProductById(id); // still just an in-process method call
    }
}

// DAL — same process, same deployment, talks to the ONE shared database
public class ProductDAL
{
    private readonly string _connString;
    public ProductDAL(IConfiguration config) => _connString = config.GetConnectionString("Default")!;

    public Product GetProductById(int id)
    {
        using var conn = new SqlConnection(_connString);
        using var cmd = new SqlCommand("sp_GetProductById", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@Id", id);

        conn.Open();
        using var reader = cmd.ExecuteReader();
        return reader.Read() ? MapToProduct(reader) : null;
    }
}
```

> Notice: **Controller → BAL → DAL is a single call stack in one process.** No HTTP call, no serialization, no network latency between them — this is the biggest structural advantage a monolith has over microservices, where that same call would cross a network boundary.

## ⚡ Performance considerations

- In-process calls (Controller → BAL → DAL) are **orders of magnitude faster** than the equivalent network calls between microservices.
- The whole app scales as one unit — horizontal scaling means running more copies of the *entire* application (more IIS/Kestrel instances behind a load balancer), even if only one small part of it is under load.
- Vertical scaling (`B.Scalability/02_Vertical_Scaling.md`) is often the simplest first lever for a monolith before considering a full horizontal or microservices approach.

## 🚨 Common mistakes

- ❌ Assuming a monolith is inherently "bad" or "outdated" — most successful products start (and often stay) as monoliths; many large, successful companies still run monoliths for core parts of their systems.
- ❌ Jumping to microservices prematurely "because that's what big tech does," without the actual scaling/team-size problems microservices are meant to solve.
- ❌ Letting a monolith's internal layers become tangled (BAL calling DAL calling BAL, Controllers doing DB access directly) — even in a monolith, clean layering (Controller → BAL → DAL, as your team already does) keeps it maintainable.

## 💡 Best practices

- ✅ Start with a monolith unless you have a clear, specific reason not to (a small team, an unclear domain, or a system that genuinely doesn't need independent scaling yet) — this is now widely accepted, mainstream advice, not a compromise.
- ✅ Keep clean internal layering (Controller → BAL → DAL) even within a monolith — it keeps the codebase maintainable and makes a *future* split into services far less painful, if it's ever actually needed.
- ✅ Use vertical scaling and the techniques from earlier chapters (Caching, Replication, Read Replicas) to get significant scale out of a monolith before considering the added complexity of microservices.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                   | Answer                                                                                                                                |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| What is a monolithic architecture?                                         | An application where UI, business logic, and data access are built and deployed together as one single unit                           |
| What's the biggest performance advantage of a monolith over microservices? | Internal calls (Controller → BAL → DAL) are in-process method calls, not network calls — no network latency/serialization overhead |
| What's a real limitation of monoliths at scale?                            | You can't scale one specific feature independently — scaling means running more copies of the entire application                     |
| Is starting with a monolith considered bad practice today?                 | No — it's widely recommended as the sensible default for most new applications, especially with a small team                         |
| What keeps a monolith maintainable as it grows?                            | Clean internal layering (e.g., Controller → BAL → DAL) — separating concerns even without separate deployments                     |

## 📝 30-second Revision Cheat Sheet

- Monolith = UI + business logic + data access, all built and deployed as ONE unit.
- Advantages: simple to build/deploy/test, no network overhead between internal layers, easy transactions.
- Limitation: scales as a whole — can't scale just one busy feature independently.
- Still the sensible DEFAULT starting point for most applications, including your team's stack.
- Keep clean layering (Controller → BAL → DAL) — it's what makes a monolith maintainable long-term.
