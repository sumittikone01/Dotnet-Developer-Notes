
# 06 — Fill() and Update()

---

## 🎯 One-Line Definition

> **`Fill()` loads data FROM the database INTO your DataTable/DataSet. `Update()` pushes your offline changes BACK to the database. Together they are the two-way sync mechanism of Disconnected Architecture.**

---

## 🔷 The Complete Disconnected Cycle

```
DATABASE                  SqlDataAdapter              YOUR CODE (RAM)
─────────                 ──────────────              ───────────────

SQL Server
   │
   │    ◄─── Fill() ───────────────────────────────── DataTable (empty)
   │    sends rows       Opens conn → Executes SELECT
   │                     → Populates DataTable
   │                     → Closes conn automatically
   │
   │    ◄── Update() ───────────────────────────────── DataTable (changed)
   │    receives changes Opens conn → Executes INSERT/UPDATE/DELETE
   │                     → Syncs only changed rows
   │                     → Closes conn automatically
   │
Done
```

---

# PART A — Fill()

---

## 🔷 What does Fill() do?

`Fill()` is the method that  **loads data from SQL Server into a DataTable or DataSet** .

It does  **four things automatically** :

1. Opens the connection (if not already open)
2. Executes the SELECT command
3. Populates the DataTable with rows and column schema
4. Closes the connection

> 💡 Think of `Fill()` as **downloading** — it pulls a snapshot of the database into your memory.

---

## 🔷 Fill() Signatures

```csharp
da.Fill(DataTable dt)                       // fills a single DataTable
da.Fill(DataSet ds, string tableName)       // fills a named table inside DataSet
da.Fill(DataSet ds, int startRow, int maxRows, string tableName)  // paged fill
da.Fill(DataTable dt, IDataReader reader)   // fills from existing reader
```

---

## 🔷 Fill() — Into DataTable

```csharp
string cs  = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";
string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

DataTable dt = new DataTable();

using (SqlDataAdapter da = new SqlDataAdapter(sql, cs))
{
    da.Fill(dt);
    // ↑ Connection opened, SELECT executed, DataTable filled, connection closed
}

// Connection is CLOSED — dt lives in RAM
Console.WriteLine($"Rows loaded: {dt.Rows.Count}");
Console.WriteLine($"Columns:     {dt.Columns.Count}");

foreach (DataRow row in dt.Rows)
{
    Console.WriteLine($"{row["Id"]}  {row["Name"]}  {row["Salary"]}");
}
```

---

## 🔷 Fill() — Into DataSet

```csharp
DataSet ds = new DataSet();

using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(ds, "Employees");   // ← table gets this name in the DataSet

// Access by the name you gave it
DataTable dt = ds.Tables["Employees"];
Console.WriteLine($"Employees: {dt.Rows.Count}");
```

---

## 🔷 Fill() — With Parameters (Filtered Data)

```csharp
// Fill only IT department employees
using (SqlConnection con = new SqlConnection(cs))
{
    string sql = "SELECT Id, Name, Salary FROM Employee WHERE Department = @Dept";

    using (SqlCommand cmd = new SqlCommand(sql, con))
    {
        cmd.Parameters.AddWithValue("@Dept", "IT");

        DataTable dt = new DataTable();
        using (SqlDataAdapter da = new SqlDataAdapter(cmd))
        {
            da.Fill(dt);
        }

        Console.WriteLine($"IT employees: {dt.Rows.Count}");
    }
}
```

---

## 🔷 Fill() — Paged Load (Large Datasets)

Load a specific range of rows — useful for custom paging:

```csharp
DataTable dt = new DataTable();

using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee ORDER BY Id", cs))
{
    //             startRow  maxRows  tableName
    da.Fill(dt,    10,       10,      "Employees");
    //       ↑ skip first 10 rows, take next 10
    // Simulates: page 2 with page size 10
}

Console.WriteLine($"Rows in page: {dt.Rows.Count}");
```

---

## 🔷 What Fill() Does to Existing Data

```csharp
DataTable dt = new DataTable();

// First fill
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(dt);
Console.WriteLine($"After fill 1: {dt.Rows.Count} rows");   // e.g. 5

// Add a row manually to the table
dt.Rows.Add(99, "Test", "Role", 0);

// Second fill — does NOT clear existing rows
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(dt);
Console.WriteLine($"After fill 2: {dt.Rows.Count} rows");   // 5 + 1 + 5 = could be 11!

// ✅ To refresh cleanly — clear first
dt.Clear();   // removes all rows
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(dt);
Console.WriteLine($"After clear + fill: {dt.Rows.Count} rows");   // 5 — clean!
```

> ⚠️ `Fill()` **appends** rows to existing data — it does NOT replace. Always call `dt.Clear()` before re-filling if you want fresh data.

