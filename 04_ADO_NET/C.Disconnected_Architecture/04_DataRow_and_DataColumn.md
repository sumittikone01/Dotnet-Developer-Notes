
# 04 — DataRow and DataColumn

---

## 🎯 One-Line Definition

> **`DataColumn` defines the structure of a table (name, type, constraints) — `DataRow` holds the actual data values in that structure. Columns are the blueprint, Rows are the data.**

---

## 🔷 The Relationship

```
DataTable "Employee"
│
├── DataColumn "Id"     ← blueprint: name="Id", type=int, PrimaryKey
├── DataColumn "Name"   ← blueprint: name="Name", type=string, not null
├── DataColumn "Salary" ← blueprint: name="Salary", type=decimal
│
├── DataRow → { 1, "Alice", 75000 }  ← actual data following the blueprint
├── DataRow → { 2, "Bob",   55000 }
└── DataRow → { 3, "Carol", 68000 }

Columns = structure  →  defined ONCE
Rows    = data       →  added MANY TIMES
```

---

# PART A — DataColumn

---

## 🔷 What is DataColumn?

`DataColumn` defines **one column** in a DataTable — its name, data type, default value, whether it allows nulls, and more.

> 💡 Think of `DataColumn` as a **column header in Excel** — it defines what kind of data that column holds and what rules apply to it.

---

## 🔷 DataColumn Key Properties

| Property              | Type       | What It Controls                                      |
| --------------------- | ---------- | ----------------------------------------------------- |
| `ColumnName`        | `string` | Name of the column —`"Id"`,`"Name"`              |
| `DataType`          | `Type`   | C# type →`typeof(int)`,`typeof(string)`          |
| `AllowDBNull`       | `bool`   | Whether the column can hold NULL (default:`true`)   |
| `DefaultValue`      | `object` | Value used when no value is provided                  |
| `MaxLength`         | `int`    | Max characters for string columns (`-1`= unlimited) |
| `AutoIncrement`     | `bool`   | Auto-increments value for each new row                |
| `AutoIncrementSeed` | `long`   | Starting value for auto-increment (default: 0)        |
| `AutoIncrementStep` | `long`   | Increment step (default: 1)                           |
| `Unique`            | `bool`   | All values in column must be unique                   |
| `ReadOnly`          | `bool`   | Column value cannot be changed after row added        |
| `Caption`           | `string` | Display label (can differ from `ColumnName`)        |
| `Ordinal`           | `int`    | Column's position index in the table (0-based)        |

---

## 🔷 Creating DataColumns

### Simple way — inside `Columns.Add()`

```csharp
DataTable dt = new DataTable("Employee");

// ── Quickest — name + type ────────────────────────────────────────
dt.Columns.Add("Id",     typeof(int));
dt.Columns.Add("Name",   typeof(string));
dt.Columns.Add("Role",   typeof(string));
dt.Columns.Add("Salary", typeof(decimal));
dt.Columns.Add("HireDate", typeof(DateTime));
dt.Columns.Add("IsActive", typeof(bool));
```

### With full control — `DataColumn` object

```csharp
// ── Id: int, auto-increment, primary key ──────────────────────────
DataColumn colId = new DataColumn("Id", typeof(int));
colId.AutoIncrement     = true;
colId.AutoIncrementSeed = 1;    // starts at 1
colId.AutoIncrementStep = 1;    // increments by 1
colId.AllowDBNull       = false;
dt.Columns.Add(colId);

// ── Name: string, required, max 100 chars ─────────────────────────
DataColumn colName = new DataColumn("Name", typeof(string));
colName.AllowDBNull = false;
colName.MaxLength   = 100;
dt.Columns.Add(colName);

// ── Salary: decimal with default value ────────────────────────────
DataColumn colSalary = new DataColumn("Salary", typeof(decimal));
colSalary.DefaultValue = 30000m;
colSalary.AllowDBNull  = false;
dt.Columns.Add(colSalary);

// ── IsActive: bool, default true ──────────────────────────────────
DataColumn colActive = new DataColumn("IsActive", typeof(bool));
colActive.DefaultValue = true;
dt.Columns.Add(colActive);

// ── Set Primary Key ───────────────────────────────────────────────
dt.PrimaryKey = new DataColumn[] { colId };
```

---

## 🔷 Reading Column Info

