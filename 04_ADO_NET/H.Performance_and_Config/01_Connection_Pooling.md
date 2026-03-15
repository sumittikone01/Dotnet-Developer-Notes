
# 01 — Connection Pooling

---

## 🎯 One-Line Definition

> **Connection Pooling reuses existing database connections instead of creating a new one every request — creating a connection takes ~200ms, reusing from a pool takes ~1ms, making it one of the most impactful performance features in ADO.NET.**

---

## 🔷 The Problem Without Pooling

```
WITHOUT pooling (every request creates a new connection):
──────────────────────────────────────────────────────────────────
Request 1:  Create TCP connection → Auth → Allocate → Query → DESTROY
Request 2:  Create TCP connection → Auth → Allocate → Query → DESTROY
Request 3:  Create TCP connection → Auth → Allocate → Query → DESTROY
...
100 users = 100 connection creations = ~20 seconds wasted just connecting

WITH pooling (connections reused from pool):
──────────────────────────────────────────────────────────────────
Startup:    Create 5 connections → held in pool
Request 1:  Take from pool → Query → Return to pool  (~1ms)
Request 2:  Take from pool → Query → Return to pool  (~1ms)
Request 3:  Take from pool → Query → Return to pool  (~1ms)
...
100 users = 100 reuses = milliseconds wasted connecting
```

---

## 🔷 How Connection Pooling Works

```
FIRST REQUEST:
  con.Open()
    → Pool is empty
    → ADO.NET creates a real TCP connection to SQL Server (~200ms)
    → Connection added to pool
    → Given to your code

  con.Close() or using block ends
    → Connection NOT destroyed
    → Returned to pool ← sitting idle, ready for next request

SECOND REQUEST (same connection string):
  con.Open()
    → Pool has an idle connection
    → Reused instantly (~1ms)
    → Given to your code

  con.Close() → returned to pool again
```

---

## 🔷 Pooling is ON by Default

```csharp
// You don't have to do anything — pooling is enabled automatically.
// Every SqlConnection with the same connection string shares a pool.

string cs = "Server=.;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";

// These two connections use the SAME POOL:
using (SqlConnection con1 = new SqlConnection(cs)) { con1.Open(); }
using (SqlConnection con2 = new SqlConnection(cs)) { con2.Open(); }
// con1 closes → returned to pool
// con2 opens → gets con1's physical connection from pool instantly
```

---

## 🔷 Connection String Pooling Keywords

```csharp
// Full connection string with pooling settings:
string cs = @"Server=.\SQLEXPRESS;
              Database=EmployeeDB;
              Trusted_Connection=True;
              TrustServerCertificate=True;
              Pooling=True;          ← ON by default — don't need to set
              Min Pool Size=5;       ← keep 5 connections alive always
              Max Pool Size=100;     ← never exceed 100 connections
              Connection Lifetime=300; ← recycle connections older than 300s
              Connect Timeout=15;    ← wait max 15s for a pool connection";
```

| Keyword                 | Default          | Meaning                                    |
| ----------------------- | ---------------- | ------------------------------------------ |
| `Pooling`             | `True`         | Enable/disable pooling                     |
| `Min Pool Size`       | `0`            | Minimum connections kept alive in pool     |
| `Max Pool Size`       | `100`          | Maximum connections in pool                |
| `Connection Lifetime` | `0`(unlimited) | Seconds before a connection is recycled    |
| `Connect Timeout`     | `15`           | Seconds to wait for a connection from pool |

---

## 🔷 The Golden Rule — Always Close Connections

```csharp
// ❌ WRONG — connection never returned to pool
SqlConnection con = new SqlConnection(cs);
con.Open();
// ... do work ...
// Exception thrown here — Close() never called
// Connection is CHECKED OUT of pool, never returned
// Pool fills up → other requests time out waiting

// ✅ CORRECT — using block guarantees return to pool
using (SqlConnection con = new SqlConnection(cs))
{
    con.Open();
    // ... do work ...
}
// using block exits → con.Dispose() called → Close() called
// → connection returned to pool ← every time, even on exception
```

> 📌 `con.Close()` and `con.Dispose()` do NOT destroy the physical connection — they **return it to the pool** for the next request.

---

