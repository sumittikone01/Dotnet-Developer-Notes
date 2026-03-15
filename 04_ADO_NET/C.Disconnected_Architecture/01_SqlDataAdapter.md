
# 01 — SqlDataAdapter

---

## 🎯 One-Line Definition

> **`SqlDataAdapter` is the bridge between SQL Server and your in-memory DataSet/DataTable — it fetches data from the database into memory, manages the connection automatically, and syncs changes back when you're ready.**

---

## 🔷 What is SqlDataAdapter?

`SqlDataAdapter` is the key class in Disconnected Architecture.
It does two jobs: **fetching data into memory** and  **saving changes back to the database** .

> 💡 Think of `SqlDataAdapter` as a **water truck** — it drives to the water source (SQL Server), fills its tank (DataSet/DataTable), drives back and delivers the water (closes connection), and when you're ready, it takes the dirty water back to be processed (saves changes).

---

## 🔷 How It Fits in the Architecture

```
SQL SERVER                SqlDataAdapter              YOUR CODE
─────────────             ──────────────              ──────────────
                          ┌──────────────────┐
                          │ SelectCommand    │──→ Fill()  → DataTable (memory)
Database ←────────────────│ InsertCommand    │←── Update()← DataTable (memory)
                          │ UpdateCommand    │
                          │ DeleteCommand    │
                          └──────────────────┘
                          Manages connection automatically
                          (opens on Fill, closes after)
```

---

## 🔷 SqlDataAdapter vs SqlDataReader

|                | `SqlDataReader`       | `SqlDataAdapter`                     |
| -------------- | ----------------------- | -------------------------------------- |
| Architecture   | Connected               | Disconnected                           |
| Connection     | Open while reading      | Opens → fills → closes automatically |
| Data stored in | Nothing — streams rows | DataTable / DataSet in memory          |
| Direction      | Read-only, forward-only | Read + Write (edit + save back)        |
| Editing        | ❌ Cannot edit          | ✅ Full edit in memory                 |
| Best for       | Fast live reads         | Forms, grids, offline editing          |

---

## 🔷 Key Properties

These four properties are the four SQL commands the adapter uses to talk to the database:

| Property          | What It Does                                                |
| ----------------- | ----------------------------------------------------------- |
| `SelectCommand` | SQL to**fetch**data → used by `Fill()`             |
| `InsertCommand` | SQL to**insert**new rows → used by `Update()`      |
| `UpdateCommand` | SQL to**update**modified rows → used by `Update()` |
| `DeleteCommand` | SQL to**delete**removed rows → used by `Update()`  |

> 📌 For read-only use (just loading data), you only need `SelectCommand`. The others are only needed when you want to save changes back.

---

## 🔷 Key Methods

| Method                                       | What It Does                                                         |
| -------------------------------------------- | -------------------------------------------------------------------- |
| `Fill(DataTable)`                          | Executes SelectCommand → fetches rows → populates DataTable        |
| `Fill(DataSet, "TableName")`               | Same but fills a named table inside a DataSet                        |
| `Update(DataTable)`                        | Scans DataTable for changes → runs Insert/Update/Delete → syncs DB |
| `FillSchema(DataTable, SchemaType.Source)` | Fetches structure only (column names, types, PK) — no data          |

---

## 🔷 Constructors

```csharp
// 1. Empty — set properties manually after
SqlDataAdapter da = new SqlDataAdapter();
da.SelectCommand = new SqlCommand("SELECT * FROM Employee", con);

// 2. With SqlCommand object
SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con);
SqlDataAdapter da = new SqlDataAdapter(cmd);

// 3. ✅ Most common — query + connection string
SqlDataAdapter da = new SqlDataAdapter(
    "SELECT * FROM Employee",
    "Server=.;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;"
);
// No need to create SqlConnection separately — adapter creates it internally
```

---

## 🔷 Fill() — Load Data into Memory

```csharp
string cs  = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";
string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

// ── Fill into DataTable ───────────────────────────────────────────
DataTable dt = new DataTable();

using (SqlDataAdapter da = new SqlDataAdapter(sql, cs))
{
    da.Fill(dt);
    // Fill() automatically:
    //   1. Opens the connection
    //   2. Executes the SELECT
    //   3. Populates dt with all rows and columns
    //   4. Closes the connection
}
// Connection is CLOSED here — dt has all the data in memory

foreach (DataRow row in dt.Rows)
{
    Console.WriteLine($"{row["Id"]}  {row["Name"]}  {row["Salary"]}");
}
```

