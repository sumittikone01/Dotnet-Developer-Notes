# 01 — What is ADO.NET

---

## 🎯 One-Line Definition

> **ADO.NET is Microsoft's built-in data access technology for .NET — it bridges your C# code and SQL Server, giving you full control to read, write, update, and delete data using raw SQL with two models: Connected (live, fast) and Disconnected (offline, scalable).**

---

## 🤔 The Problem ADO.NET Solves

Your C# application speaks C#. Your database speaks SQL. They can't talk directly.

```
WITHOUT ADO.NET:
────────────────────────────────────────────────────────────
C# App: "I need all employees from the IT department"
           ↓
SQL Server: ???  ← can't understand C# objects
No connection. No data. Nothing works.


WITH ADO.NET:
────────────────────────────────────────────────────────────
C# App: "I need IT department employees"
           ↓
ADO.NET: SqlConnection + SqlCommand + SqlDataReader
           ↓ translates to SQL
SQL Server: "SELECT * FROM Employees WHERE Department = 'IT'"
           ↓ returns rows
ADO.NET: maps rows back to C# objects
           ↓
C# App: List<Employee> — ready to use
```

---

## 🔑 What ADO.NET Stands For

```
ADO = ActiveX Data Objects
.NET = the .NET platform

Originally "ADO" was Microsoft's COM-based data access (Visual Basic era).
ADO.NET is the complete .NET rewrite — nothing in common except the name.
```

---

## 🔑 Where ADO.NET Lives in Your Project

In an ASP.NET Core MVC project with the Controller → BAL → DAL pattern:

```
┌─────────────────────────────────────────────────────────┐
│                  ASP.NET Core MVC App                   │
│                                                         │
│  Browser → Controller → BAL (Business Logic) → DAL     │
│                                                  ↓      │
│                                          ADO.NET Code   │
│                              (SqlConnection, SqlCommand) │
│                                                  ↓      │
│                                       SQL Server        │
└─────────────────────────────────────────────────────────┘

ADO.NET code ONLY lives in:
  ✅ DAL (Data Access Layer)        ← if three-layer architecture
  ✅ Repository class               ← if repository pattern
  ❌ Controller                     ← never put DB code here
  ❌ View / Razor Page              ← never
  ❌ BAL / Service Layer            ← business logic only, no DB
```

---

## 🔑 The Two Architecture Models

ADO.NET has two fundamentally different ways to work with data:

```
┌──────────────────────────────────────────────────────────────────┐
│                       ADO.NET                                     │
│                                                                    │
│    CONNECTED MODEL              DISCONNECTED MODEL                │
│    ─────────────────            ───────────────────               │
│                                                                    │
│    Connection stays OPEN        Connection closes after fetch     │
│    while you read data          Data lives in memory              │
│                                                                    │
│    SqlConnection                SqlConnection                     │
│    SqlCommand                   SqlCommand                        │
│    SqlDataReader  ←── reads     SqlDataAdapter ←── fills         │
│                                 DataSet / DataTable               │
│                                                                    │
│    Like: drinking water         Like: filling a bottle and        │
│    directly from the tap        taking it with you                │
└──────────────────────────────────────────────────────────────────┘
```

### Connected Model

```
Open Connection → Execute → Read Row by Row → Close Connection

✅ Real-time data — always fresh from DB
✅ Low memory — reads one row at a time
✅ Fast for quick reads
❌ Connection tied up while reading
❌ Less scalable under heavy load

Best for: login checks, live lookups, quick reads
```

### Disconnected Model

```
Open → Fetch ALL data into DataSet → Close → Edit Offline → Reopen → Save → Close

✅ Connection freed immediately after fetch
✅ Edit data without DB connection
✅ Highly scalable — connection open for milliseconds
✅ Perfect for Kendo Grid (loads data, connection already closed)
❌ Data is a snapshot — not real-time
❌ More memory (all rows loaded)

Best for: Kendo Grids, reports, data editing forms
```

---

## 🔑 ADO.NET vs Entity Framework — Know Both

