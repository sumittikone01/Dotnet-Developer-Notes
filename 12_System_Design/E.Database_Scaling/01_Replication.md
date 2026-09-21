# 01_Replication

> **Replication** = keeping multiple copies of the same database on different servers, kept in sync, so reads (and sometimes writes) can be spread across more than one machine.

> New chapter: **E.Database_Scaling**. Where `D.Caching` reduced load on the DB by avoiding hits altogether, Replication tackles the problem from the other side — scaling the DB itself.

## 📌 What is it?

Instead of one SQL Server instance handling every query, you run **multiple copies** of the database (replicas). Data written to the **primary** (a.k.a. master) is automatically copied to one or more **replicas** (a.k.a. secondaries/slaves).

## 🤔 Why do we need it?

| Problem with a single DB server                       | How replication helps                           |
| ----------------------------------------------------- | ----------------------------------------------- |
| One server can only handle so many concurrent reads   | Spread reads across multiple replicas           |
| Primary server crashes → total outage                | A replica can be promoted to primary (failover) |
| Read-heavy reporting queries slow down normal traffic | Route heavy reports to a dedicated replica      |
| Users far from the primary DB get high latency        | Place replicas geographically closer to users   |

## 🌍 Real-world analogy

A **newspaper printing press with regional branches**. The main press (primary) writes the master copy. Regional branches (replicas) get identical copies printed and distributed locally — readers in each region get their paper faster, without everyone lining up at the single main press.

## ⚙️ Internal working

```
                 WRITE
Client ────────────────────► Primary DB
                                 │
                    (replication stream / log shipping)
                                 │
                 ┌───────────────┼───────────────┐
                 ▼               ▼               ▼
            Replica 1       Replica 2       Replica 3
                 ▲               ▲               ▲
                 │               │               │
Client ──────────┴───────────────┴───────────────┘
                       READS
```

- **Writes** → always go to the Primary.
- **Reads** → can be served by any Replica (or the Primary too).
- Replicas apply the same changes the Primary made, usually via a **transaction log** being shipped and replayed.

## 📊 Replication Types

| Type                       | How it works                                                                            | Trade-off                                                     |
| -------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Synchronous**      | Primary waits for replica(s) to confirm the write before acknowledging it to the client | Strong consistency, but higher write latency                  |
| **Asynchronous**     | Primary acknowledges the write immediately; replicas catch up afterward                 | Fast writes, but replicas can briefly lag ("replication lag") |
| **Semi-synchronous** | Primary waits for at least one replica to confirm, others catch up later                | A middle ground                                               |

## 📊 Replication Topologies

| Topology                                   | Description                                            | Use case                                                          |
| ------------------------------------------ | ------------------------------------------------------ | ----------------------------------------------------------------- |
| **Primary–Replica (Master–Slave)** | One primary handles writes; replicas handle reads only | Most common — read scaling                                       |
| **Multi-Primary (Master–Master)**   | Multiple nodes can accept writes, sync with each other | Multi-region write availability — more complex conflict handling |

## 🖼 Replication lag — the catch

```
Primary:  Product #5 price updated to $50   [t = 0ms]

Replica:  still shows Product #5 price = $40    [t = 0ms to ~50ms]
                                                  ↑
                                     "replication lag" window —
                                     replica hasn't caught up yet

Replica:  finally shows $50   [t = ~50ms+]
```

If a user updates data and immediately reads it back from a **replica**, they might briefly see the **old** value — this is called a **read-after-write consistency** problem, and it's the most common gotcha with asynchronous replication.

## 💻 Code examples

### Basic — conceptual: routing reads vs writes to different connection strings

```csharp
public class ProductDAL
{
    private readonly string _primaryConnString;   // for writes
    private readonly string _replicaConnString;   // for reads

    public Product GetProductById(int id)
    {
        // READ — use replica connection
        using var conn = new SqlConnection(_replicaConnString);
        using var cmd = new SqlCommand("sp_GetProductById", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@Id", id);

        conn.Open();
        using var reader = cmd.ExecuteReader();
        // ... map reader to Product ...
        return MapToProduct(reader);
    }

    public void UpdateProduct(Product product)
    {
        // WRITE — must use primary connection
        using var conn = new SqlConnection(_primaryConnString);
        using var cmd = new SqlCommand("sp_UpdateProduct", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@Id", product.Id);
        cmd.Parameters.AddWithValue("@Name", product.Name);

        conn.Open();
        cmd.ExecuteNonQuery();
    }
}
```

### Intermediate — handling read-after-write consistency

```csharp
public Product UpdateAndReturnProduct(Product product)
{
    UpdateProduct(product); // writes to PRIMARY

    // DON'T immediately read this same product from a replica — it might be stale (lag).
    // Instead, return the object you already have in memory, or explicitly read from the PRIMARY:
    return GetProductByIdFromPrimary(product.Id);
}
```

> This is a very real, very common bug: "I just updated the record but the page still shows the old value" — almost always a replication-lag / read-from-replica-too-soon issue.

## ⚡ Performance considerations

- Replication scales **reads** very well — add more replicas as read traffic grows.
- It does **not** scale writes — all writes still funnel through the primary. (Sharding, `02_Sharding.md`, addresses write scaling.)
- Asynchronous replication is the norm in most real-world setups because synchronous replication's write-latency cost is usually not worth the extra consistency guarantee.

## 🚨 Common mistakes

- ❌ Reading immediately after a write from a replica and expecting fresh data (classic replication-lag bug).
- ❌ Assuming replication solves write scalability — it only helps with reads.
- ❌ Not monitoring replication lag — if a replica falls far behind, it silently serves increasingly stale data.

## 💡 Best practices

- ✅ Route write-then-immediate-read flows back to the **primary**, not a replica.
- ✅ Monitor replication lag as a first-class metric (ties into `Logging_and_Monitoring` chapter later).
- ✅ Use asynchronous replication for most systems; reserve synchronous for cases where strong consistency truly matters (e.g., financial ledgers).

## 🎤 Interview Quick-Fire Q&A

| Question                                                                  | Answer                                                                                                              |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| What does replication scale — reads, writes, or both?                    | Reads only; writes still go through the primary                                                                     |
| What is "replication lag"?                                                | The delay between a write on the primary and that change appearing on a replica                                     |
| Sync vs async replication — what's the trade-off?                        | Sync = stronger consistency, slower writes; Async = faster writes, possible temporary staleness on replicas         |
| What's the classic replication bug?                                       | Reading a just-written value from a replica before it has caught up — seeing stale data                            |
| What's the difference between Primary-Replica and Multi-Primary topology? | Primary-Replica: only one node accepts writes; Multi-Primary: multiple nodes accept writes and sync with each other |

## 📝 30-second Revision Cheat Sheet

- Replication = copies of the DB (replicas) kept in sync with a primary.
- Writes → primary only; Reads → can be spread across replicas.
- Sync replication = consistent but slower writes; Async = faster but replicas can lag.
- Watch out for **read-after-write** staleness when reading from a replica too soon.
- Solves read scaling and failover — not write scaling (that's Sharding, next chapter).