---

# PART B — Update()

---

## 🔷 What does Update() do?

`Update()` scans the DataTable for changed rows and sends the appropriate SQL command for each:

```
For each row in DataTable where RowState ≠ Unchanged:
  RowState = Added    →  execute InsertCommand
  RowState = Modified →  execute UpdateCommand
  RowState = Deleted  →  execute DeleteCommand
```

> 💡 Think of `Update()` as **uploading** — it pushes only your changes back to the database, not the entire table.

---

## 🔷 The Problem — You Need Commands for Update()

`Update()` needs `InsertCommand`, `UpdateCommand`, and `DeleteCommand` to work.
You have two ways to set them:

```
Way 1: SqlCommandBuilder — auto-generates all three (quick, for simple tables)
Way 2: Manual — you write each SQL command (for complex scenarios)
```

---

## 🔷 Update() — with SqlCommandBuilder (Quick Way)

```csharp
DataTable dt = new DataTable();

using (SqlConnection con = new SqlConnection(cs))
{
    SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", con);

    // ── Auto-generate Insert/Update/Delete commands ───────────────
    SqlCommandBuilder builder = new SqlCommandBuilder(da);
    // builder reads the SelectCommand and generates the three write commands

    // ── Fill ──────────────────────────────────────────────────────
    da.Fill(dt);

    // ── Make changes offline ──────────────────────────────────────
    // Modify a row
    DataRow emp = dt.Rows.Find(1);
    if (emp != null)
        emp["Salary"] = 85000m;

    // Add a new row
    dt.Rows.Add(0, "Dave", "Designer", 62000m);

    // Delete a row
    DataRow toDelete = dt.Rows.Find(2);
    if (toDelete != null)
        toDelete.Delete();

    // ── Sync all changes back to database ─────────────────────────
    int rowsAffected = da.Update(dt);
    Console.WriteLine($"✅ {rowsAffected} row(s) updated in database.");

    // Mark all rows as Unchanged
    dt.AcceptChanges();
}
```

---

## 🔷 Update() — Manual Commands (Full Control)

For precise control over every SQL statement:

```csharp
using (SqlConnection con = new SqlConnection(cs))
{
    SqlDataAdapter da = new SqlDataAdapter();

    // ── SelectCommand ─────────────────────────────────────────────
    da.SelectCommand = new SqlCommand(
        "SELECT Id, Name, Role, Salary FROM Employee", con);

    // ── InsertCommand ─────────────────────────────────────────────
    da.InsertCommand = new SqlCommand(
        "INSERT INTO Employee (Name, Role, Salary) VALUES (@Name, @Role, @Salary)", con);
    da.InsertCommand.Parameters.Add("@Name",   SqlDbType.NVarChar, 100, "Name");
    da.InsertCommand.Parameters.Add("@Role",   SqlDbType.NVarChar, 50,  "Role");
    da.InsertCommand.Parameters.Add("@Salary", SqlDbType.Decimal,  0,   "Salary");
    // ↑ The 4th argument "Name"/"Role"/"Salary" = DataTable column name to read from

    // ── UpdateCommand ─────────────────────────────────────────────
    da.UpdateCommand = new SqlCommand(
        "UPDATE Employee SET Name=@Name, Role=@Role, Salary=@Salary WHERE Id=@Id", con);
    da.UpdateCommand.Parameters.Add("@Name",   SqlDbType.NVarChar, 100, "Name");
    da.UpdateCommand.Parameters.Add("@Role",   SqlDbType.NVarChar, 50,  "Role");
    da.UpdateCommand.Parameters.Add("@Salary", SqlDbType.Decimal,  0,   "Salary");
    SqlParameter idParam = da.UpdateCommand.Parameters.Add("@Id", SqlDbType.Int, 0, "Id");
    idParam.SourceVersion = DataRowVersion.Original;
    // ↑ SourceVersion.Original = use the ORIGINAL Id value (before any edits)

    // ── DeleteCommand ─────────────────────────────────────────────
    da.DeleteCommand = new SqlCommand(
        "DELETE FROM Employee WHERE Id = @Id", con);
    da.DeleteCommand.Parameters.Add("@Id", SqlDbType.Int, 0, "Id")
        .SourceVersion = DataRowVersion.Original;

    // ── Fill and modify ───────────────────────────────────────────
    DataTable dt = new DataTable();
    da.Fill(dt);

    dt.Rows[0]["Salary"] = 99000m;   // modify
    dt.Rows.Add(0, "Eve", "PM", 70000m);   // add

    // ── Save back ─────────────────────────────────────────────────
    da.Update(dt);
    dt.AcceptChanges();
}
```

---

## 🔷 Update Only Changed Rows — GetChanges()

