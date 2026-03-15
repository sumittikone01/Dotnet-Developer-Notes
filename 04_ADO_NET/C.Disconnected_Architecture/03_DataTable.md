
# 03 — DataTable

---

## 🎯 One-Line Definition

> **`DataTable` is an in-memory representation of a single database table — it has columns, rows, and constraints, supports full offline CRUD, and is the core building block of a DataSet.**

---

## 🔷 What is DataTable?

`DataTable` is a single table stored completely in your application's RAM.
It has the same row-and-column structure as a real SQL table, but lives in memory.

> 💡 Think of `DataTable` as a **spreadsheet in RAM** — it has column headers (DataColumns), data rows (DataRows), and rules like primary keys (Constraints) — but it never touches the network.

---

## 🔷 DataTable in the Architecture

```
SQL SERVER                            YOUR APP (RAM)
──────────                            ──────────────
Real Table "Employee"                 DataTable "Employee"
┌────┬───────┬──────────┬────────┐    ┌────┬───────┬──────────┬────────┐
│ Id │ Name  │ Role     │ Salary │    │ Id │ Name  │ Role     │ Salary │
├────┼───────┼──────────┼────────┤    ├────┼───────┼──────────┼────────┤
│  1 │ Alice │ Dev      │ 75000  │    │  1 │ Alice │ Dev      │ 75000  │
│  2 │ Bob   │ QA       │ 55000  │    │  2 │ Bob   │ QA       │ 55000  │
└────┴───────┴──────────┴────────┘    └────┴───────┴──────────┴────────┘

Stored on disk / SQL Server            Stored in RAM — no connection needed
```

---

## 🔷 DataTable Structure

```
DataTable "Employee"
│
├── Columns (DataColumnCollection)
│   ├── DataColumn "Id"     — typeof(int),    PrimaryKey
│   ├── DataColumn "Name"   — typeof(string), not null
│   ├── DataColumn "Role"   — typeof(string)
│   └── DataColumn "Salary" — typeof(decimal)
│
├── Rows (DataRowCollection)
│   ├── DataRow → 1, Alice, Dev, 75000
│   ├── DataRow → 2, Bob,   QA,  55000
│   └── DataRow → 3, Carol, PM,  68000
│
└── Constraints (ConstraintCollection)
    └── PrimaryKeyConstraint on "Id"
```

---

## 🔷 Key Properties

| Property        | Type                     | What It Returns                   |
| --------------- | ------------------------ | --------------------------------- |
| `TableName`   | `string`               | Name of the table                 |
| `Columns`     | `DataColumnCollection` | All column definitions            |
| `Rows`        | `DataRowCollection`    | All data rows                     |
| `Constraints` | `ConstraintCollection` | Primary keys, unique constraints  |
| `PrimaryKey`  | `DataColumn[]`         | Column(s) forming the primary key |
| `HasErrors`   | `bool`                 | `true`if any row has errors     |
| `DataSet`     | `DataSet`              | The parent DataSet (if any)       |

---

## 🔷 Key Methods

| Method                      | What It Does                                                  |
| --------------------------- | ------------------------------------------------------------- |
| `NewRow()`                | Creates a new empty DataRow with the table's column structure |
| `Rows.Add(row)`           | Adds a DataRow to the table                                   |
| `Rows.Add(val1, val2...)` | Adds a row directly by values                                 |
| `Rows.Find(keyValue)`     | Finds a row by primary key value                              |
| `Select(filter)`          | Filters rows — returns `DataRow[]`                         |
| `Select(filter, sort)`    | Filter + sort — returns `DataRow[]`                        |
| `Clear()`                 | Removes all rows — keeps column structure                    |
| `Copy()`                  | Full copy — structure + data                                 |
| `Clone()`                 | Structure only — no rows                                     |
| `AcceptChanges()`         | Marks all rows as Unchanged                                   |
| `RejectChanges()`         | Reverts all changes since last `AcceptChanges()`            |
| `GetChanges()`            | Returns new DataTable with only modified rows                 |
| `Merge(DataTable)`        | Merges another DataTable's data into this one                 |
| `NewRow()`                | Creates blank row matching this table's schema                |

---

## 🔷 Creating a DataTable Manually

```csharp
using System;
using System.Data;

// ── Step 1: Create the table ──────────────────────────────────────
DataTable dt = new DataTable("Employee");

// ── Step 2: Define columns ────────────────────────────────────────
dt.Columns.Add("Id",     typeof(int));
dt.Columns.Add("Name",   typeof(string));
dt.Columns.Add("Role",   typeof(string));
dt.Columns.Add("Salary", typeof(decimal));

// ── Step 3: Set Primary Key ───────────────────────────────────────
dt.PrimaryKey = new DataColumn[] { dt.Columns["Id"] };

// ── Step 4: Add rows using NewRow() ───────────────────────────────
DataRow row1 = dt.NewRow();
row1["Id"]     = 1;
row1["Name"]   = "Alice";
row1["Role"]   = "Developer";
row1["Salary"] = 75000m;
dt.Rows.Add(row1);

// ── Step 5: Add rows using shortcut ───────────────────────────────
dt.Rows.Add(2, "Bob",   "QA Engineer", 55000m);
dt.Rows.Add(3, "Carol", "PM",          68000m);

// ── Step 6: Read all rows ─────────────────────────────────────────
foreach (DataRow row in dt.Rows)
{
    Console.WriteLine($"{row["Id"]}  {row["Name"]}  {row["Salary"]}");
}
```

---

## 🔷 Creating a DataTable from SQL Server

