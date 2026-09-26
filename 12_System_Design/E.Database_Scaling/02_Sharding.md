# 02_Sharding

> **Sharding** = splitting a single large database **horizontally** into multiple independent databases (shards), where each shard holds a **different subset of rows**, and — critically — each shard can live on its **own server**.

> Where `01_Replication.md` scaled **reads** by copying the *entire* dataset onto multiple servers, Sharding scales **writes** (and storage) by splitting the dataset itself across multiple servers. Different axis, different problem.

## 📌 What is it?

Instead of one giant `Products` table on one server holding 500 million rows, you split it into, say, 4 **shards** — each a full, independent database with its own schema, each holding ~125 million rows, each potentially on its own physical server.

```
Single DB (before sharding):
┌─────────────────────────────┐
│   Products (500M rows)      │   ← one server, one disk, one bottleneck
└─────────────────────────────┘

Sharded (after):
┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐
│ Shard 1   │ │ Shard 2   │ │ Shard 3   │ │ Shard 4   │
│ 125M rows │ │ 125M rows │ │ 125M rows │ │ 125M rows │
│ Server A  │ │ Server B  │ │ Server C  │ │ Server D  │
└───────────┘ └───────────┘ └───────────┘ └───────────┘
```

## 🤔 Why do we need it?

Replication (previous chapter) copies the **whole** dataset onto every replica — that helps reads, but every replica still has to store 100% of the data, and every **write** still funnels through one primary. Once a dataset (or its write volume) outgrows what a single machine can hold or handle, you need to **divide the data itself**. That's Sharding.

| Limit hit                                 | How Sharding helps                                                      |
| ----------------------------------------- | ----------------------------------------------------------------------- |
| Dataset too big for one server's disk     | Each shard stores only a fraction of the data                           |
| Too many writes for one primary to handle | Writes are spread across multiple independent primaries (one per shard) |
| Single point of write bottleneck          | No single server owns 100% of the write load                            |

## 🌍 Real-world analogy

A **postal system split by ZIP code**. Instead of one giant national sorting facility (one DB) handling every letter in the country, mail is routed to regional sorting centers based on ZIP code (shard key). Each regional center only ever deals with its own slice of the country's mail — faster, and no single center is a bottleneck.

## ⚙️ Internal working — the Shard Key

The **shard key** (a.k.a. partition key) is the column used to decide **which shard a row belongs to**. Choosing it well is the single most important decision in sharding.

```
Row: { UserId: 4521, Name: "Sumit" }

Shard Key = UserId
Shard = hash(UserId) % NumberOfShards

hash(4521) % 4 = 1  →  goes to Shard 1
```

```
                    Application
                        │
                 (computes shard from key)
                        │
        ┌───────────────┼───────────────┬───────────────┐
        ▼               ▼               ▼               ▼
    Shard 0         Shard 1         Shard 2         Shard 3
  (UserIds          (UserIds        (UserIds        (UserIds
   ending in         ending in       ending in       ending in
   hash%4==0)        hash%4==1)      hash%4==2)      hash%4==3)
```

## 📊 Sharding Strategies

| Strategy                  | How it decides the shard                                                      | Pros                             | Cons                                                                           |
| ------------------------- | ----------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------ |
| **Range-based**     | Rows split by value ranges (e.g., UserId 1–1M → Shard 1, 1M–2M → Shard 2) | Simple, easy range queries       | Uneven load if data isn't evenly distributed ("hotspots")                      |
| **Hash-based**      | `hash(key) % N` decides the shard                                           | Even distribution of load        | Range queries across shards become hard; resharding is painful                 |
| **Directory-based** | A lookup table maps each key to its shard explicitly                          | Very flexible, easy to rebalance | The lookup table itself becomes a critical, potentially bottlenecked component |
| **Geo-based**       | Shard by region/location (e.g., India users → Shard-IN)                      | Low latency for regional users   | Uneven shard sizes if usage isn't evenly spread geographically                 |

## 🚨 The hardest part: Cross-shard operations

```
Query: "Get total order count for all users"

Single DB:  SELECT COUNT(*) FROM Orders;              → 1 query

Sharded:    SELECT COUNT(*) FROM Orders; (on Shard 0)
            SELECT COUNT(*) FROM Orders; (on Shard 1)  → N queries,
            SELECT COUNT(*) FROM Orders; (on Shard 2)     then SUM them
            SELECT COUNT(*) FROM Orders; (on Shard 3)     in application code
```

