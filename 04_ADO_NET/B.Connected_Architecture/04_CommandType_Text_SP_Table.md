
# 04 — CommandType: Text, StoredProcedure, TableDirect

---

## 🎯 One-Line Definition

> **`CommandType` is an enum that tells `SqlCommand` what `CommandText` contains — raw SQL (`Text`), a stored procedure name (`StoredProcedure`), or a table name (`TableDirect`).**

---

## 🔷 What is CommandType?

Every `SqlCommand` has a `CommandType` property.
It answers one question: **"What kind of thing am I sending to SQL Server?"**

```csharp
cmd.CommandType = CommandType.Text;             // sending SQL query
cmd.CommandType = CommandType.StoredProcedure;  // sending SP name
cmd.CommandType = CommandType.TableDirect;      // sending table name
```

> 📌 `CommandType` comes from the `System.Data` namespace (not SqlClient).

---

## 🔷 The Three Values

```
CommandType Enum
│
├── Text            → CommandText = full SQL query string
│                     "SELECT * FROM Employee WHERE Id = @Id"
│
├── StoredProcedure → CommandText = stored procedure name only
│                     "sp_GetEmployeeById"
│
└── TableDirect     → CommandText = table name only
                      "Employee"
                      (OleDb only — not supported in SQL Server)
```

---

## 🔷 CommandType.Text — Inline SQL (Default)

