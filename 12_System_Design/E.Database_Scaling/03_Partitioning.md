# 03_Partitioning

> **Partitioning** = splitting a large table into smaller, more manageable pieces (**partitions**) — usually **within the same database server/instance** — so the engine can query, maintain, and store the data more efficiently.

> ⚠️ **This is the most confused pair of terms in System Design.** `02_Sharding.md` and Partitioning both "split data into pieces," but they answer different questions. Read the comparison table below first — it will save you from mixing them up in an interview.

## 📌 What is it?

A single logical table (say, `Orders` with 200 million rows) is physically broken into multiple **partitions** — each partition still lives on the **same SQL Server instance**, but the engine treats each partition as a separate, smaller physical unit internally (often, but not always, mapped to separate files/filegroups).

```
Logical view (what your queries see):        Physical reality (what SQL Server stores):

┌─────────────────────────┐                  ┌───────────┐ ┌───────────┐ ┌───────────┐
│   Orders (200M rows)     │       ═══►       │Partition 1│ │Partition 2│ │Partition 3│
│  (one table, as far as   │                  │(2023 rows)│ │(2024 rows)│ │(2025 rows)│
│   your queries know)     │                  └───────────┘ └───────────┘ └───────────┘
└─────────────────────────┘                          all on the SAME SQL Server instance
```

## 🚨 Sharding vs Partitioning — THE critical distinction

| Aspect                                         | **Partitioning**                                                                     | **Sharding**                                                                   |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| Where do the pieces live?                      | Same database server/instance                                                              | Different, independent servers                                                       |
| Is it visible to the application?              | Usually**no** — the DB engine handles it transparently; app still queries one table | **Yes** — the application (or a routing layer) must know which shard to query |
| What does it solve?                            | Query performance, maintenance (index rebuilds, backups) on ONE large table                | Scaling beyond what a single server's CPU/disk/memory can handle                     |
| Does it help write throughput across machines? | No — still one server, one write bottleneck                                               | Yes — writes spread across multiple independent servers                             |
| Complexity                                     | Lower — mostly a DBA/schema-level configuration                                           | Higher — application-level routing, cross-shard query logic                         |
| Analogy                                        | One warehouse with labeled, organized aisles                                               | Multiple separate warehouses in different cities                                     |

> 🧠 **Memory shortcut:** *Partitioning organizes data within one house. Sharding puts data in different houses.* A table can be partitioned **and** each of those shards can *also* be partitioned internally — the two techniques compose.

## 🤔 Why do we need it?

- A 200-million-row table with **one** index on `OrderDate` still means SQL Server may have to scan huge portions of the index for date-range queries.
- Maintenance operations (rebuilding an index, running backups) on a giant single table are slow and can lock/block production traffic.
- Partitioning lets the engine **skip entire partitions** that don't match a query's filter — a huge performance win (called "partition pruning" / "partition elimination").

## 🌍 Real-world analogy

A **filing cabinet with labeled drawers by year** (2023, 2024, 2025) instead of one giant unsorted pile of paper in a single drawer. Looking for a 2024 invoice? Open only the 2024 drawer — you never touch the others. It's still the *same cabinet, same room* (same server) — just organized.

## ⚙️ Internal working — Partition Pruning

```
Query: SELECT * FROM Orders WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01'

Without partitioning:
   Engine scans across the ENTIRE 200M-row table/index

With partitioning by year:
   Engine knows this query only needs "Partition_2024"
   → skips Partition_2023 and Partition_2025 entirely
   → scans a much smaller dataset          ← "Partition Pruning"
```

## 📊 Types of Partitioning

| Type                              | Split axis                                              | Example                                                                                                                                                                     |
| --------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Horizontal Partitioning** | By rows (same columns, subset of rows per partition)    | `Orders` split by `OrderDate` year — this is what "Sharding" is a *distributed* version of                                                                           |
| **Vertical Partitioning**   | By columns (same rows, subset of columns per partition) | Split`Users` into `Users_Core` (Id, Name, Email) and `Users_Profile` (Bio, AvatarUrl, Preferences) — rarely-used columns stored separately from frequently-used ones |

```
Horizontal Partitioning:                Vertical Partitioning:

Orders                                  Users
┌────┬──────┬────────────┐              ┌────┬──────┬───────┬──────────┐
│ Id │ ...  │ OrderDate  │              │ Id │ Name │ Email │   Bio    │
├────┼──────┼────────────┤              └────┴──────┴───────┴──────────┘
│ .. │ ...  │ 2023-xx-xx │  → Part_2023          split into:
│ .. │ ...  │ 2024-xx-xx │  → Part_2024   Users_Core (Id,Name,Email)
│ .. │ ...  │ 2025-xx-xx │  → Part_2025   Users_Profile (Id, Bio)
└────┴──────┴────────────┘
   (rows split)                            (columns split)
```

