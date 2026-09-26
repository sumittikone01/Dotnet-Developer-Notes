# 04_Read_Replicas

> **Read Replica** = a replica (from `01_Replication.md`) that is specifically used to **offload read traffic** away from the primary, so the primary is left free to handle writes efficiently.

> This isn't a new mechanism — it's the most common **application pattern built on top of Replication**. If `01_Replication.md` was "how copies get made," this chapter is "how you actually use those copies in your app's read/write routing."

## 📌 What is it?

You already know replicas exist (`01_Replication.md`). A **Read Replica** setup means your application is explicitly designed to:

- Send **all writes** → Primary
- Send **all reads** → one or more Read Replicas

This is a deliberate **routing strategy** in your BAL/DAL layer, not just a database feature sitting unused.

```
              WRITES
Application ───────────► Primary DB
     │
     │ READS
     └───────────► Read Replica 1
     └───────────► Read Replica 2
     └───────────► Read Replica 3
```

## 🤔 Why do we need it?

Most real-world applications are **read-heavy** — a typical ratio is 80-90% reads vs 10-20% writes (product catalogs, dashboards, user profiles are read far more often than they're updated). If every read *and* write hits the same single server, that server becomes the bottleneck long before it needs to be.

| Without read/write splitting                        | With Read Replicas                                                      |
| --------------------------------------------------- | ----------------------------------------------------------------------- |
| Primary handles 100% of reads + 100% of writes      | Primary handles ~100% of writes only                                    |
| One server = one throughput ceiling                 | Read throughput scales by adding more replicas                          |
| Heavy reporting query can slow down checkout writes | Route reporting queries to a dedicated replica, isolate from write path |

## 🌍 Real-world analogy

A **restaurant with one chef (writes) and several waiters (reads)**. Customers asking "what's in this dish?" (reads) go to any waiter — you don't need the chef to answer that. Only actual food preparation (writes) requires the chef. Add more waiters (replicas) as the dining room gets busier; you don't need a second kitchen for that.

## ⚙️ Internal working — read/write splitting in the BAL/DAL

```
Controller
    │
    ▼
BAL (Business Logic Layer)
    │
    ▼
DAL ──── decides based on operation type ────┐
    │                                         │
    ▼ (SELECT)                                ▼ (INSERT/UPDATE/DELETE)
Read Replica connection                  Primary connection
```

This decision is usually made **per stored-procedure call** — your DAL methods already know whether they're reading or writing, so routing is just "pick the right connection string."

## 📊 Where to route each kind of query

| Operation                                                          | Goes to                          | Why                                                              |
| ------------------------------------------------------------------ | -------------------------------- | ---------------------------------------------------------------- |
| `SELECT` (product list, profile view, search)                    | Read Replica                     | Doesn't need the freshest possible data (usually)                |
| `INSERT` / `UPDATE` / `DELETE`                               | Primary                          | Must be durable, consistent, and immediately correct             |
| `SELECT` right after a write, same request (read-your-own-write) | **Primary** (not replica!) | Avoids the replication-lag bug from`01_Replication.md`         |
| Heavy analytics/reporting queries                                  | Dedicated Read Replica           | Isolates expensive long-running queries from user-facing traffic |

## 💻 Code examples

### Basic — a DAL that explicitly splits read/write connections

```csharp
public class ProductDAL
{
    private readonly string _writeConnString; // → Primary
    private readonly string _readConnString;  // → Read Replica

    public ProductDAL(IConfiguration config)
    {
        _writeConnString = config.GetConnectionString("PrimaryDb")!;
        _readConnString  = config.GetConnectionString("ReadReplicaDb")!;
    }

    // READ — goes to the replica
    public List<Product> GetAllProducts()
    {
        var products = new List<Product>();

        using var conn = new SqlConnection(_readConnString);
        using var cmd = new SqlCommand("sp_GetAllProducts", conn) { CommandType = CommandType.StoredProcedure };

        conn.Open();
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            products.Add(MapToProduct(reader));
        }
        return products;
    }

    // WRITE — goes to the primary
    public void InsertProduct(Product product)
    {
        using var conn = new SqlConnection(_writeConnString);
        using var cmd = new SqlCommand("sp_InsertProduct", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@Name", product.Name);
        cmd.Parameters.AddWithValue("@Price", product.Price);

        conn.Open();
        cmd.ExecuteNonQuery();
    }
}
```

### Intermediate — a small helper to centralize the routing decision

```csharp
public class ConnectionRouter
{
    private readonly string _primary;
    private readonly List<string> _replicas;
    private int _roundRobinIndex = 0;

    public ConnectionRouter(string primary, List<string> replicas)
    {
        _primary = primary;
        _replicas = replicas;
    }

    public string GetWriteConnection() => _primary;

    // Simple round-robin across replicas to spread load evenly
    public string GetReadConnection()
    {
        var connStr = _replicas[_roundRobinIndex % _replicas.Count];
        _roundRobinIndex++;
        return connStr;
    }

    // For "read your own write" scenarios in the same request
    public string GetConsistentReadConnection() => _primary;
}
```

### Practical — handling the read-after-write case correctly

```csharp
public Product CreateProduct(ProductViewModel model)
{
    var product = model.ToEntity();
    _dal.InsertProduct(product); // → Primary

    // WRONG: immediately calling GetProductById(product.Id) here would hit a REPLICA
    // and might not see the row yet (replication lag) → looks like the insert "failed"

    // RIGHT: either return the object we already built in memory...
    return product;

    // ...or, if you must re-fetch, explicitly read from the PRIMARY for this one call
}
```

## ⚡ Performance considerations

- Read replicas scale **horizontally** — need more read capacity? Add another replica. Very cheap way to grow read throughput.
- **Load-balance across replicas** (round-robin, least-connections) so no single replica becomes a new bottleneck.
- Doesn't help write throughput at all — for that, see `02_Sharding.md`.
- Dedicate one replica specifically to **heavy analytical queries** so they never compete with regular user traffic, even on the read side.

## 🚨 Common mistakes

- ❌ Reading immediately after writing from a replica instead of the primary (the #1 read-replica bug — inherited directly from `01_Replication.md`'s lag problem).
- ❌ Sending ALL traffic (including writes) to replicas by mistake due to a misconfigured connection string — replicas are often read-only at the DB level and will throw errors, but a subtle bug can still cause confusing failures.
- ❌ Not monitoring individual replica health/lag — a lagging replica can silently serve significantly stale data while looking "up."

## 💡 Best practices

- ✅ Explicitly split read and write connection strings in configuration (`PrimaryDb`, `ReadReplicaDb`) — never rely on the "same connection happens to work for both."
- ✅ For any read that must reflect a write from the *same* request/transaction, explicitly route to the primary.
- ✅ Load-balance reads across multiple replicas rather than pinning all reads to just one.
- ✅ Consider a dedicated replica purely for reporting/analytics so heavy queries never impact regular user-facing reads.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                     | Answer                                                                                                                                                            |
| ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between "Replication" and "Read Replicas" as concepts? | Replication is the underlying copy mechanism; Read Replicas is the application-level pattern of deliberately routing reads to those copies to offload the primary |
| Why are most systems read-heavy, and why does that matter here?              | Typical apps see far more reads than writes (e.g. 80/20) — so offloading just the reads to replicas relieves the majority of load from the primary               |
| When should a read go to the Primary instead of a Replica?                   | Immediately after a write in the same request/flow ("read-your-own-write"), to avoid replication-lag staleness                                                    |
| How do you scale read capacity further with this pattern?                    | Add more read replicas and load-balance reads across them                                                                                                         |
| What kind of query is a great candidate for a dedicated replica?             | Long-running analytics/reporting queries — isolates them from user-facing traffic                                                                                |

## 📝 30-second Revision Cheat Sheet

- Read Replicas = the deliberate pattern of sending writes → Primary, reads → Replicas.
- Built directly on top of Replication (`01_Replication.md`) — not a separate mechanism.
- Great fit for read-heavy apps (most apps are read-heavy).
- Golden rule: read-after-write in the same flow → go to Primary, not a Replica.
- Scales read throughput horizontally by adding more replicas; does NOT scale writes (that needs Sharding).

---

✅ **E.Database_Scaling chapter complete** (01–04). Next up: **F.Distributed_Systems**.
