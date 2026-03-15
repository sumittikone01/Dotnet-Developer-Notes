
# 01 — Insert with ADO.NET

---

## 🎯 One-Line Definition

> **INSERT adds a new row to the database — use `ExecuteNonQuery()` in Connected Architecture for a direct insert, or add a DataRow and call `adapter.Update()` in Disconnected Architecture.**

---

## 🔷 Two Ways to INSERT

```
┌─────────────────────────────────────────────────────────────────┐
│  WAY 1 — CONNECTED (SqlCommand + ExecuteNonQuery)               │
│  Direct insert → immediate → no in-memory table needed         │
│  Best for: single insert from a form, API, controller          │
├─────────────────────────────────────────────────────────────────┤
│  WAY 2 — DISCONNECTED (SqlDataAdapter + DataSet.Update)         │
│  Add row to DataTable → call adapter.Update() → syncs to DB    │
│  Best for: batch inserts, grids, offline editing               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Way 1 — Connected: SqlCommand + ExecuteNonQuery

### How It Works

```
Open Connection
  ↓
Create SqlCommand with INSERT SQL
  ↓
Add @parameters (prevents SQL injection)
  ↓
ExecuteNonQuery() → sends INSERT to SQL Server → returns rows affected
  ↓
Close Connection (auto via using)
```

### Complete Code

```csharp
using System.Data;
using Microsoft.Data.SqlClient;

public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    public int Insert(string name, string role, string email, int salary)
    {
        string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                       VALUES (@Name, @Role, @Email, @Salary)";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100).Value = name;
                cmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100).Value = role;
                cmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100).Value = email;
                cmd.Parameters.Add("@Salary", SqlDbType.Int).Value          = salary;

                int rows = cmd.ExecuteNonQuery();
                return rows;   // 1 if successful, 0 if failed
            }
        }
    }
}
```

### Insert and Get New ID

```csharp
public int InsertAndGetId(string name, string role, string email, int salary)
{
    // SCOPE_IDENTITY() returns the auto-generated Id
    string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                   VALUES (@Name, @Role, @Email, @Salary);
                   SELECT SCOPE_IDENTITY();";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100).Value = name;
            cmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100).Value = role;
            cmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100).Value = email;
            cmd.Parameters.Add("@Salary", SqlDbType.Int).Value          = salary;

            int newId = Convert.ToInt32(cmd.ExecuteScalar());
            Console.WriteLine($"✅ Inserted. New Id = {newId}");
            return newId;
        }
    }
}
```

---

## 🔷 Way 2 — Disconnected: DataAdapter + DataSet

### How It Works

```
Fill DataSet from DB (loads existing data + column schema)
  ↓
Create new DataRow using dt.NewRow()
  ↓
Set column values on the new row
  ↓
Add row to table:  dt.Rows.Add(newRow)   → RowState = Added
  ↓
Set adapter.InsertCommand with SQL + mapped parameters
  ↓
adapter.Update() → scans for Added rows → executes InsertCommand
  ↓
Connection opens, INSERT fires, connection closes
```

### Complete Code (from your working example)

```csharp
public int InsertRecord(string name, string role, string email, int salary)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        // Step 1: Fill DataSet to get the schema (column structure)
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
        DataSet ds = new DataSet();
        adapter.Fill(ds, "Employee");

        // Step 2: Create a new row using the table's schema
        DataRow newRow = ds.Tables["Employee"].NewRow();
        newRow["Name"]   = name;
        newRow["Role"]   = role;
        newRow["Email"]  = email;
        newRow["Salary"] = salary;

        // Step 3: Add the row → RowState becomes "Added"
        ds.Tables["Employee"].Rows.Add(newRow);

        // Step 4: Define the InsertCommand
        SqlCommand insertCmd = new SqlCommand(
            "INSERT INTO Employee(Name, Role, Email, Salary) VALUES(@Name,@Role,@Email,@Salary)",
            con);

        // Map parameters to DataTable column names (4th argument)
        insertCmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100, "Name");
        insertCmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100, "Role");
        insertCmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100, "Email");
        insertCmd.Parameters.Add("@Salary", SqlDbType.Int,     0,   "Salary");

        adapter.InsertCommand = insertCmd;

        // Step 5: Update() scans for Added rows → fires INSERT for each
        return adapter.Update(ds, "Employee");
    }
}
```

---

## 🔷 Understanding the 4th Parameter in `.Parameters.Add()`

```csharp
insertCmd.Parameters.Add("@Name", SqlDbType.VarChar, 100, "Name");
//                          ↑         ↑               ↑      ↑
//                    param name    SQL type         size   DataTable column name
//
// The 4th argument "Name" tells the adapter:
// "When building the INSERT, read the value from the DataRow's 'Name' column"
// This is called SourceColumn — it maps SQL parameter → DataTable column
```

---

## 🔷 Connected vs Disconnected INSERT — When to Use Which

|                 | Connected                  | Disconnected                |
| --------------- | -------------------------- | --------------------------- |
| Code simplicity | ✅ Simpler                 | More setup                  |
| Best for        | Single inserts, forms      | Batch inserts, grid editing |
| Get new ID      | ✅`SCOPE_IDENTITY()`easy | Harder                      |
| Works offline   | ❌ Needs live connection   | ✅ Fill first, insert later |
| Common in MVC   | ✅ Controllers/repos       | Grids, bulk data            |

---

## ⚠️ Common Mistakes

| Mistake                                                        | What Happens                        | Fix                                                                  |
| -------------------------------------------------------------- | ----------------------------------- | -------------------------------------------------------------------- |
| String concatenation in SQL                                    | SQL Injection risk                  | Always use `@param`parameters                                      |
| Not checking `rows > 0`                                      | Silent failure                      | Always check return value                                            |
| Missing `SourceColumn`in disconnected                        | Adapter sends null values           | 4th argument in `Parameters.Add()`must match DataTable column name |
| Not calling `NewRow()`— directly using `Rows.Add()`values | May work but misses schema defaults | Use `dt.NewRow()`to get schema-aware blank row                     |

---

## ⭐ Interview Quick-Fire

| Question                                              | Answer                                                      |
| ----------------------------------------------------- | ----------------------------------------------------------- |
| Which method executes INSERT in connected mode?       | `ExecuteNonQuery()`                                       |
| What does it return?                                  | `int`— rows affected (1 for success)                     |
| How to get the new auto-generated ID?                 | Add `SELECT SCOPE_IDENTITY()`and use `ExecuteScalar()`  |
| In disconnected mode, which RowState triggers INSERT? | `Added`— set when `dt.Rows.Add(newRow)`is called       |
| What is the 4th argument in `Parameters.Add()`?     | `SourceColumn`— maps the parameter to a DataTable column |
