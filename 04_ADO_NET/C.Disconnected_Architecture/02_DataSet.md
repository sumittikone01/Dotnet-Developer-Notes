
# 02 — DataSet

---

## 🎯 One-Line Definition

> **`DataSet` is an in-memory mini-database — it holds multiple DataTables, supports relationships between them, and lets you work with data completely offline after the connection is closed.**

---

## 🔷 What is DataSet?

A `DataSet` is like a  **temporary database that lives in your application's RAM** .
You fetch data from SQL Server, it goes into the DataSet, the connection closes, and you work with the data freely.

> 💡 Think of a DataSet like a **briefcase** — you go to the office (database), fill the briefcase with documents (tables), close the office (connection closed), and work with those documents at home (offline). When you're done, you take the briefcase back and file the changes.

---

## 🔷 The Problem DataSet Solves

```
WITHOUT DataSet (always connected):
  Your app → Open connection → Read row → Close → Open → Read → Close...
  Database is busy the whole time
  Doesn't scale under many users

WITH DataSet (disconnected):
  Your app → Open → Fetch ALL data into DataSet → Close
  Database is free — connection released immediately
  Your app works with DataSet in memory for as long as needed
  When done → Open → Save changes → Close
```

---

## 🔷 DataSet Structure — The Mini-Database

```
DataSet
├── Tables (DataTableCollection)
│   ├── DataTable "Employees"
│   │   ├── Columns: Id, Name, Role, Salary
│   │   ├── Rows:    1-Alice-Dev-75000
│   │   │            2-Bob-QA-55000
│   │   └── Constraints: PrimaryKey(Id)
│   │
│   └── DataTable "Departments"
│       ├── Columns: Id, Name
│       ├── Rows:    1-IT
│       │            2-HR
│       └── Constraints: PrimaryKey(Id)
│
└── Relations (DataRelationCollection)
    └── DataRelation "EmpDept"
        ├── ParentTable: Departments.Id
        └── ChildTable:  Employees.DeptId
```

---

## 🔷 Key Properties

| Property        | Type                       | What It Returns                            |
| --------------- | -------------------------- | ------------------------------------------ |
| `Tables`      | `DataTableCollection`    | All DataTables in this DataSet             |
| `Relations`   | `DataRelationCollection` | All DataRelations defined                  |
| `DataSetName` | `string`                 | Name of the DataSet                        |
| `HasErrors`   | `bool`                   | `true`if any table has validation errors |

---

## 🔷 Key Methods

| Method                    | What It Does                                               |
| ------------------------- | ---------------------------------------------------------- |
| `Fill()`                | (via SqlDataAdapter) Loads data into the DataSet           |
| `Tables.Add(DataTable)` | Adds a DataTable to the DataSet                            |
| `Tables["Name"]`        | Gets a DataTable by name                                   |
| `AcceptChanges()`       | Commits all pending changes in all tables                  |
| `RejectChanges()`       | Discards all pending changes in all tables                 |
| `HasChanges()`          | Returns `true`if any row has been added/modified/deleted |
| `GetChanges()`          | Returns a new DataSet with only the changed rows           |
| `Clear()`               | Removes all data from all tables (keeps structure)         |
| `Copy()`                | Returns a full copy — structure + data                    |
| `Clone()`               | Returns a copy of structure only — no data                |
| `Merge(DataSet)`        | Merges another DataSet's data into this one                |
| `WriteXml()`            | Saves the DataSet to XML                                   |
| `ReadXml()`             | Loads a DataSet from XML                                   |

---

## 🔷 Basic Setup — Load and Read

```csharp
using System.Data;
using Microsoft.Data.SqlClient;

string cs  = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";
string sql = "SELECT Id, Name, Role, Salary FROM Employee";

// Step 1: Create the DataSet
DataSet ds = new DataSet("EmployeeData");

// Step 2: Fill using SqlDataAdapter
using (SqlDataAdapter da = new SqlDataAdapter(sql, cs))
{
    da.Fill(ds, "Employees");
    // Connection opens → data fetched → connection CLOSED automatically
}

// Step 3: Work offline — connection already closed
DataTable empTable = ds.Tables["Employees"];
Console.WriteLine($"Total employees: {empTable.Rows.Count}");

foreach (DataRow row in empTable.Rows)
{
    Console.WriteLine($"{row["Id"]}  {row["Name"]}  {row["Salary"]}");
}
```

---

## 🔷 Multiple Tables in One DataSet

