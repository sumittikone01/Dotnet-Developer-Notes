
# 03 — SqlCommand

---

## 🎯 One-Line Definition

> **`SqlCommand` sends your SQL instructions to the database — choose `ExecuteReader()` for SELECT, `ExecuteNonQuery()` for INSERT/UPDATE/DELETE, or `ExecuteScalar()` for a single value like COUNT.**

---

## 🔷 What is SqlCommand?

`SqlCommand` is the class that  **sends SQL instructions to SQL Server** .
Once you have an open `SqlConnection`, `SqlCommand` tells SQL Server  *what to do* .

> 💡 If `SqlConnection` is the **door** to the database, `SqlCommand` is the **words you speak** after walking through.

---

## 🔷 Basic Syntax

```csharp
using (SqlCommand cmd = new SqlCommand("SQL query here", con))
{
    // execute here
}
```

---

## 🔷 Constructors

| Constructor                                                      | When to Use                                         |
| ---------------------------------------------------------------- | --------------------------------------------------- |
| `SqlCommand()`                                                 | Empty — set properties manually later              |
| `SqlCommand(string sql)`                                       | Query only — assign connection separately          |
| `SqlCommand(string sql, SqlConnection con)`                    | ✅**Most used**— query + connection together |
| `SqlCommand(string sql, SqlConnection con, SqlTransaction tx)` | Inside a transaction                                |

```csharp
// ✅ Most common form:
using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
{
    // ...
}

// Empty constructor alternative:
SqlCommand cmd = new SqlCommand();
cmd.CommandText = "SELECT * FROM Employee";
cmd.Connection  = con;
```

---

## 🔷 Important Properties

| Property           | Type                       | Purpose                                |
| ------------------ | -------------------------- | -------------------------------------- |
| `CommandText`    | `string`                 | The SQL query or stored procedure name |
| `CommandType`    | `CommandType`enum        | `Text`or `StoredProcedure`         |
| `Connection`     | `SqlConnection`          | The open connection to use             |
| `CommandTimeout` | `int`                    | Max seconds to wait (default: 30)      |
| `Parameters`     | `SqlParameterCollection` | Safe parameterized values              |
| `Transaction`    | `SqlTransaction`         | Associated transaction (optional)      |

---

## 🔷 The Three Execute Methods

```
What do you need back from SQL Server?
│
├── Multiple rows (SELECT)          →  ExecuteReader()    → SqlDataReader
│
├── Rows affected (INSERT/UPDATE/DELETE) → ExecuteNonQuery() → int
│
└── Single value (COUNT/MAX/MIN/SUM)   → ExecuteScalar()   → object (cast it)
```

---

### ExecuteReader() — SELECT multiple rows

```csharp
using (SqlCommand cmd = new SqlCommand("SELECT Id, Name, Salary FROM Employee", con))
{
    using (SqlDataReader reader = cmd.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"{reader["Id"]}  {reader["Name"]}  {reader["Salary"]}");
        }
    }
}
```

---

### ExecuteNonQuery() — INSERT / UPDATE / DELETE

```csharp
string sql = "INSERT INTO Employee(Name, Salary) VALUES(@Name, @Salary)";

using (SqlCommand cmd = new SqlCommand(sql, con))
{
    cmd.Parameters.AddWithValue("@Name",   "Sumit");
    cmd.Parameters.AddWithValue("@Salary", 65000);

    int rowsAffected = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rowsAffected} row(s) inserted.");
}
```

---

### ExecuteScalar() — Single value

```csharp
using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
{
    int count = (int)cmd.ExecuteScalar();
    Console.WriteLine($"Total employees: {count}");
}
```

---

## 🔷 Execute Methods — Quick Comparison

| Method                | Returns                 | Use For                           |
| --------------------- | ----------------------- | --------------------------------- |
| `ExecuteReader()`   | `SqlDataReader`       | SELECT — multiple rows           |
| `ExecuteNonQuery()` | `int`(rows affected)  | INSERT, UPDATE, DELETE            |
| `ExecuteScalar()`   | `object`(cast needed) | COUNT, MAX, MIN, SUM — one value |

---

## 🔷 CommandType — Text vs StoredProcedure

`CommandType` tells ADO.NET whether `CommandText` is raw SQL or a stored procedure name.

### CommandType.Text — Inline SQL (Default)

```csharp
string query = "SELECT * FROM Employee WHERE Id = @Id";

using (SqlCommand cmd = new SqlCommand(query, con))
{
    cmd.CommandType = CommandType.Text;   // default — can be omitted
    cmd.Parameters.AddWithValue("@Id", 5);
    // ...
}
```

### CommandType.StoredProcedure — Calling a Stored Procedure

```csharp
using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
{
    cmd.CommandType = CommandType.StoredProcedure;   // ← MUST set this
    cmd.Parameters.AddWithValue("@Id", 5);
    // ...
}
```

