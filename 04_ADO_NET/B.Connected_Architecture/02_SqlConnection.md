
# 02 — SqlConnection

---

## 🎯 One-Line Definition

> **`SqlConnection` is the gateway to SQL Server — it opens the channel to the database and must always be wrapped in a `using` block so it closes automatically, even if an exception occurs.**

---

## 🔷 What is SqlConnection?

`SqlConnection` is the class that  **opens and manages a connection to SQL Server** .
It is the **first step** in any ADO.NET operation — nothing works until you have a connection open.

> 💡 Think of `SqlConnection` as the **door** to your database. You must open it to get in, and you must close it when you're done — or you'll block others from entering.

---

## 🔷 Basic Syntax

```csharp
string cs = @"Server=.\SQLEXPRESS; Database=EmployeeDB; Trusted_Connection=True; TrustServerCertificate=True;";

using (SqlConnection con = new SqlConnection(cs))
{
    con.Open();
    // ... do work here
}   // ← con.Close() + con.Dispose() called automatically
```

---

## 🔷 SqlConnection Properties

| Property              | Returns             | Example                                       |
| --------------------- | ------------------- | --------------------------------------------- |
| `ConnectionString`  | `string`          | The connection string text                    |
| `ConnectionTimeout` | `int`             | Wait time in seconds (default: 15)            |
| `Database`          | `string`          | Current database name:`"EmployeeDB"`        |
| `DataSource`        | `string`          | Server name:`".\SQLEXPRESS"`                |
| `ServerVersion`     | `string`          | SQL Server version:`"15.00.2000"`           |
| `State`             | `ConnectionState` | `Open`,`Closed`,`Connecting`,`Broken` |

```csharp
con.Open();
Console.WriteLine(con.Database);       // EmployeeDB
Console.WriteLine(con.DataSource);     // .\SQLEXPRESS
Console.WriteLine(con.ServerVersion);  // 15.00.2000
Console.WriteLine(con.State);          // Open
```

---

## 🔷 SqlConnection Methods

| Method                   | Purpose                                            |
| ------------------------ | -------------------------------------------------- |
| `Open()`               | Opens the connection                               |
| `Close()`              | Closes the connection (returns to pool)            |
| `Dispose()`            | Releases all resources — called by `using`      |
| `CreateCommand()`      | Creates a new SqlCommand linked to this connection |
| `BeginTransaction()`   | Starts a database transaction                      |
| `ChangeDatabase(name)` | Switches to a different database on same server    |
| `OpenAsync()`          | Opens connection asynchronously (async/await)      |

---

## 🔷 The `using` Block — Why It Matters

### ❌ Without `using` — Dangerous

```csharp
SqlConnection con = new SqlConnection(cs);
con.Open();
SomeMethod();       // ← if exception thrown here...
con.Close();        // ← NEVER REACHED
con.Dispose();      // ← NEVER REACHED
// Result: connection leak → pool fills up → app crashes
```

### ✅ With `using` — Always Safe

```csharp
using (SqlConnection con = new SqlConnection(cs))
{
    con.Open();
    SomeMethod();   // even if this throws an exception...
}                   // ← con.Close() + con.Dispose() ALWAYS called here
```

### What `using` Compiles to Internally

```csharp
SqlConnection con = new SqlConnection(cs);
try
{
    con.Open();
    SomeMethod();
}
finally
{
    con.Dispose();   // ← runs ALWAYS — success or exception
}
```

> ✅ `Dispose()` calls `Close()` — so you don't need both. `using` handles everything.

---

## 🔷 Two `using` Syntax Styles — Both Valid

```csharp
// Style 1 — Block syntax (traditional, explicit scope)
using (SqlConnection con = new SqlConnection(cs))
{
    con.Open();
    // ... work here
}   // con disposed here — scope is clear

// Style 2 — Declaration syntax (C# 8+, simpler)
using SqlConnection con = new SqlConnection(cs);
con.Open();
// ... work here
// con disposed automatically at end of containing method
```

---

## ❗ Important — Two Different `using` Keywords

```csharp
// using Type 1 — Namespace import (top of file)
using System.Data;
using Microsoft.Data.SqlClient;

// using Type 2 — Resource management (in methods)
using (SqlConnection con = new SqlConnection(cs))
{
    // ...
}
```

|                       | `using System.Data;`  | `using (SqlConnection con = ...)` |
| --------------------- | ----------------------- | ----------------------------------- |
| What it is            | Namespace import        | Resource management statement       |
| Where it goes         | Top of file             | Inside methods                      |
| What it does          | Makes classes available | Auto-closes and disposes            |
| Related to C# version | Always available        | Block: always / Declaration: C# 8+  |

---

## 🔷 Connection State — Check Before Opening

```csharp
using (SqlConnection con = new SqlConnection(cs))
{
    // Check state before opening
    if (con.State == ConnectionState.Closed)
    {
        con.Open();
    }

    Console.WriteLine(con.State);   // Open
    Console.WriteLine(con.Database); // EmployeeDB
}
Console.WriteLine(con.State);       // Closed (after using block)
```

---

## 🔷 Complete Real Example in MVC — Repository Pattern

```csharp
using System.Data;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;

public class EmployeeRepository
{
    private readonly string _cs;

    // IConfiguration reads appsettings.json automatically via DI
    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    public int GetEmployeeCount()
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();

            // Optional: verify connection is open
            if (con.State == ConnectionState.Open)
            {
                Console.WriteLine($"Connected to: {con.Database}");
            }

            using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
            {
                return (int)cmd.ExecuteScalar();
            }
        }   // con.Close() called automatically here
    }
}
```

```csharp
// Program.cs — register so DI injects IConfiguration automatically
builder.Services.AddScoped<EmployeeRepository>();
```

---

## 🔷 Without `using` vs With `using` — Summary

|              | Without `using`                      | With `using`                      |
| ------------ | -------------------------------------- | ----------------------------------- |
| Cleanup      | Manual — you must call Close/Dispose  | Automatic — compiler guarantees it |
| On exception | Close() NOT called — connection leaks | Dispose() ALWAYS called via finally |
| Pool impact  | Connection stays checked out           | Connection returned to pool         |
| Recommended  | ❌ No                                  | ✅ Yes — always                    |

---

## ⭐ Interview Quick-Fire

| Question                                         | Answer                                                                                                      |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| What is SqlConnection?                           | Class that opens and manages a connection to SQL Server                                                     |
| Why use `using`with SqlConnection?             | Guarantees Close() + Dispose() even if exception thrown                                                     |
| What does `using`compile to?                   | try { } finally { con.Dispose(); }                                                                          |
| Does Close() return to pool?                     | ✅ Yes — doesn't destroy, returns to connection pool                                                       |
| Two syntax styles for `using`?                 | Block `using (var x = ...) { }`and declaration `using var x = ...;`                                     |
| Difference: ConnectionTimeout vs CommandTimeout? | ConnectionTimeout = wait to open (connection string). CommandTimeout = wait for query (SqlCommand property) |
| How to check if connection is open?              | `con.State == ConnectionState.Open`                                                                       |
| Where to store connection string?                | `appsettings.json`→`ConnectionStrings`section                                                          |
| How to read it in code?                          | `config.GetConnectionString("DefaultConnection")`                                                         |