```csharp
// After filling from DB or creating manually
foreach (DataColumn col in dt.Columns)
{
    Console.WriteLine($"Name: {col.ColumnName}  |  Type: {col.DataType.Name}  |  Ordinal: {col.Ordinal}");
}

// Access a specific column by name
DataColumn salaryCol = dt.Columns["Salary"];
Console.WriteLine($"Salary type: {salaryCol.DataType}");
Console.WriteLine($"Allows null: {salaryCol.AllowDBNull}");

// Access by index
DataColumn firstCol = dt.Columns[0];
Console.WriteLine($"First column: {firstCol.ColumnName}");
```

---

# PART B — DataRow

---

## 🔷 What is DataRow?

`DataRow` holds the actual data for one record in a DataTable.
Each row contains one value per column, tracks its own change state, and can hold its original and current values separately.

> 💡 Think of `DataRow` as a **single filled-in form** — the columns are the blank fields on the form, and the DataRow is one person's completed answers.

---

## 🔷 DataRow Key Properties

| Property      | Type             | What It Returns                          |
| ------------- | ---------------- | ---------------------------------------- |
| `RowState`  | `DataRowState` | Current change status of the row         |
| `HasErrors` | `bool`         | `true`if the row has validation errors |
| `RowError`  | `string`       | Error message for this row               |
| `ItemArray` | `object[]`     | All column values as an array            |
| `Table`     | `DataTable`    | The parent DataTable this row belongs to |

---

## 🔷 DataRow Key Methods

| Method                               | What It Does                                |
| ------------------------------------ | ------------------------------------------- |
| `row["ColName"]`                   | Get or set a column value by name           |
| `row[index]`                       | Get or set a column value by index          |
| `row.Delete()`                     | Mark row for deletion (RowState = Deleted)  |
| `row.BeginEdit()`                  | Start an edit — suspends row change events |
| `row.EndEdit()`                    | Commit the edit                             |
| `row.CancelEdit()`                 | Discard changes made during `BeginEdit()` |
| `row.AcceptChanges()`              | Mark this row as Unchanged                  |
| `row.RejectChanges()`              | Revert this row to its original values      |
| `row.SetColumnError("Col", "msg")` | Set a validation error on a column          |
| `row.GetChildRows(relation)`       | Get child rows via a DataRelation           |
| `row.GetParentRow(relation)`       | Get parent row via a DataRelation           |

---

## 🔷 Adding Rows — Three Ways

```csharp
DataTable dt = new DataTable("Employee");
dt.Columns.Add("Id",     typeof(int));
dt.Columns.Add("Name",   typeof(string));
dt.Columns.Add("Salary", typeof(decimal));

// ── Way 1: NewRow() — full control ────────────────────────────────
DataRow row1 = dt.NewRow();    // creates blank row matching table schema
row1["Id"]     = 1;
row1["Name"]   = "Alice";
row1["Salary"] = 75000m;
dt.Rows.Add(row1);

// ── Way 2: Rows.Add() with values directly ─────────────────────────
dt.Rows.Add(2, "Bob", 55000m);

// ── Way 3: ItemArray — assign all columns at once ─────────────────
DataRow row3 = dt.NewRow();
row3.ItemArray = new object[] { 3, "Carol", 68000m };
dt.Rows.Add(row3);
```

---

## 🔷 Reading Row Values

```csharp
foreach (DataRow row in dt.Rows)
{
    // ── By column name (most readable) ────────────────────────────
    int     id     = (int)row["Id"];
    string  name   = row["Name"].ToString();
    decimal salary = (decimal)row["Salary"];

    // ── By column index ───────────────────────────────────────────
    int     id2    = (int)row[0];
    string  name2  = row[1].ToString();

    Console.WriteLine($"{id}  {name}  {salary}");
}

// Access single specific row by index
DataRow first = dt.Rows[0];
Console.WriteLine($"First employee: {first["Name"]}");
```

---

## 🔷 Updating Row Values

```csharp
// Direct update
dt.Rows[0]["Salary"] = 85000m;
dt.Rows[0]["Role"]   = "Senior Developer";

// Update using Find (requires PrimaryKey to be set)
DataRow found = dt.Rows.Find(1);   // finds row where PrimaryKey = 1
if (found != null)
{
    found["Salary"] = 90000m;
}

// Update with BeginEdit / EndEdit (suspends change events during edit)
DataRow row = dt.Rows[0];
row.BeginEdit();
row["Name"]   = "Alice Johnson";
row["Salary"] = 90000m;
row.EndEdit();    // commits — fires events once

// Cancel mid-edit
row.BeginEdit();
row["Salary"] = 0;
row.CancelEdit();   // reverts — Salary stays at 90000
```

---

## 🔷 Deleting Rows