```csharp
string cs  = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";
string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

DataTable dt = new DataTable();

using (SqlDataAdapter da = new SqlDataAdapter(sql, cs))
{
    da.Fill(dt);   // fills structure + data automatically
}

// Now works offline
Console.WriteLine($"Rows:    {dt.Rows.Count}");
Console.WriteLine($"Columns: {dt.Columns.Count}");

foreach (DataRow row in dt.Rows)
{
    Console.WriteLine($"{row["Name"]}  —  {row["Salary"]}");
}
```

---

## 🔷 CRUD on DataTable

### Read — by index and by name

```csharp
// By column name (readable)
string name   = dt.Rows[0]["Name"].ToString();
decimal salary = Convert.ToDecimal(dt.Rows[0]["Salary"]);

// By column index (faster)
int id    = (int)dt.Rows[0][0];
string nm = dt.Rows[0][1].ToString();

// Loop all rows
foreach (DataRow row in dt.Rows)
{
    Console.WriteLine($"{row["Id"]}  {row["Name"]}");
}
```

### Update — modify a row's value

```csharp
// Find by primary key
DataRow found = dt.Rows.Find(1);   // finds row where Id = 1
if (found != null)
{
    found["Salary"] = 85000m;
    found["Role"]   = "Senior Developer";
}
```

### Delete — remove a row

```csharp
// Mark for deletion (soft delete — row still exists until AcceptChanges)
DataRow toDelete = dt.Rows.Find(2);
if (toDelete != null)
{
    toDelete.Delete();   // marks RowState = Deleted
}

// Hard delete — remove from collection immediately
dt.Rows.RemoveAt(0);    // remove by index
```

### AcceptChanges and RejectChanges

```csharp
dt.Rows[0]["Salary"] = 99999m;   // modify

// Check state
Console.WriteLine(dt.Rows[0].RowState);   // Modified

// Commit the change
dt.AcceptChanges();
Console.WriteLine(dt.Rows[0].RowState);   // Unchanged

// OR — undo it
dt.RejectChanges();
Console.WriteLine(dt.Rows[0]["Salary"]);  // original value restored
```

---

## 🔷 Select() — Filter and Sort Rows

```csharp
// ── Filter: salary > 60000 ────────────────────────────────────────
DataRow[] highEarners = dt.Select("Salary > 60000");

foreach (DataRow row in highEarners)
    Console.WriteLine(row["Name"]);

// ── Filter + Sort: IT dept, sorted by salary descending ───────────
DataRow[] itStaff = dt.Select("Role = 'Developer'", "Salary DESC");

// ── Find by primary key ───────────────────────────────────────────
DataRow emp = dt.Rows.Find(3);   // finds where PrimaryKey = 3
if (emp != null)
    Console.WriteLine($"Found: {emp["Name"]}");
```

---

## 🔷 DataRow States — RowState

Every DataRow has a `RowState` property tracking its change status:

| RowState      | Meaning                                             |
| ------------- | --------------------------------------------------- |
| `Unchanged` | No changes since last `AcceptChanges()`           |
| `Added`     | New row not yet saved to DB                         |
| `Modified`  | Existing row was changed                            |
| `Deleted`   | Row marked for deletion                             |
| `Detached`  | Created with `NewRow()`but not yet added to table |

```csharp
DataRow newRow = dt.NewRow();         // Detached
dt.Rows.Add(newRow);                  // Added
dt.Rows[0]["Salary"] = 9999;          // Modified
dt.Rows[1].Delete();                  // Deleted
dt.AcceptChanges();                   // all → Unchanged
```

---

## 🔷 DataTable in MVC Repository Pattern

```csharp
public DataTable GetAllEmployees()
{
    DataTable dt = new DataTable();
    string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

    using (SqlDataAdapter da = new SqlDataAdapter(sql, _cs))
    {
        da.Fill(dt);
    }

    return dt;   // returned to controller/BAL for display
}
```

---

## ⚠️ Common Mistakes

| Mistake                                     | What Happens                                               | Fix                                                                                        |
| ------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Reading `row["Col"]`without null check    | `InvalidCastException`on DB NULL                         | Use `row["Col"] == DBNull.Value ? null : row["Col"].ToString()`                          |
| Using `Rows.Find()`without PrimaryKey set | Exception                                                  | Set `dt.PrimaryKey`before using `Find()`                                               |
| `Delete()`then reading the row            | Row is still in collection but Deleted — causes confusion | Use `dt.Rows.Remove(row)`for immediate removal, or `AcceptChanges()`after `Delete()` |
| Adding raw `null`to a row                 | Exception                                                  | Use `DBNull.Value`instead of `null`                                                    |

---

## ⭐ Interview Quick-Fire

| Question                             | Answer                                                                             |
| ------------------------------------ | ---------------------------------------------------------------------------------- |
| What is DataTable?                   | In-memory single-table structure — rows + columns + constraints                   |
| How many tables in DataTable?        | One — single table only (DataSet holds multiple)                                  |
| What namespace?                      | `System.Data`                                                                    |
| How to add a row?                    | `dt.NewRow()`+ fill values +`dt.Rows.Add(row)`                                 |
| How to filter rows?                  | `dt.Select("filter expression")`→ returns `DataRow[]`                         |
| How to find by primary key?          | `dt.Rows.Find(keyValue)`— PrimaryKey must be set                                |
| What is RowState?                    | Tracks change status:`Unchanged`,`Added`,`Modified`,`Deleted`,`Detached` |
| What does `Clear()`do?             | Removes all rows — keeps column structure                                         |
| Difference:`Clone()`vs `Copy()`? | `Clone()`= structure only,`Copy()`= structure + data                           |
| Is DataTable part of DataSet?        | Yes — DataSet is a collection of DataTables                                       |