For large DataTables, send only the changed rows for efficiency:

```csharp
DataTable dt = new DataTable();

using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
{
    SqlCommandBuilder cb = new SqlCommandBuilder(da);
    da.Fill(dt);

    // Make some changes...
    dt.Rows[0]["Salary"] = 90000m;
    dt.Rows.Add(0, "Frank", "Analyst", 65000m);

    // Get only the changed rows
    DataTable changes = dt.GetChanges();

    if (changes != null && changes.Rows.Count > 0)
    {
        da.Update(changes);    // ← send only 2 rows, not the full table
        dt.AcceptChanges();
        Console.WriteLine($"✅ Saved {changes.Rows.Count} change(s).");
    }
    else
    {
        Console.WriteLine("No changes to save.");
    }
}
```

---

## 🔷 Fill vs Update — Side by Side

|               | `Fill()`                     | `Update()`                                          |
| ------------- | ------------------------------ | ----------------------------------------------------- |
| Direction     | Database → DataTable          | DataTable → Database                                 |
| What it does  | Loads all rows matching SELECT | Sends only changed rows                               |
| Connection    | Auto-opens, auto-closes        | Auto-opens, auto-closes                               |
| Requires      | `SelectCommand`              | `InsertCommand`+`UpdateCommand`+`DeleteCommand` |
| Easy setup    | ✅ Just a SELECT               | Needs `SqlCommandBuilder`or manual commands         |
| Rows affected | All SELECT rows                | Only Added/Modified/Deleted rows                      |
| After call    | DataTable has data             | Database updated, call `AcceptChanges()`            |

---

## 🔷 Complete MVC Repository Pattern

```csharp
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // ── Get all employees into DataTable ─────────────────────────
    public DataTable GetAll()
    {
        DataTable dt = new DataTable();
        using (SqlDataAdapter da = new SqlDataAdapter(
            "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name", _cs))
        {
            da.Fill(dt);
        }
        return dt;
    }

    // ── Save all changes from DataTable back to DB ────────────────
    public int SaveChanges(DataTable dt)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            SqlDataAdapter da = new SqlDataAdapter(
                "SELECT Id, Name, Role, Salary FROM Employee", con);
            SqlCommandBuilder cb = new SqlCommandBuilder(da);

            int affected = da.Update(dt);
            dt.AcceptChanges();

            return affected;
        }
    }

    // ── Get filtered data ─────────────────────────────────────────
    public DataTable GetByDepartment(string dept)
    {
        DataTable dt = new DataTable();

        using (SqlConnection con = new SqlConnection(_cs))
        {
            string sql = "SELECT Id, Name, Salary FROM Employee WHERE Department = @Dept";
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Dept", dept);
                using (SqlDataAdapter da = new SqlDataAdapter(cmd))
                {
                    da.Fill(dt);
                }
            }
        }

        return dt;
    }
}
```

---

## ⚠️ Common Mistakes

| Mistake                                              | What Happens                                                  | Fix                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------- |
| Calling `Fill()`twice without `Clear()`first     | Rows duplicated in DataTable                                  | Call `dt.Clear()`before re-filling                    |
| `Update()`without `SqlCommandBuilder`or commands | Exception — no Insert/Update/Delete commands                 | Add `new SqlCommandBuilder(da)`                       |
| Not calling `AcceptChanges()`after `Update()`    | Rows still marked as Modified/Added — Update would run again | Always call `dt.AcceptChanges()`after `da.Update()` |
| `Update()`on large DataTable every time            | Performance issue — sends all rows                           | Use `dt.GetChanges()`and update only the changes      |
| SqlCommandBuilder needs PK in SELECT                 | Exception — builder can't generate commands without PK       | Include the primary key column in your SELECT           |

---

## ⭐ Interview Quick-Fire

| Question                                             | Answer                                                                        |
| ---------------------------------------------------- | ----------------------------------------------------------------------------- |
| What does `Fill()`do?                              | Loads data from DB into DataTable/DataSet — auto-opens and closes connection |
| What does `Update()`do?                            | Sends changed rows (Added/Modified/Deleted) back to the database              |
| Does `Fill()`auto-open the connection?             | ✅ Yes — and auto-closes after filling                                       |
| Does `Fill()`replace existing data?                | ❌ No — it appends. Call `Clear()`first to refresh                         |
| What is `SqlCommandBuilder`?                       | Auto-generates Insert/Update/Delete commands from SelectCommand               |
| Must you call `AcceptChanges()`after `Update()`? | ✅ Yes — marks rows as Unchanged so they won't be sent again                 |
| What is `GetChanges()`for?                         | Returns a DataTable with only modified rows — more efficient Update          |
| `SqlCommandBuilder`requirement?                    | SELECT must include the Primary Key column                                    |
