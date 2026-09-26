# 04_CQRS

> **CQRS (Command Query Responsibility Segregation)** = splitting an application's **write logic (Commands)** and **read logic (Queries)** into two separate models, instead of using one unified model for both.

> Builds on everything you already do: your team's stack technically already separates "read stored procedures" from "write stored procedures" — CQRS takes that separation further, potentially all the way to separate models, and even separate databases.

## 📌 What is it?

Traditionally (and in most CRUD apps, including a typical monolith), the **same model** is used for both reading and writing:

```
                    ┌───────────────┐
Read  ──────────►   │   Product     │   ◄────────── Write
                    │ (one shared    │
                    │  model for     │
                    │  both)         │
                    └───────────────┘
```

**CQRS** splits this into two distinct paths:

```
Command (Write) Path:                    Query (Read) Path:

Client ──► CommandHandler ──► Write Model  Client ──► QueryHandler ──► Read Model
              │                                          │
              ▼                                          ▼
         Write Database                             Read Database
         (optimized for                              (optimized for
          consistency,                                fast, flexible
          validation)                                 querying/display)
```

## 🤔 Why do we need it?

| Problem with one shared model                                                                  | How CQRS helps                                                                                                           |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Writes need strict validation/business rules; reads just need to display data fast             | Each side is optimized independently for its actual job                                                                  |
| A read-heavy dashboard query is awkward to express against a normalized write-optimized schema | The read model can be a denormalized, query-friendly shape (even a separate DB)                                          |
| Read and write load patterns are very different (e.g., 95% reads, 5% writes)                   | Read and write sides can be**scaled independently** (read side gets more replicas, matches `E.Database_Scaling`) |
| Complex reporting queries slow down the transactional write path                               | Reporting reads a separate, purpose-built read store — zero impact on writes                                            |

## 🌍 Real-world analogy

A **restaurant with a kitchen and a menu display board**. The kitchen (write/command side) handles the complex, careful process of actually cooking food — following recipes precisely, managing raw ingredients. The menu board (read/query side) is a simple, fast, purpose-built display of "what's available right now" — it doesn't need to know *how* the food is cooked, just what to show. They're updated at different times, for different purposes, by different concerns.

## ⚙️ Internal working — CQRS with your team's existing stack

```
COMMAND (Write) Path:
  Controller → CommandHandler (BAL) → Write DAL → sp_InsertProduct / sp_UpdateProduct
                                             │
                                             ▼
                                       Write Database
                                       (normalized, validated,
                                        source of truth)

                          (data syncs to the read side — see "Eventual Consistency" below)

QUERY (Read) Path:
  Controller → QueryHandler (BAL) → Read DAL → sp_GetProductListForDisplay
                                          │
                                          ▼
                                    Read Database / View
                                    (denormalized, fast,
                                     shaped exactly for the UI —
                                     e.g., a Kendo Grid view)
```

## 📊 Levels of CQRS — you don't have to go all the way to separate databases

| Level                                   | What's separated                                                                                                                          | Complexity                                    |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Level 1 — Separate methods**   | Just separate`GetX()` vs `SaveX()` methods/classes in your BAL                                                                        | Low — many teams already effectively do this |
| **Level 2 — Separate models**    | `ProductWriteModel` (strict, validated) vs `ProductReadDto` (denormalized, display-friendly) — same database                         | Moderate                                      |
| **Level 3 — Separate databases** | Write DB (normalized, transactional) and Read DB (denormalized, e.g., a reporting replica or a NoSQL read store), kept in sync via events | High — full CQRS                             |

> Most real-world systems stop at Level 1 or 2. Level 3 (with fully separate databases) is reserved for systems with genuinely very different, high-scale read and write patterns.

## 🖼 Eventual Consistency — the cost of full (Level 3) CQRS

```
1. Command: UpdateProductPrice(id=5, newPrice=49.99)
2. Write DB updated IMMEDIATELY → consistent, correct, source of truth

3. An event ("ProductPriceChanged") is published (ties into G.Communication's Pub/Sub)
4. Read DB is updated ASYNCHRONOUSLY, shortly after

   ┌─── brief window where Read DB still shows the OLD price ───┐
   │         (this is "eventual consistency")                    │
   └───────────────────────────────────────────────────────────┘
```

This is the exact same trade-off already covered in `01_CAP_Theorem.md` (favoring Availability over immediate Consistency) and `01_Replication.md` (replication lag) — CQRS's read/write split introduces the same kind of brief staleness window when fully separated.

## 💻 Code examples

### Basic — Level 1: separate Command and Query classes (same database)

