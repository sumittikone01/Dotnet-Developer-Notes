
# 03 — Update with ADO.NET

---

## 🎯 One-Line Definition

> **UPDATE modifies existing rows — use `ExecuteNonQuery()` in Connected Architecture for a direct update, or modify a DataRow and call `adapter.Update()` in Disconnected Architecture.**

---

## 🔷 Two Ways to UPDATE

```
┌─────────────────────────────────────────────────────────────────┐
│  WAY 1 — CONNECTED (SqlCommand + ExecuteNonQuery)               │
│  Direct SQL UPDATE → immediate → simple                         │
│  Best for: single record updates, API/controller actions        │
├─────────────────────────────────────────────────────────────────┤
│  WAY 2 — DISCONNECTED (SqlDataAdapter + DataSet.Update)         │
│  Find row in DataTable → change values → RowState = Modified    │
│  adapter.Update() sends UPDATE only for changed rows            │
│  Best for: form editing, batch updates, offline changes         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Way 1 — Connected: SqlCommand + ExecuteNonQuery

### How It Works

```
Open Connection
  ↓
Create SqlCommand with UPDATE SQL
  ↓
Add @parameters — values to SET and WHERE condition
  ↓
ExecuteNonQuery() → sends UPDATE to SQL Server → returns rows affected
  ↓
Close Connection (auto via using)
```

### Complete Code

```csharp
public int Update(int id, string name, string role, string email, int salary)
{
    string sql = @"UPDATE Employee
                   SET Name   = @Name,
                       Role   = @Role,
                       Email  = @Email,
                       Salary = @Salary
                   WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100).Value = name;
            cmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100).Value = role;
            cmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100).Value = email;
            cmd.Parameters.Add("@Salary", SqlDbType.Int).Value          = salary;
            cmd.Parameters.Add("@Id",     SqlDbType.Int).Value          = id;

            int rows = cmd.ExecuteNonQuery();

            if (rows > 0)
                Console.WriteLine($"✅ Updated — {rows} row(s) affected.");
            else
                Console.WriteLine($"⚠️ No record found with Id = {id}");

            return rows;
        }
    }
}
```

---

## 🔷 Way 2 — Disconnected: DataAdapter + DataSet

### How It Works

```
Fill DataSet from DB (loads all rows)
  ↓
Find the row by Primary Key → table.Rows.Find(id)
  ↓
Change column values on the DataRow → RowState = Modified
  ↓
Set adapter.UpdateCommand with SQL + mapped parameters
  ↓
adapter.Update() → finds Modified rows → fires UpdateCommand for each
```

### Important: DataRowVersion.Original

```
When you update a row's Id column or use Id in WHERE:
  Current version: the Id value AFTER any edits
  Original version: the Id value AS IT WAS when loaded from DB

For WHERE Id = @Id → always use DataRowVersion.Original
Otherwise, if someone accidentally changed Id, the wrong row gets updated.
```

### Complete Code (from your working example)

```csharp
public int UpdateRecord(int id, string name, string role, string email, int salary)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        // Step 1: Load all records into DataSet
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
        DataSet ds = new DataSet();
        adapter.Fill(ds, "Employee");

        DataTable table = ds.Tables["Employee"];

        // ⚠️ PrimaryKey must be set for Rows.Find() to work
        table.PrimaryKey = new DataColumn[] { table.Columns["Id"] };

        // Step 2: Find the row by primary key
        DataRow row = table.Rows.Find(id);

        if (row == null)
        {
            Console.WriteLine("Record not found.");
            return 0;
        }

        // Step 3: Modify values → RowState becomes "Modified"
        row["Name"]   = name;
        row["Role"]   = role;
        row["Email"]  = email;
        row["Salary"] = salary;

        // Step 4: Define the UpdateCommand
        SqlCommand updateCmd = new SqlCommand(
            @"UPDATE Employee
              SET Name=@Name, Role=@Role, Email=@Email, Salary=@Salary
              WHERE Id=@Id",
            con);

        // Map parameters → DataTable column names
        updateCmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100, "Name");
        updateCmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100, "Role");
        updateCmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100, "Email");
        updateCmd.Parameters.Add("@Salary", SqlDbType.Int,     0,   "Salary");

        // @Id must use Original version — the Id as loaded from DB
        SqlParameter idParam = updateCmd.Parameters.Add("@Id", SqlDbType.Int);
        idParam.SourceColumn  = "Id";
        idParam.SourceVersion = DataRowVersion.Original;  // ← critical

        adapter.UpdateCommand = updateCmd;

        // Step 5: Update() fires SQL for Modified rows only
        return adapter.Update(ds, "Employee");
    }
}
```

---

## 🔷 Why `DataRowVersion.Original`?

```
SCENARIO: You load employee Id=5 from DB and change their Name.

DataRow has TWO versions of each value:
  Current  version → the value NOW  (after your edit)
  Original version → the value WHEN LOADED (before any edits)

For the WHERE clause:
  WHERE Id = @Id
  We need the ORIGINAL Id to find the correct row in the database.
  If we used Current (which is the default), and someone had edited Id,
  we might UPDATE the wrong row or no row at all.

idParam.SourceVersion = DataRowVersion.Original
→ "Use the Id as it was when this row was loaded from the DB"
→ Ensures the correct row is found and updated
```

---

## 🔷 `DataRowVersion` Values

| Version      | What It Holds                                            |
| ------------ | -------------------------------------------------------- |
| `Original` | Value before any changes — as loaded from DB            |
| `Current`  | Value right now — after any edits                       |
| `Default`  | Uses `Current`if row is Modified;`Original`otherwise |
| `Proposed` | Value during a `BeginEdit()`/`EndEdit()`edit session |

---

## 🔷 Connected vs Disconnected UPDATE

|                             | Connected                | Disconnected                          |
| --------------------------- | ------------------------ | ------------------------------------- |
| Setup                       | Simple                   | More setup (commands + SourceVersion) |
| Rows.Find() needed?         | ❌ Just use WHERE in SQL | ✅ Need to find row in DataTable      |
| `DataRowVersion.Original` | Not needed               | ✅ Required for WHERE Id param        |
| Best for                    | Single row update        | Batch updates, form editing           |

---

## ⚠️ Common Mistakes

| Mistake                                      | What Happens                          | Fix                                                                          |
| -------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------- |
| Forgetting `DataRowVersion.Original`for Id | Updates wrong row or no row           | Always set `SourceVersion = DataRowVersion.Original`on the WHERE key param |
| No PrimaryKey set before `Rows.Find()`     | Exception                             | Set `table.PrimaryKey`before calling `Find()`                            |
| Return 0 not handled                         | Silent failure — user sees nothing   | Check return value and show appropriate message                              |
| Not setting `SourceColumn`                 | Adapter sends null for that parameter | 4th argument or `SourceColumn`property must match DataTable column name    |

---

## ⭐ Interview Quick-Fire

| Question                                           | Answer                                                                 |
| -------------------------------------------------- | ---------------------------------------------------------------------- |
| Connected UPDATE uses which method?                | `ExecuteNonQuery()`                                                  |
| Disconnected UPDATE — which RowState triggers it? | `Modified`— set when you change a column value on a DataRow         |
| Why use `DataRowVersion.Original`?               | Ensures the WHERE clause uses the original key value as loaded from DB |
| How to find a row in DataTable?                    | `table.Rows.Find(id)`— requires `table.PrimaryKey`to be set       |
| Return value of `ExecuteNonQuery()`for UPDATE?   | `int`— rows affected (0 = no match, 1+ = success)                   |
