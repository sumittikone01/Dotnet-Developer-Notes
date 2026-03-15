
# 03 — Using Statement Prevents Leaks

---

## 🎯 One-Line Definition

> **The `using` statement guarantees that `Dispose()` is called on a database object — whether the code succeeds or throws an exception — preventing connection leaks, memory leaks, and pool exhaustion.**

---

## 🔷 What is a Resource Leak?

```
A resource leak happens when your code OPENS something
but never CLOSES it — usually because an exception fired
before the Close() line was reached.

In ADO.NET, the most dangerous leaks are:
  → SqlConnection left open  = connection pool fills up → app crashes
  → SqlDataReader not closed  = server cursor held open → locks tables
  → SqlCommand not disposed   = minor memory leak
```

---

## 🔷 The Problem — Exception Before Close

```csharp
// ❌ DANGEROUS — what if an exception happens?
SqlConnection con = new SqlConnection(_cs);
con.Open();

SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con);
SqlDataReader reader = cmd.ExecuteReader();

// ← What if an exception fires HERE? ───────────────────────────
//   All the Close() calls below NEVER EXECUTE.

while (reader.Read()) { ... }

reader.Close();   // ← NEVER REACHED if exception above
cmd.Dispose();    // ← NEVER REACHED
con.Close();      // ← NEVER REACHED — connection stuck in pool forever!
```

---

## 🔷 The Fix — `using` Block

```csharp
// ✅ SAFE — using guarantees cleanup even on exception
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
    {
        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            // Exception can fire anywhere inside here...
            while (reader.Read()) { ... }
        }   // ← reader.Dispose() ALWAYS called here
    }       // ← cmd.Dispose() ALWAYS called here
}           // ← con.Dispose() = con.Close() ALWAYS called here
```

---

## 🔷 What `using` Compiles To

The compiler transforms every `using` block into a `try/finally`:

```csharp
// What YOU write:
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    // ... work ...
}

// What the COMPILER generates:
SqlConnection con = new SqlConnection(_cs);
try
{
    con.Open();
    // ... work ...
}
finally
{
    if (con != null)
        con.Dispose();   // ← runs ALWAYS — success or exception
}
```

> `finally` block always executes — even if an exception is thrown.
> This is why `using` is safe — it guarantees cleanup via `finally`.

---

## 🔷 What Happens on Each Dispose

```csharp
SqlConnection.Dispose()
  → Calls Close()
  → Returns connection to pool
  → Marks as available for next request

SqlCommand.Dispose()
  → Releases the command object
  → Clears parameter collection from memory

SqlDataReader.Dispose()
  → Closes the server-side cursor
  → Releases the result set held on SQL Server
  → Frees memory used to buffer rows

SqlDataAdapter.Dispose()
  → Releases internal SqlCommand objects
  → Frees unmanaged resources
```

---

## 🔷 Two `using` Syntax Styles

### Style 1 — Classic block (C# all versions)

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    using (SqlCommand cmd = new SqlCommand(sql, con))
    {
        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read()) { ... }
        }
    }
}
```

### Style 2 — Declaration syntax (C# 8+, simpler)

```csharp
using SqlConnection con = new SqlConnection(_cs);
con.Open();

using SqlCommand cmd = new SqlCommand(sql, con);
using SqlDataReader reader = cmd.ExecuteReader();

while (reader.Read()) { ... }
// All three disposed at end of enclosing method
```

### Style 3 — Collapsed nesting (same result, less indentation)

```csharp
using SqlConnection con = new SqlConnection(_cs);
con.Open();
using SqlCommand cmd    = new SqlCommand(sql, con);
using SqlDataReader rdr = cmd.ExecuteReader();

while (rdr.Read())
{
    // ...
}
```

---

## 🔷 All ADO.NET Objects That Need `using`

```csharp
// Every IDisposable in ADO.NET — always wrap in using:

using (SqlConnection    con  = new SqlConnection(_cs))    { }
using (SqlCommand       cmd  = new SqlCommand(sql, con))  { }
using (SqlDataReader    rdr  = cmd.ExecuteReader())       { }
using (SqlDataAdapter   da   = new SqlDataAdapter(sql,con)){ }
using (SqlCommandBuilder cb  = new SqlCommandBuilder(da)) { }
using (SqlTransaction   tx   = con.BeginTransaction())    { }

// These implement IDisposable — using calls Dispose automatically
```

---

## 🔷 Complete Production Pattern

```csharp
public List<Employee> GetAll()
{
    var list = new List<Employee>();
    string sql = "SELECT Id, Name, Role, Salary FROM Employee";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.CommandTimeout = 30;

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    list.Add(new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Salary = reader.GetDecimal(3)
                    });
                }
            }   // reader disposed here
        }       // cmd disposed here
    }           // con closed and returned to pool here

    return list;
}
```

---

## 🔷 `using` vs `try/finally` — Are They the Same?

```csharp
// These are IDENTICAL in behaviour:

// using block:
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
}

// try/finally equivalent:
SqlConnection con = new SqlConnection(_cs);
try   { con.Open(); }
finally { con?.Dispose(); }

// ✅ Use using — it's shorter, cleaner, and the compiler handles the rest
```

---

## 🔷 ❗ Important — Two Meanings of `using` Keyword

```csharp
// Meaning 1 — Namespace import (at top of file)
using System.Data;
using Microsoft.Data.SqlClient;

// Meaning 2 — Resource management (inside methods)
using (SqlConnection con = new SqlConnection(_cs))
{
    // ...
}

// They look similar but are completely different!
// Namespace import → makes classes available
// Resource management → guarantees Dispose() is called
```

---

## 🔷 Without `using` — The Real Impact

```
One request leaks a connection:
  Pool has 100 connections
  99 available → 98 → 97 → ... → 0
  101st request: waiting... waiting... waiting...
  After 15 seconds: "Timeout expired obtaining connection from pool"
  App appears broken for all users

The cause: one missing using block somewhere in the code.
```

---

## ⚠️ Common Mistakes

| Mistake                                              | What Happens                                      | Fix                                         |
| ---------------------------------------------------- | ------------------------------------------------- | ------------------------------------------- |
| `con.Close()`in `try`without `finally`         | Close not called if exception fires               | Use `using`block — compiler handles it   |
| No `using`on `SqlDataReader`                     | Server cursor held open → table locks accumulate | Always `using`on reader                   |
| Thinking `using`closes the physical TCP connection | It returns to pool — it stays open               | This is correct behaviour — pool reuses it |
| Nested `using`without inner `using`on reader     | Reader leaks if exception fires after Open        | Every IDisposable gets its own `using`    |

---

## ⭐ Interview Quick-Fire

| Question                                        | Answer                                                                      |
| ----------------------------------------------- | --------------------------------------------------------------------------- |
| What does `using`guarantee?                   | `Dispose()`is always called — even if an exception is thrown             |
| What does `using`compile to?                  | A `try/finally`block where `Dispose()`is in the `finally`             |
| What does `con.Dispose()`do?                  | Calls `Close()`→ returns connection to pool — does NOT destroy it       |
| What happens without `using`on SqlConnection? | Connection leaks — pool exhausted — timeout exceptions for all users      |
| Name three ADO.NET objects that need `using`? | `SqlConnection`,`SqlCommand`,`SqlDataReader`                          |
| Two meanings of `using`keyword?               | Namespace import (top of file) and resource management (inside methods)     |
| Is `using`available in all C# versions?       | Block syntax: yes always. Declaration syntax (`using var x = ...`): C# 8+ |