```csharp
DataSet ds = new DataSet();

// Fill first table
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(ds, "Employees");

// Fill second table — same DataSet, different table name
using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Department", cs))
    da.Fill(ds, "Departments");

// Access each table independently
DataTable employees   = ds.Tables["Employees"];
DataTable departments = ds.Tables["Departments"];

Console.WriteLine($"Employees:   {employees.Rows.Count}");
Console.WriteLine($"Departments: {departments.Rows.Count}");

// Access by index too
DataTable first  = ds.Tables[0];   // Employees
DataTable second = ds.Tables[1];   // Departments
```

---

## 🔷 Edit Data Offline — AcceptChanges / RejectChanges

```csharp
DataSet ds = new DataSet();

using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    da.Fill(ds, "Employees");

DataTable dt = ds.Tables["Employees"];

// ── Make changes offline ──────────────────────────────────────────
dt.Rows[0]["Salary"] = 85000;          // modify row 1
dt.Rows.Add(0, "Carol", "Analyst");    // add new row
dt.Rows[1].Delete();                   // mark row 2 for deletion

// ── Check if there are changes ────────────────────────────────────
Console.WriteLine($"Has changes: {ds.HasChanges()}");   // true

// ── Option A: Accept — commit all changes as permanent ────────────
ds.AcceptChanges();
// All rows now marked as "Unchanged" — changes locked in

// ── Option B: Reject — undo all changes since last AcceptChanges ──
ds.RejectChanges();
// All rows reverted to original state
```

---

## 🔷 GetChanges() — Only the Modified Rows

```csharp
// After making changes...
if (ds.HasChanges())
{
    // Get only rows that were added/modified/deleted
    DataSet changesOnly = ds.GetChanges();

    // Send only changes to the server — not the entire DataSet
    using (SqlDataAdapter da = new SqlDataAdapter("SELECT * FROM Employee", cs))
    {
        SqlCommandBuilder cb = new SqlCommandBuilder(da);
        da.Update(changesOnly.Tables["Employees"]);
    }

    ds.AcceptChanges();   // mark all as saved
}
```

---

## 🔷 Clone vs Copy

```csharp
// Copy — structure + data (full duplicate)
DataSet fullCopy = ds.Copy();

// Clone — structure only (empty DataSet, same columns/keys)
DataSet emptyStructure = ds.Clone();
// Useful for creating a template DataSet with known structure
```

---

## 🔷 DataSet in MVC Repository Pattern

```csharp
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // Return DataSet with employees and departments together
    public DataSet GetEmployeesWithDepartments()
    {
        DataSet ds = new DataSet("HR");

        using (SqlDataAdapter da1 = new SqlDataAdapter("SELECT * FROM Employee", _cs))
            da1.Fill(ds, "Employees");

        using (SqlDataAdapter da2 = new SqlDataAdapter("SELECT * FROM Department", _cs))
            da2.Fill(ds, "Departments");

        return ds;
    }
}
```

---

## 🔷 DataSet vs DataTable — When to Use Which

|            | `DataSet`                     | `DataTable`               |
| ---------- | ------------------------------- | --------------------------- |
| Holds      | Multiple tables                 | One table                   |
| Relations  | ✅ DataRelation support         | ❌ No cross-table relations |
| Use when   | Multiple tables needed together | Single table operation      |
| Memory     | More (multiple tables)          | Less (single table)         |
| Complexity | Higher                          | Simpler                     |
| Best for   | Forms with lookups, dashboard   | Single-table grids, lists   |

---

## ⭐ Interview Quick-Fire

| Question                                | Answer                                                                     |
| --------------------------------------- | -------------------------------------------------------------------------- |
| What is DataSet?                        | In-memory container holding multiple DataTables, used in disconnected mode |
| How many tables can DataSet hold?       | Multiple — no limit                                                       |
| What namespace?                         | `System.Data`                                                            |
| Does DataSet need a live DB connection? | ❌ No — works offline after `Fill()`                                    |
| What does `AcceptChanges()`do?        | Commits all pending changes — marks all rows as Unchanged                 |
| What does `RejectChanges()`do?        | Discards all pending changes since last `AcceptChanges()`                |
| What does `GetChanges()`return?       | A new DataSet containing only the modified/added/deleted rows              |
| Difference:`Clone()`vs `Copy()`?    | `Clone()`= structure only.`Copy()`= structure + data                   |
| Is DataSet faster than DataReader?      | ❌ No — DataReader is faster for reading; DataSet is more flexible        |