## 🔷 What Happens When Pool is Full

```
Max Pool Size = 100 (default)

101st request arrives:
  → Pool has 100 connections all checked out
  → ADO.NET waits for one to be returned
  → Waits up to Connect Timeout (15 seconds default)
  → If nothing returned in 15 seconds:
     → InvalidOperationException:
       "Timeout expired. The timeout period elapsed prior to
        obtaining a connection from the pool."

This exception = connection leak somewhere in your code.
Someone opened a connection and never closed it.
```

---

## 🔷 Pool Per Connection String

```csharp
// IMPORTANT: each unique connection string gets its OWN pool

string cs1 = "Server=.;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";
string cs2 = "Server=.;Database=HRDatabase;Trusted_Connection=True;TrustServerCertificate=True;";
string cs3 = "Server=.;Database=EmployeeDB;User Id=sa;Password=123;";

// cs1 → Pool A
// cs2 → Pool B  (different database)
// cs3 → Pool C  (different auth)

// Even a single extra space in the string = different pool:
string cs4 = "Server=. ;Database=EmployeeDB;...";  // ← space after dot = different pool!
// Always keep connection strings consistent and identical
```

---

## 🔷 Clearing the Pool (Rarely Needed)

```csharp
// Clear pool for one specific connection
SqlConnection.ClearPool(con);

// Clear ALL pools (for all connection strings)
SqlConnection.ClearAllPools();

// When to use:
// → After SQL Server restart (stale connections)
// → During testing/teardown
// → After changing SQL Server credentials
// Never use in normal app flow — defeats the purpose of pooling
```

---

## 🔷 Min Pool Size — Warm Pool Pattern

```csharp
// Set Min Pool Size to keep connections alive during quiet periods
// Avoids the cold-start penalty on the first request after idle time

string cs = @"Server=.\SQLEXPRESS;Database=EmployeeDB;
              Trusted_Connection=True;TrustServerCertificate=True;
              Min Pool Size=5;
              Max Pool Size=50;";

// With Min Pool Size=5:
//   App starts → 5 connections created immediately
//   First 5 requests → instant response (no connection creation)
//   Pool never drops below 5 even if app is idle
```

---

## 🔷 Monitoring Pool Health

```csharp
// Check connection state before using
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    Console.WriteLine($"Database:    {con.Database}");
    Console.WriteLine($"Server:      {con.DataSource}");
    Console.WriteLine($"State:       {con.State}");        // Open
    Console.WriteLine($"Timeout:     {con.ConnectionTimeout}s");

    // con.State values: Closed, Open, Connecting, Broken, Executing, Fetching
}
```

---

## ⚠️ Common Mistakes

| Mistake                                        | What Happens                                                    | Fix                                                                |
| ---------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------ |
| Not using `using`block                       | Connection leaks — pool fills up → timeout exceptions         | Always use `using (SqlConnection con = ...)`                     |
| Inconsistent connection strings (spaces, case) | Each variation creates its own pool — wasteful                 | Keep connection string identical — read from `appsettings.json` |
| `Max Pool Size`too low for traffic           | Requests queue waiting for pool — slow response                | Tune `Max Pool Size`based on expected concurrent users           |
| Calling `ClearAllPools()`in request handler  | Destroys all pools on every request — catastrophic performance | Only call `ClearPool`during maintenance/startup                  |

---

## ⭐ Interview Quick-Fire

| Question                                              | Answer                                                                 |
| ----------------------------------------------------- | ---------------------------------------------------------------------- |
| What is connection pooling?                           | Reusing existing connections instead of creating new ones each request |
| Is pooling on by default?                             | ✅ Yes — enabled automatically, no configuration needed               |
| What does `con.Close()`actually do?                 | Returns the connection to the pool — does NOT destroy it              |
| What causes "timeout obtaining connection from pool"? | Pool is full — connection leak — someone opened without closing      |
| How many pools does ADO.NET create?                   | One per unique connection string                                       |
| What is `Min Pool Size`?                            | Minimum connections kept alive even when idle                          |
| What is `Max Pool Size`?                            | Maximum connections allowed (default 100)                              |
| How to prevent connection leaks?                      | Always use `using`block for `SqlConnection`                        |