```csharp
// ── Soft delete — marks RowState = Deleted (preferred)  ───────────
dt.Rows[1].Delete();
// Row still exists in Rows collection but RowState = Deleted
// SqlDataAdapter.Update() sends DELETE command to DB
// After AcceptChanges() — row is permanently removed

// ── Hard delete — removes immediately from collection ─────────────
DataRow toRemove = dt.Rows[0];
dt.Rows.Remove(toRemove);    // removed immediately, no RowState tracking

// ── Remove by index ───────────────────────────────────────────────
dt.Rows.RemoveAt(0);
```

---

## 🔷 RowState — The Change Tracker

Every row tracks its own state:

| RowState      | When                                      | Description                 |
| ------------- | ----------------------------------------- | --------------------------- |
| `Detached`  | After `NewRow()`, before `Rows.Add()` | Not part of table yet       |
| `Added`     | After `Rows.Add()`                      | New row not yet saved to DB |
| `Unchanged` | After `Fill()`or `AcceptChanges()`    | No pending changes          |
| `Modified`  | After editing a value                     | Changed, not yet saved      |
| `Deleted`   | After `row.Delete()`                    | Marked for removal          |

```csharp
DataRow r = dt.NewRow();
Console.WriteLine(r.RowState);     // Detached

dt.Rows.Add(r);
Console.WriteLine(r.RowState);     // Added

dt.AcceptChanges();
Console.WriteLine(r.RowState);     // Unchanged

r["Salary"] = 99999m;
Console.WriteLine(r.RowState);     // Modified

r.Delete();
Console.WriteLine(r.RowState);     // Deleted

dt.AcceptChanges();
// row is gone — fully removed from Rows collection
```

---

## 🔷 Handling NULL in DataRow

```csharp
// ❌ WRONG — C# null is not DB NULL
row["Email"] = null;               // throws ArgumentNullException

// ✅ CORRECT — use DBNull.Value for database NULLs
row["Email"] = DBNull.Value;

// ── Reading with null check ───────────────────────────────────────
string email = row["Email"] == DBNull.Value
               ? null
               : row["Email"].ToString();

// ── Safe string read ──────────────────────────────────────────────
string dept = row["Department"] as string;   // null if DBNull
```

---

## 🔷 Complete Example — Manually Build and Use a DataTable

```csharp
DataTable dt = new DataTable("Employee");

// Define structure
DataColumn colId = new DataColumn("Id", typeof(int));
colId.AutoIncrement     = true;
colId.AutoIncrementSeed = 1;
colId.AutoIncrementStep = 1;
dt.Columns.Add(colId);
dt.Columns.Add("Name",   typeof(string));
dt.Columns.Add("Role",   typeof(string));
dt.Columns.Add("Salary", typeof(decimal));
dt.PrimaryKey = new DataColumn[] { dt.Columns["Id"] };

// Add data
dt.Rows.Add(DBNull.Value, "Alice", "Developer", 75000m);  // Id auto-set to 1
dt.Rows.Add(DBNull.Value, "Bob",   "QA",        55000m);  // Id auto-set to 2
dt.Rows.Add(DBNull.Value, "Carol", "PM",         68000m);  // Id auto-set to 3

// Read
foreach (DataRow row in dt.Rows)
    Console.WriteLine($"{row["Id"]}  {row["Name"]}  {row["Salary"]}");

// Filter
DataRow[] devs = dt.Select("Role = 'Developer'");
Console.WriteLine($"Developers: {devs.Length}");

// Update
DataRow emp = dt.Rows.Find(1);
emp["Salary"] = 80000m;

// RowState tracking
Console.WriteLine(dt.Rows[0].RowState);   // Modified

// Commit
dt.AcceptChanges();
Console.WriteLine(dt.Rows[0].RowState);   // Unchanged
```

---

## ⭐ Interview Quick-Fire

| Question                       | Answer                                                                             |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| What is DataColumn?            | Defines one column's structure — name, type, constraints                          |
| What is DataRow?               | Holds one record's data values                                                     |
| How to create a row?           | `dt.NewRow()`then fill values then `dt.Rows.Add(row)`                          |
| Five RowState values?          | `Detached`,`Added`,`Unchanged`,`Modified`,`Deleted`                      |
| How to find a row by PK?       | `dt.Rows.Find(keyValue)`— requires PrimaryKey to be set                         |
| Soft vs hard delete?           | `row.Delete()`= soft (RowState = Deleted).`Rows.Remove(row)`= hard (immediate) |
| How to pass NULL to a row?     | `DBNull.Value`— never C#`null`                                                |
| What is `BeginEdit()`for?    | Suspends row events during multi-column edit — commit with `EndEdit()`          |
| What does `AutoIncrement`do? | Column auto-generates incrementing int values for each new row                     |