|                    | ADO.NET                       | Entity Framework                          |
| ------------------ | ----------------------------- | ----------------------------------------- |
| Level              | Low-level, direct SQL         | High-level ORM (Object Relational Mapper) |
| SQL                | You write SQL manually        | EF generates SQL automatically            |
| Control            | Full control over every query | Less control, more abstraction            |
| Performance        | Faster — no ORM overhead     | Slightly slower                           |
| Code amount        | More code required            | Less code required                        |
| Learning curve     | Steeper                       | Easier to start                           |
| Stored procedures  | First-class support           | Possible but awkward                      |
| Used in your stack | ✅ Yes — daily               | Future learning                           |

> **Why must you learn ADO.NET even when EF exists?**
> Every senior .NET developer must know it. Real production apps use it for stored procedures, bulk operations, fine-tuned queries, and performance-critical code where EF adds too much overhead.

---

## 🔑 Core Namespaces

```csharp
using System.Data;                    // DataSet, DataTable, DataRow
using Microsoft.Data.SqlClient;       // SqlConnection, SqlCommand (modern)
// OR:
using System.Data.SqlClient;          // same, older package (still works)
```

---

## 🔑 Supported Databases

ADO.NET works with any database — you just swap the provider:

| Database             | Package / Namespace                        |
| -------------------- | ------------------------------------------ |
| **SQL Server** | `Microsoft.Data.SqlClient`← this course |
| MySQL                | `MySql.Data.MySqlClient`                 |
| PostgreSQL           | `Npgsql`                                 |
| Oracle               | `Oracle.ManagedDataAccess.Client`        |
| OLE DB               | `System.Data.OleDb`                      |

---

## 🔑 What ADO.NET Can Do — CRUD in One Picture

```
C R E A T E → SqlCommand + ExecuteNonQuery → INSERT INTO Employees...
R E A D     → SqlCommand + ExecuteReader  → SELECT * FROM Employees...
U P D A T E → SqlCommand + ExecuteNonQuery → UPDATE Employees SET...
D E L E T E → SqlCommand + ExecuteNonQuery → DELETE FROM Employees...
```

---

## 📊 Connected vs Disconnected — Final Table

|                | Connected                | Disconnected                 |
| -------------- | ------------------------ | ---------------------------- |
| Connection     | Stays open while reading | Closes after data is fetched |
| Primary class  | `SqlDataReader`        | `DataSet`/`DataTable`    |
| Bridge class   | `SqlCommand`           | `SqlDataAdapter`           |
| Data editing   | ❌ Read-only             | ✅ Full edit in memory       |
| Memory use     | Low (one row at a time)  | Higher (all rows loaded)     |
| Scalability    | Lower                    | Higher                       |
| Real-time data | ✅ Yes                   | ❌ Snapshot                  |
| Best for       | Login, live reads        | Kendo Grids, reports         |

---

## ❓ Interview Questions

**Q: What is ADO.NET?**

> Microsoft's built-in data access technology for .NET. It provides classes to connect to databases, execute SQL commands, and retrieve results — giving developers full control over database operations using raw SQL.

**Q: What are the two architecture models in ADO.NET?**

> Connected and Disconnected. Connected keeps the database connection open while reading data row-by-row using `SqlDataReader` — best for live, fast reads. Disconnected fetches all data into a `DataSet`/`DataTable`, closes the connection immediately, and lets you work with data in memory — best for grids and reports.

**Q: What is the difference between ADO.NET and Entity Framework?**

> ADO.NET is low-level — you write SQL manually, have full control, and get maximum performance. EF is a high-level ORM — it generates SQL automatically from C# objects, requires less code, but has more overhead. Production apps often use both: EF for standard CRUD, ADO.NET for complex stored procedures and performance-critical operations.

**Q: Where does ADO.NET code go in a three-layer architecture?**

> In the DAL (Data Access Layer). Never in the Controller or BAL — those layers should never know SQL Server exists. The DAL is the only layer that opens connections and executes commands.
>