- **JOINs across shards** are extremely painful — usually avoided by design (denormalize, or ensure related data lives on the same shard).
- **Cross-shard transactions** (updating rows in Shard 1 and Shard 2 atomically) require distributed transaction protocols (e.g., two-phase commit) — complex and slow. Most systems avoid this by design (`04_Read_Replicas.md` and later `Distributed_Systems` chapters go deeper here).

## 💻 Code examples

### Basic — computing the shard for a given key

```csharp
public class ShardRouter
{
    private readonly string[] _shardConnectionStrings;

    public ShardRouter(string[] shardConnectionStrings)
    {
        _shardConnectionStrings = shardConnectionStrings;
    }

    public string GetConnectionStringForUser(int userId)
    {
        int shardIndex = Math.Abs(userId.GetHashCode()) % _shardConnectionStrings.Length;
        return _shardConnectionStrings[shardIndex];
    }
}
```

### Intermediate — routing a DAL call to the correct shard

```csharp
public class OrderDAL
{
    private readonly ShardRouter _router;

    public Order GetOrderByUserId(int userId, int orderId)
    {
        string connStr = _router.GetConnectionStringForUser(userId); // pick correct shard

        using var conn = new SqlConnection(connStr);
        using var cmd = new SqlCommand("sp_GetOrderById", conn) { CommandType = CommandType.StoredProcedure };
        cmd.Parameters.AddWithValue("@OrderId", orderId);

        conn.Open();
        using var reader = cmd.ExecuteReader();
        return MapToOrder(reader);
    }
}
```

### Practical — aggregating a cross-shard query in application code

```csharp
public int GetTotalOrderCountAcrossAllShards()
{
    int total = 0;

    foreach (var connStr in _router.AllShardConnectionStrings)
    {
        using var conn = new SqlConnection(connStr);
        using var cmd = new SqlCommand("sp_GetOrderCount", conn) { CommandType = CommandType.StoredProcedure };

        conn.Open();
        total += (int)cmd.ExecuteScalar(); // sum results from every shard, in app code
    }

    return total;
}
```

## ⚡ Performance considerations

- Sharding scales **both reads and writes**, unlike replication (reads only) — the biggest reason to reach for it once write volume is the bottleneck.
- **Resharding** (changing the number of shards later) is expensive and risky — hash-based sharding in particular requires re-hashing and moving huge amounts of data. Consistent Hashing (`F.Distributed_Systems/02_Consistent_Hashing.md`) exists specifically to ease this pain.
- A poorly chosen shard key causes **hotspots** — one shard gets disproportionately more traffic than others, defeating the purpose.

## 🚨 Common mistakes

- ❌ Sharding too early — it adds huge operational complexity; most apps should exhaust simpler options (indexing, caching, replication, vertical scaling) first.
- ❌ Choosing a shard key that creates hotspots (e.g., sharding by `CreatedDate` when all new writes naturally land on the newest/"hot" shard).
- ❌ Designing queries that assume cross-shard JOINs/transactions are cheap — they're not, and often aren't even directly supported.

## 💡 Best practices

- ✅ Exhaust caching, indexing, and replication before reaching for sharding — it's a heavy, hard-to-reverse decision.
- ✅ Choose a shard key that distributes load evenly **and** matches your most common query pattern (e.g., shard by `UserId` if most queries are "get this user's data").
- ✅ Keep related data that's often queried together **on the same shard** (denormalize if needed) to avoid cross-shard joins.
- ✅ Plan for resharding from day one (e.g., via consistent hashing) — datasets grow, and shard count will eventually need to change.

## 🎤 Interview Quick-Fire Q&A

| Question                                                     | Answer                                                                                                                                                         |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the core difference between Replication and Sharding? | Replication copies the*entire* dataset to multiple servers (scales reads); Sharding splits the dataset into subsets across servers (scales reads AND writes) |
| What is a shard key?                                         | The column used to determine which shard a given row belongs to                                                                                                |
| Why are cross-shard JOINs a problem?                         | Data lives on physically separate databases — a JOIN can't span them natively; must be done in application code                                               |
| What causes a "hotspot" in sharding?                         | A poorly chosen shard key that sends disproportionate traffic to one shard instead of distributing evenly                                                      |
| Name two common sharding strategies.                         | Range-based and Hash-based (also: Directory-based, Geo-based)                                                                                                  |

## 📝 30-second Revision Cheat Sheet

- Sharding = split the dataset itself across multiple independent DB servers.
- Scales BOTH reads and writes (Replication only scales reads).
- Shard key decides which shard a row lives on — choosing it well is critical.
- Strategies: Range-based, Hash-based, Directory-based, Geo-based.
- Cross-shard JOINs/transactions are hard — avoid by design, don't rely on them.
- Heavy, hard-to-reverse decision — use only after simpler scaling options are exhausted.