```csharp
// COMMAND side — handles writes, enforces business rules
public class UpdateProductPriceCommand
{
    public int ProductId { get; set; }
    public decimal NewPrice { get; set; }
}

public class UpdateProductPriceCommandHandler
{
    private readonly ProductDAL _dal;

    public void Handle(UpdateProductPriceCommand command)
    {
        if (command.NewPrice <= 0)
            throw new ArgumentException("Price must be positive.");

        _dal.UpdatePrice(command.ProductId, command.NewPrice); // sp_UpdateProductPrice
    }
}

// QUERY side — handles reads, shaped exactly for display, no business rule concerns
public class GetProductListQueryHandler
{
    private readonly ProductDAL _dal;

    public List<ProductListItemDto> Handle()
    {
        return _dal.GetProductListForGrid(); // sp_GetProductListForDisplay — already shaped for Kendo Grid
    }
}
```

### Intermediate — Level 2: distinct Write Model vs Read DTO

```csharp
// WRITE model — strict, includes validation-relevant fields, matches normalized schema
public class ProductWriteModel
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Cost { get; set; }
    public decimal MarginPercent { get; set; }
}

// READ model — denormalized, pre-calculated, shaped exactly for the UI grid
public class ProductReadDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal DisplayPrice { get; set; }     // Cost + Margin, PRE-CALCULATED — no logic needed at display time
    public string CategoryName { get; set; } = ""; // denormalized — joined in advance, not at query time
}
```

### Practical — Controller routing to the correct side

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly UpdateProductPriceCommandHandler _commandHandler;
    private readonly GetProductListQueryHandler _queryHandler;

    [HttpPut("{id}/price")]
    public IActionResult UpdatePrice(int id, [FromBody] decimal newPrice)
    {
        _commandHandler.Handle(new UpdateProductPriceCommand { ProductId = id, NewPrice = newPrice });
        return NoContent(); // Commands typically don't return data
    }

    [HttpGet]
    public IActionResult GetList([DataSourceRequest] DataSourceRequest request)
    {
        var products = _queryHandler.Handle();
        return Ok(products.ToDataSourceResult(request)); // feeds directly into a Kendo Grid
    }
}
```

## ⚡ Performance considerations

- The read side can be **heavily optimized/denormalized** (pre-joined, pre-calculated) purely for display speed, without worrying about normalization rules that matter for writes.
- Read and write sides can be scaled completely independently — e.g., many Read Replicas (`E.Database_Scaling/04_Read_Replicas.md`) for the query side, while the write side stays on a single strongly-consistent primary.
- Full CQRS (Level 3) introduces real latency/staleness in the read side — only worth it when read and write scaling needs are genuinely very different.

## 🚨 Common mistakes

- ❌ Jumping straight to full Level 3 CQRS (separate databases) for a simple CRUD app — massive overkill; Level 1 or 2 is enough for most systems.
- ❌ Putting business validation logic in the Query side — Commands should own all business rules; Queries should just fetch and shape data.
- ❌ Not accounting for eventual consistency when using Level 3 — showing a user stale data right after they made a change, with no UX handling for that gap.

## 💡 Best practices

- ✅ Start with Level 1 (separate command/query methods) — it's nearly free and immediately clarifies intent in your codebase.
- ✅ Move to Level 2 (separate models) when your read-side display needs (grids, dashboards) start looking meaningfully different from your write-side validation needs.
- ✅ Only consider Level 3 (separate databases) when you have genuinely large-scale, divergent read/write patterns — and be prepared to handle eventual consistency in the UX.

## 🎤 Interview Quick-Fire Q&A

| Question                                             | Answer                                                                                                                        |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| What does CQRS stand for?                            | Command Query Responsibility Segregation                                                                                      |
| What's the core idea?                                | Separate the write path (Commands) from the read path (Queries), instead of using one shared model for both                   |
| Does CQRS require two separate databases?            | No — that's the most advanced form (Level 3); many systems benefit from just separate methods or models on the same database |
| What consistency trade-off does full CQRS introduce? | Eventual consistency — the read model may briefly lag behind the write model after an update                                 |
| When is CQRS overkill?                               | For simple CRUD applications without significantly different read/write scaling or modeling needs                             |

## 📝 30-second Revision Cheat Sheet

- CQRS = separate the write path (Commands) from the read path (Queries).
- Three levels: separate methods (cheap) → separate models (moderate) → separate databases (full CQRS, complex).
- Full CQRS introduces eventual consistency between write and read stores.
- Lets each side be optimized/scaled independently — reads can be denormalized and heavily replicated.
- Most systems only need Level 1 or 2 — reserve full CQRS for genuinely divergent read/write scaling needs.
