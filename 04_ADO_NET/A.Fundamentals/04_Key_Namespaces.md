
# 04 — Key Namespaces in ADO.NET

---

## 🎯 One-Line Definition

> **Namespaces are organized folders — before using any ADO.NET class, you must import its namespace at the top of your file, or C# cannot find it.**

---

## 🔷 The Two Lines Every ADO.NET File Needs

```csharp
using System.Data;                  // Core — DataSet, DataTable, DataRow, CommandType
using Microsoft.Data.SqlClient;     // SQL Server — SqlConnection, SqlCommand, SqlDataReader
```

These two lines are the **foundation** of every ADO.NET file you write.

---

## 🔷 Namespace 1 — `System.Data`

This is the **heart of ADO.NET** — the provider-independent core.
Contains classes used in **both** Connected and Disconnected architecture.

| Class / Enum        | What It Does                                         |
| ------------------- | ---------------------------------------------------- |
| `DataSet`         | In-memory mini-database — holds multiple DataTables |
| `DataTable`       | Single table in memory (rows + columns)              |
| `DataRow`         | One record inside a DataTable                        |
| `DataColumn`      | Defines one column (name, type, constraints)         |
| `DataRelation`    | Parent-child relationship between two DataTables     |
| `DataView`        | Filtered / sorted view of a DataTable                |
| `CommandType`     | Enum →`Text`,`StoredProcedure`,`TableDirect`  |
| `ConnectionState` | Enum →`Open`,`Closed`,`Connecting`,`Broken` |
| `IDbConnection`   | Interface — base for all connection classes         |
| `IDbCommand`      | Interface — base for all command classes            |
| `IDataReader`     | Interface — base for all reader classes             |

---

## 🔷 Namespace 2 — `Microsoft.Data.SqlClient` ✅ (Always Use This)

This is the **SQL Server-specific namespace** — the concrete classes that actually talk to SQL Server.

| Class                 | What It Does                                           |
| --------------------- | ------------------------------------------------------ |
| `SqlConnection`     | Opens and manages a connection to SQL Server           |
| `SqlCommand`        | Sends SQL queries or stored procedures to the DB       |
| `SqlDataReader`     | Reads query results row-by-row (Connected model)       |
| `SqlDataAdapter`    | Bridge — fetches data into memory, saves changes back |
| `SqlParameter`      | Safely passes values into SQL (prevents SQL injection) |
| `SqlTransaction`    | Manages transactions — commit / rollback              |
| `SqlCommandBuilder` | Auto-generates INSERT/UPDATE/DELETE commands           |
| `SqlException`      | Exception thrown when SQL Server returns an error      |
| `SqlBulkCopy`       | Efficiently inserts thousands of rows at once          |

---

## 🔷 Legacy Namespace — `System.Data.SqlClient` ❌ (Old — Avoid)

```csharp
using System.Data.SqlClient;    // ❌ OLD — do not use for new projects
```

|                     | `Microsoft.Data.SqlClient`✅ | `System.Data.SqlClient`❌ |
| ------------------- | ------------------------------ | --------------------------- |
| Maintained          | Yes — actively updated        | No — legacy                |
| .NET Core / .NET 6+ | Full support                   | Limited                     |
| Azure AD / MFA auth | ✅ Supported                   | ❌ Not supported            |
| Always Encrypted    | ✅ Supported                   | ❌ Not supported            |
| New projects        | **Use this**             | Avoid                       |

> 📌 Class names are **identical** in both — only the namespace changes. Swapping is one line.

---

## 🔷 Class Hierarchy — How Everything Relates

```
System.Data                         ← provider-agnostic core
├── DataSet
├── DataTable
│   ├── DataRow
│   └── DataColumn
├── DataRelation
├── DataView
├── CommandType  (enum)
├── ConnectionState (enum)
├── IDbConnection  ←─ interface
├── IDbCommand     ←─ interface
└── IDataReader    ←─ interface

Microsoft.Data.SqlClient            ← SQL Server concrete classes
├── SqlConnection    implements IDbConnection
├── SqlCommand       implements IDbCommand
├── SqlDataReader    implements IDataReader
├── SqlDataAdapter
├── SqlParameter
├── SqlTransaction
├── SqlCommandBuilder
├── SqlBulkCopy
└── SqlException
```

---

## 🔷 Which Namespace Does Each Class Come From?

```csharp
// System.Data — provider-independent
DataSet          ds  = new DataSet();
DataTable        dt  = new DataTable();
DataRow          row = dt.NewRow();
CommandType      ct  = CommandType.StoredProcedure;
ConnectionState  cs  = ConnectionState.Open;

// Microsoft.Data.SqlClient — SQL Server specific
SqlConnection    con = new SqlConnection(connStr);
SqlCommand       cmd = new SqlCommand("SELECT...", con);
SqlDataReader    dr  = cmd.ExecuteReader();
SqlDataAdapter   da  = new SqlDataAdapter(cmd);
SqlParameter     p   = new SqlParameter("@Id", 5);
SqlTransaction   tx  = con.BeginTransaction();
```

---

## 🔷 Installing the NuGet Package

`Microsoft.Data.SqlClient` is **not built-in** — you must install it:

```bash
# Package Manager Console (Visual Studio)
Install-Package Microsoft.Data.SqlClient

# .NET CLI
dotnet add package Microsoft.Data.SqlClient
```

After installing, the `using` line works:

```csharp
using Microsoft.Data.SqlClient;   // ✅ now available
```

---

## 🔷 Minimal ADO.NET File Template

```csharp
using System.Data;                  // DataTable, CommandType, ConnectionState
using Microsoft.Data.SqlClient;     // SqlConnection, SqlCommand, SqlDataReader

public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    public void SomeMethod()
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            // ... ADO.NET work here
        }
    }
}
```

---

## ⭐ Interview Quick-Fire

| Question                                 | Answer                            |
| ---------------------------------------- | --------------------------------- |
| Core ADO.NET namespace?                  | `System.Data`                   |
| SQL Server namespace (modern)?           | `Microsoft.Data.SqlClient`      |
| Old SQL Server namespace?                | `System.Data.SqlClient`— avoid |
| Class for connecting to SQL Server?      | `SqlConnection`                 |
| Class for executing SQL?                 | `SqlCommand`                    |
| Class for reading rows (connected)?      | `SqlDataReader`                 |
| Class for in-memory table?               | `DataTable`                     |
| Class for in-memory database?            | `DataSet`                       |
| Class for safe query parameters?         | `SqlParameter`                  |
| Class for auto-generating commands?      | `SqlCommandBuilder`             |
| Class for bulk insert?                   | `SqlBulkCopy`                   |
| Where does `CommandType`come from?     | `System.Data`                   |
| Where does `ConnectionState`come from? | `System.Data`                   |