### Text vs StoredProcedure — Comparison

|                         | `CommandType.Text`   | `CommandType.StoredProcedure`   |
| ----------------------- | ---------------------- | --------------------------------- |
| `CommandText`contains | Full SQL query         | Just the SP name                  |
| SQL lives in            | C# code file           | SQL Server database               |
| Performance             | Good                   | Slightly better (SP pre-compiled) |
| Use for                 | Quick queries          | Complex business logic            |
| Must set CommandType?   | No — it's the default | ✅ Yes — mandatory               |

---

## 🔷 Complete CRUD Example

```csharp
using System;
using System.Data;
using Microsoft.Data.SqlClient;

public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // ── INSERT ────────────────────────────────────────────────────────
    public void InsertEmployee(string name, string role, decimal salary)
    {
        string sql = "INSERT INTO Employee(Name, Role, Salary) VALUES(@Name, @Role, @Salary)";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Name",   name);
                cmd.Parameters.AddWithValue("@Role",   role);
                cmd.Parameters.AddWithValue("@Salary", salary);

                int rows = cmd.ExecuteNonQuery();
                Console.WriteLine($"✅ Inserted — {rows} row(s) affected.");
            }
        }
    }

    // ── UPDATE ────────────────────────────────────────────────────────
    public void UpdateSalary(int id, decimal newSalary)
    {
        string sql = "UPDATE Employee SET Salary = @Salary WHERE Id = @Id";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Salary", newSalary);
                cmd.Parameters.AddWithValue("@Id",     id);

                int rows = cmd.ExecuteNonQuery();
                Console.WriteLine($"✅ Updated — {rows} row(s) affected.");
            }
        }
    }

    // ── DELETE ────────────────────────────────────────────────────────
    public void DeleteEmployee(int id)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand("DELETE FROM Employee WHERE Id = @Id", con))
            {
                cmd.Parameters.AddWithValue("@Id", id);

                int rows = cmd.ExecuteNonQuery();
                Console.WriteLine($"🗑️ Deleted — {rows} row(s) affected.");
            }
        }
    }

    // ── COUNT (ExecuteScalar) ─────────────────────────────────────────
    public int GetCount()
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
            {
                return (int)cmd.ExecuteScalar();
            }
        }
    }
}
```

---

## 🔷 SqlCommand with Stored Procedure

```sql
-- First create this SP in SQL Server:
CREATE PROCEDURE sp_GetEmployeeById
    @Id INT
AS
BEGIN
    SELECT Id, Name, Role, Salary FROM Employee WHERE Id = @Id
END
```

```csharp
public void GetEmployeeBySP(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;   // ← required

            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                if (reader.Read())
                {
                    Console.WriteLine($"{reader["Name"]} — {reader["Salary"]}");
                }
            }
        }
    }
}
```

---

## 🔷 SqlCommand with Transaction

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    SqlTransaction tx = con.BeginTransaction();

    using (SqlCommand cmd = new SqlCommand(
        "INSERT INTO Employee(Name, Salary) VALUES(@Name, @Salary)", con, tx))
    {
        try
        {
            cmd.Parameters.AddWithValue("@Name",   "Amit");
            cmd.Parameters.AddWithValue("@Salary", 55000);
            cmd.ExecuteNonQuery();

            tx.Commit();
            Console.WriteLine("✅ Committed.");
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"❌ Rolled back: {ex.Message}");
        }
    }
}
```

---

## 🔷 Best Practices

* ✅ Always use `using` block for SqlCommand
* ✅ Always use parameterized queries (`@param`) — never string concatenation
* ✅ Open connection **just before** executing — not long before
* ✅ Set `CommandType = StoredProcedure` when calling SPs — not optional
* ✅ Use `ExecuteScalar()` for single values — not `ExecuteReader()`

---

## ⭐ Interview Quick-Fire

| Question                           | Answer                                                  |
| ---------------------------------- | ------------------------------------------------------- |
| What does SqlCommand do?           | Sends SQL to the database for execution                 |
| Three execute methods?             | `ExecuteReader`,`ExecuteNonQuery`,`ExecuteScalar` |
| Which for SELECT multiple rows?    | `ExecuteReader()`→ returns `SqlDataReader`         |
| Which for INSERT/UPDATE/DELETE?    | `ExecuteNonQuery()`→ returns `int`(rows affected)  |
| Which for COUNT or single value?   | `ExecuteScalar()`→ returns `object`(cast needed)   |
| Default CommandType?               | `CommandType.Text`                                    |
| Must you set CommandType for SPs?  | ✅ Yes — mandatory                                     |
| What does CommandText hold for SP? | Just the stored procedure name — no SQL                |
| Why use parameters (@param)?       | Prevents SQL injection attacks                          |