## 💻 Code examples

### Basic — SQL Server horizontal partitioning by date range

```sql
-- 1. Define a partition function: how to split the values
CREATE PARTITION FUNCTION OrderDateRangePF (DATE)
AS RANGE RIGHT FOR VALUES ('2023-01-01', '2024-01-01', '2025-01-01');

-- 2. Define a partition scheme: which filegroup each partition lives in
CREATE PARTITION SCHEME OrderDatePS
AS PARTITION OrderDateRangePF
ALL TO ([PRIMARY]);

-- 3. Create the table ON the partition scheme, using OrderDate as the partitioning column
CREATE TABLE Orders
(
    OrderId     INT           NOT NULL,
    CustomerId  INT           NOT NULL,
    OrderDate   DATE          NOT NULL,
    Amount      DECIMAL(10,2) NOT NULL
)
ON OrderDatePS(OrderDate);

-- From here, the application's stored procedures don't change AT ALL —
-- SQL Server transparently routes rows to the correct partition.
```

### Intermediate — stored procedure is completely unaware of partitioning

```sql
-- sp_GetOrdersByDateRange — looks like a normal query
CREATE PROCEDURE sp_GetOrdersByDateRange
    @StartDate DATE,
    @EndDate DATE
AS
BEGIN
    SELECT OrderId, CustomerId, OrderDate, Amount
    FROM Orders
    WHERE OrderDate >= @StartDate AND OrderDate < @EndDate;
    -- SQL Server automatically prunes to only the relevant partition(s)
    -- Your ADO.NET/DAL code calling this SP needs ZERO changes for partitioning
END
```

> Compare this to Sharding's `ShardRouter` from `02_Sharding.md` — there, the **application** had to know which shard to hit. Here, partitioning is entirely the **database engine's** concern; your C# BAL/DAL code is unaware it's even happening.

## ⚡ Performance considerations

- **Partition pruning** dramatically speeds up range-filtered queries on huge tables.
- Index maintenance (rebuild/reorganize) can be done **per-partition**, avoiding full-table locks during maintenance windows.
- Old partitions (e.g., data older than 3 years) can be **archived or dropped as a whole partition** — near-instant compared to a `DELETE` of millions of rows.
- Does **not** help if your server's total capacity (CPU/RAM/disk) itself is the bottleneck — that's when you need Sharding.

## 🚨 Common mistakes

- ❌ Using the terms "Sharding" and "Partitioning" interchangeably — in interviews, this is an immediate red flag that the distinction isn't understood.
- ❌ Partitioning by a column that queries rarely filter on — you lose the "pruning" benefit entirely (queries still have to check every partition).
- ❌ Expecting partitioning to solve a single-server capacity ceiling — it optimizes within one server, it doesn't add more servers.

## 💡 Best practices

- ✅ Partition by the column your queries **most commonly filter by** (often a date column for time-series data like orders, logs, events).
- ✅ Use horizontal partitioning for "big table, need faster range queries / easier archival" problems.
- ✅ Use vertical partitioning when a table has a mix of frequently-accessed and rarely-accessed columns (e.g., separate a large `Bio`/`ProfileImage` blob column from hot login-check columns).
- ✅ Remember: partitioning first, sharding only if partitioning + a single beefier server still isn't enough.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                               | Answer                                                                                                                 |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| What's the#1 difference between Partitioning and Sharding?                             | Partitioning splits a table within the SAME server; Sharding splits data across DIFFERENT servers                      |
| Is partitioning visible to the application code?                                       | No — the database engine handles it transparently; the app queries the table normally                                 |
| What is "partition pruning"?                                                           | The query engine skipping partitions that can't contain relevant rows, based on the query's filter                     |
| What's the difference between horizontal and vertical partitioning?                    | Horizontal splits by rows (subset of rows per partition); Vertical splits by columns (subset of columns per partition) |
| Does partitioning help if a single server's total CPU/disk capacity is the bottleneck? | No — that requires Sharding (adding more servers), not partitioning (reorganizing within one server)                  |

## 📝 30-second Revision Cheat Sheet

- Partitioning = splitting a table into pieces **within the same server** (transparent to the app).
- Sharding = splitting data across **different servers** (application must know how to route).
- Horizontal partitioning = split by rows (e.g., by date); Vertical = split by columns.
- "Partition pruning" = engine skips irrelevant partitions → big performance win for range queries.
- Partition first (cheap, transparent); shard only when a single server truly can't handle the load.