`Text` is the **default** value — you can omit setting it, but being explicit is clearer.

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    string sql = "SELECT Id, Name, Role, Salary FROM Employee WHERE Salary > @Salary";

    using (SqlCommand cmd = new SqlCommand(sql, con))
    {
        cmd.CommandType = CommandType.Text;  // default — can be omitted

        cmd.Parameters.AddWithValue("@Salary", 50000);

        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine($"{reader["Name"]} — {reader["Salary"]}");
            }
        }
    }
}
```

### When to Use Text

```
✅ Writing SQL directly in C# code
✅ Dynamic queries (filters change at runtime)
✅ Simple one-off queries
✅ Learning / prototyping
❌ Complex business logic (hard to maintain in C# strings)
❌ Repeated heavy queries (no pre-compilation)
```

---

## 🔷 CommandType.StoredProcedure — Calling a Stored Procedure

`CommandText` contains **only the procedure name** — no SQL keywords.
You **must** set `CommandType = CommandType.StoredProcedure` — it does not default.

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
    {
        cmd.CommandType = CommandType.StoredProcedure;  // ← MANDATORY — must set this
        //                                                 without this, ADO.NET sends
        //                                                 "sp_GetEmployeeById" as SQL
        //                                                 → syntax error!

        cmd.Parameters.AddWithValue("@Id", 5);

        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            if (reader.Read())
            {
                Console.WriteLine($"{reader["Name"]} — {reader["Role"]}");
            }
        }
    }
}
```

### What the SP Looks Like in SQL Server

```sql
CREATE PROCEDURE sp_GetEmployeeById
    @Id INT
AS
BEGIN
    SELECT Id, Name, Role, Salary
    FROM Employee
    WHERE Id = @Id
END
```

### When to Use StoredProcedure

```
✅ Complex SQL logic that belongs in the database
✅ Reusing the same SQL across multiple applications
✅ Performance-critical queries (SP is pre-compiled)
✅ Security — users get execute permission only, not table access
✅ This is the real-world industry standard for production apps
❌ Simple one-line queries (overkill)
```

---

## 🔷 CommandType.TableDirect — Table Name Only

Sends just a table name — ADO.NET generates `SELECT * FROM TableName` automatically.

```csharp
// ⚠️ Only works with OleDb provider — NOT SqlClient (SQL Server)
using (OleDbConnection con = new OleDbConnection(cs))
{
    con.Open();
    using (OleDbCommand cmd = new OleDbCommand("Employee", con))
    {
        cmd.CommandType = CommandType.TableDirect;
        // Equivalent to: SELECT * FROM Employee
        OleDbDataReader reader = cmd.ExecuteReader();
    }
}
```

> ⚠️ `TableDirect` is **not supported** by `SqlCommand` (SQL Server).
> It only works with `OleDbCommand`. You will rarely use this.

---

## 🔷 What Happens If You Forget to Set CommandType for SP?

```csharp
// ❌ WRONG — CommandType not set (defaults to Text)
using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
{
    // CommandType = CommandType.Text  ← default
    // ADO.NET sends: "sp_GetEmployeeById" as a SQL statement
    // SQL Server tries to execute it as SQL → syntax error!

    cmd.Parameters.AddWithValue("@Id", 5);
    cmd.ExecuteReader();   // ← SqlException: "sp_GetEmployeeById" is not valid SQL
}

// ✅ CORRECT
using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
{
    cmd.CommandType = CommandType.StoredProcedure;  // ← tells ADO.NET: this is an SP name
    cmd.Parameters.AddWithValue("@Id", 5);
    cmd.ExecuteReader();   // ✅ works correctly
}
```

---

## 🔷 Text vs StoredProcedure — Full Comparison

|                            | `CommandType.Text`    | `CommandType.StoredProcedure` |
| -------------------------- | ----------------------- | ------------------------------- |
| `CommandText`contains    | Full SQL query          | Just the SP name                |
| Default value?             | ✅ Yes                  | ❌ Must set explicitly          |
| SQL lives in               | C# source file          | SQL Server database             |
| Pre-compiled by SQL Server | ❌ Compiled each call   | ✅ Pre-compiled — faster       |
| Reusable across apps       | ❌ No                   | ✅ Yes                          |
| Security control           | Table-level             | Execute permission only         |
| Parameters                 | `@param`in SQL string | `@param`matching SP params    |
| Best for                   | Quick/dynamic queries   | Business logic, production      |

---

## 🔷 Complete Side-by-Side Example

### Same operation — two ways:

```csharp
// ── Way 1: CommandType.Text ───────────────────────────────────────
using (SqlCommand cmd = new SqlCommand(
    "INSERT INTO Employee(Name, Role, Salary) VALUES(@Name, @Role, @Salary)", con))
{
    cmd.CommandType = CommandType.Text;   // default, can omit

    cmd.Parameters.AddWithValue("@Name",   "Alice");
    cmd.Parameters.AddWithValue("@Role",   "Developer");
    cmd.Parameters.AddWithValue("@Salary", 70000);

    cmd.ExecuteNonQuery();
}


// ── Way 2: CommandType.StoredProcedure ────────────────────────────
// SQL Server has: CREATE PROCEDURE sp_InsertEmployee @Name, @Role, @Salary

using (SqlCommand cmd = new SqlCommand("sp_InsertEmployee", con))
{
    cmd.CommandType = CommandType.StoredProcedure;   // mandatory

    cmd.Parameters.AddWithValue("@Name",   "Alice");
    cmd.Parameters.AddWithValue("@Role",   "Developer");
    cmd.Parameters.AddWithValue("@Salary", 70000);

    cmd.ExecuteNonQuery();
}
```

> Both do **exactly the same thing** — the difference is where the SQL lives and who maintains it.

---

## ⭐ Interview Quick-Fire

| Question                                          | Answer                                               |
| ------------------------------------------------- | ---------------------------------------------------- |
| What is `CommandType`?                          | Enum that tells SqlCommand what CommandText contains |
| Three values of CommandType?                      | `Text`,`StoredProcedure`,`TableDirect`         |
| Default CommandType?                              | `CommandType.Text`                                 |
| What goes in CommandText for `Text`?            | Full SQL query string                                |
| What goes in CommandText for `StoredProcedure`? | Only the stored procedure name                       |
| Must you set CommandType for SP?                  | ✅ Yes — mandatory, no default                      |
| What happens if you forget CommandType for SP?    | SP name treated as SQL → SqlException               |
| Does `TableDirect`work with SqlCommand?         | ❌ No — OleDb only                                  |
| Which CommandType is pre-compiled?                | `StoredProcedure`— faster repeated execution      |
| Where does CommandType come from?                 | `System.Data`namespace                             |