---

## 🔷 Fill() — Load into DataSet

```csharp
DataSet ds = new DataSet();

using (SqlDataAdapter da = new SqlDataAdapter(sql, cs))
{
    da.Fill(ds, "Employees");   // ← name the table inside the DataSet
}

// Access the table by name
DataTable empTable = ds.Tables["Employees"];
Console.WriteLine($"Total rows: {empTable.Rows.Count}");

foreach (DataRow row in ds.Tables["Employees"].Rows)
{
    Console.WriteLine(row["Name"]);
}
```

---

## 🔷 Load Multiple Tables in One DataSet

```csharp
DataSet ds = new DataSet();

// Load Employees
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(ds, "Employees");

// Load Departments
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Department", cs))
    da.Fill(ds, "Departments");

// Now ds has two in-memory tables
Console.WriteLine($"Employees:   {ds.Tables["Employees"].Rows.Count}");
Console.WriteLine($"Departments: {ds.Tables["Departments"].Rows.Count}");
```

---

## 🔷 Update() — Save Changes Back to Database

After editing the DataTable in memory, use `Update()` to send changes back.
You must set `InsertCommand`, `UpdateCommand`, and `DeleteCommand` — or use `SqlCommandBuilder` to generate them automatically.

```csharp
// SqlCommandBuilder auto-generates Insert/Update/Delete commands from SelectCommand
DataTable dt = new DataTable();

using (SqlConnection con = new SqlConnection(cs))
{
    SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", con);
    SqlCommandBuilder builder = new SqlCommandBuilder(da);
    // builder automatically generates InsertCommand, UpdateCommand, DeleteCommand

    da.Fill(dt);   // load data

    // ── Make changes offline ──────────────────────────────────────
    dt.Rows[0]["Salary"] = 90000;           // modify existing row
    dt.Rows.Add(3, "Carol", "Analyst", 60000);  // add new row
    dt.Rows[1].Delete();                    // mark row for deletion

    // ── Save all changes back to database ────────────────────────
    da.Update(dt);   // sends only changed rows — not the entire table
}
```

---

## 🔷 MVC Repository Pattern — Real Usage

```csharp
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // Load all employees into a DataTable
    public DataTable GetAll()
    {
        DataTable dt = new DataTable();
        string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

        using (SqlDataAdapter da = new SqlDataAdapter(sql, _cs))
        {
            da.Fill(dt);
        }

        return dt;   // caller works with in-memory DataTable
    }

    // Load by department
    public DataTable GetByDepartment(string dept)
    {
        DataTable dt = new DataTable();
        string sql = "SELECT * FROM Employee WHERE Department = @Dept";

        using (SqlConnection con = new SqlConnection(_cs))
        {
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

| Mistake                                                             | What Happens                              | Fix                                                   |
| ------------------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------------- |
| Calling `Fill()`while connection is already open                  | Exception — adapter tries to open again  | Let adapter manage connection — don't open manually  |
| `Update()`without setting commands or using `SqlCommandBuilder` | Exception — no InsertCommand etc.        | Add `SqlCommandBuilder(da)`or set commands manually |
| Not using `using`for `SqlDataAdapter`                           | Minor resource leak                       | Always wrap in `using`                              |
| Expecting `Fill()`to keep connection open                         | Confusion — connection closed after Fill | DataTable works fully offline after Fill              |

---

## ⭐ Interview Quick-Fire

| Question                                                  | Answer                                                                   |
| --------------------------------------------------------- | ------------------------------------------------------------------------ |
| What is `SqlDataAdapter`?                               | Bridge between SQL Server and in-memory DataSet/DataTable                |
| Does `Fill()`need a manually opened connection?         | ❌ No — adapter opens and closes it automatically                       |
| What does `Update()`do?                                 | Scans DataTable for changes and syncs them to the database               |
| What is `SqlCommandBuilder`?                            | Auto-generates Insert/Update/Delete commands from SelectCommand          |
| Difference:`Fill(DataTable)`vs `Fill(DataSet, name)`? | One fills a single table, the other fills a named table inside a DataSet |
| What is `FillSchema()`for?                              | Fetches column structure and keys — no data rows                        |
| Which architecture uses SqlDataAdapter?                   | Disconnected Architecture                                                |
