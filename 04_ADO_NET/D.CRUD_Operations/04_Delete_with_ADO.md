
# 04 — Delete with ADO.NET

---

## 🎯 One-Line Definition

> **DELETE removes rows from the database — use `ExecuteNonQuery()` in Connected Architecture for a direct delete, or call `row.Delete()` on a DataRow and call `adapter.Update()` in Disconnected Architecture.**

---

## 🔷 Two Ways to DELETE

```
┌─────────────────────────────────────────────────────────────────┐
│  WAY 1 — CONNECTED (SqlCommand + ExecuteNonQuery)               │
│  Direct DELETE → immediate → simple                             │
│  Best for: single record delete, API/controller actions         │
├─────────────────────────────────────────────────────────────────┤
│  WAY 2 — DISCONNECTED (SqlDataAdapter + DataSet.Update)         │
│  Find row → call row.Delete() → RowState = Deleted             │
│  adapter.Update() sends DELETE only for marked rows            │
│  Best for: batch deletes, grid row removal, offline changes    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Way 1 — Connected: SqlCommand + ExecuteNonQuery

### How It Works

```
Open Connection
  ↓
Create SqlCommand with DELETE SQL
  ↓
Add @Id parameter — identifies which row to delete
  ↓
ExecuteNonQuery() → sends DELETE to SQL Server → returns rows affected
  ↓
Close Connection (auto via using)
```

### Complete Code

```csharp
public int Delete(int id)
{
    string sql = "DELETE FROM Employee WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.Add("@Id", SqlDbType.Int).Value = id;

            int rows = cmd.ExecuteNonQuery();

            if (rows > 0)
                Console.WriteLine($"🗑️ Deleted — {rows} row(s) removed.");
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
Set PrimaryKey on the DataTable (required for Rows.Find)
  ↓
Find the row → table.Rows.Find(id) → returns DataRow or null
  ↓
row.Delete()  → marks RowState = Deleted (soft delete)
  ↓
Set adapter.DeleteCommand with SQL + @Id using Original version
  ↓
adapter.Update() → finds Deleted rows → fires DeleteCommand for each
```

### Critical: `DataRowVersion.Original` on Delete

```
After row.Delete(), the row still exists in the Rows collection.
Its values are still accessible via DataRowVersion.Original.
For the WHERE clause in DELETE, we must use Original version
to ensure we're deleting the correct row from the database.
```

### Complete Code (from your working example)

```csharp
public int DeleteRecord(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        // Step 1: Load all records into DataSet
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
        DataSet ds = new DataSet();
        adapter.Fill(ds, "Employee");

        DataTable table = ds.Tables["Employee"];

        // Step 2: Set PrimaryKey — required for Rows.Find()
        table.PrimaryKey = new DataColumn[] { table.Columns["Id"] };

        // Step 3: Find the row by primary key
        DataRow row = table.Rows.Find(id);

        if (row == null)
        {
            Console.WriteLine("Record not found.");
            return 0;
        }

        // Step 4: Mark for deletion → RowState becomes "Deleted"
        row.Delete();

        // Step 5: Define the DeleteCommand
        SqlCommand deleteCmd = new SqlCommand(
            "DELETE FROM Employee WHERE Id = @Id",
            con);

        // @Id must use Original version — Id before any changes
        SqlParameter idParam = deleteCmd.Parameters.Add("@Id", SqlDbType.Int);
        idParam.SourceColumn  = "Id";
        idParam.SourceVersion = DataRowVersion.Original;  // ← critical

        adapter.DeleteCommand = deleteCmd;

        // Step 6: Update() fires DELETE for all Deleted rows
        return adapter.Update(ds, "Employee");
    }
}
```

---

## 🔷 `row.Delete()` vs `Rows.Remove(row)` — Know the Difference

```
row.Delete()
  → Marks RowState = Deleted
  → Row still exists in Rows collection (until AcceptChanges)
  → SqlDataAdapter.Update() will fire DELETE SQL for this row
  → Use when you want the adapter to sync the delete to DB

Rows.Remove(row)   OR   Rows.RemoveAt(index)
  → Removes the row immediately from the collection
  → RowState tracking lost
  → SqlDataAdapter.Update() will NOT fire DELETE SQL
  → Use when you want to discard a row locally without touching DB

✅ For Disconnected CRUD → always use row.Delete()
```

---

## 🔷 After adapter.Update() — AcceptChanges

```csharp
// After Update() succeeds — the Deleted rows are still in the collection
// with RowState = Deleted until you call AcceptChanges()

adapter.Update(ds, "Employee");
ds.AcceptChanges();   // ← permanently removes Deleted rows from collection
                      // All remaining rows → RowState = Unchanged
```

---

## 🔷 Delete with Confirmation Check

```csharp
public int DeleteWithConfirm(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
        DataSet ds = new DataSet();
        adapter.Fill(ds, "Employee");

        DataTable table = ds.Tables["Employee"];
        table.PrimaryKey = new DataColumn[] { table.Columns["Id"] };

        DataRow row = table.Rows.Find(id);

        if (row == null)
        {
            Console.WriteLine($"❌ Employee with Id={id} does not exist.");
            return 0;
        }

        // Show what will be deleted before confirming
        Console.WriteLine($"About to delete: {row["Name"]} — {row["Role"]}");

        row.Delete();

        SqlCommand deleteCmd = new SqlCommand(
            "DELETE FROM Employee WHERE Id = @Id", con);
        SqlParameter idParam = deleteCmd.Parameters.Add("@Id", SqlDbType.Int);
        idParam.SourceColumn  = "Id";
        idParam.SourceVersion = DataRowVersion.Original;

        adapter.DeleteCommand = deleteCmd;

        int rows = adapter.Update(ds, "Employee");
        ds.AcceptChanges();

        Console.WriteLine($"✅ Deleted successfully.");
        return rows;
    }
}
```

---

## 🔷 Full CRUD Program — From Your Working Code

```csharp
using Microsoft.Data.SqlClient;
using System;
using System.Data;

namespace CRUD_Disconnected
{
    class Program
    {
        static string _cs = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";

        static void Main(string[] args)
        {
            Console.WriteLine("Enter process: insert, update, delete, show");
            string process = Console.ReadLine().ToLower();
            int result = 0;

            switch (process)
            {
                case "insert": result = InsertRecord(); break;
                case "update": result = UpdateRecord(); break;
                case "delete": result = DeleteRecord(); break;
                case "show":   ShowAllRecords(); return;
                default: Console.WriteLine("Invalid process."); return;
            }

            Console.WriteLine(result > 0 ? "✅ Operation Successful." : "❌ Operation Failed.");
            Console.WriteLine("\n── All Records ──\n");
            ShowAllRecords();
        }

        static void ShowAllRecords()
        {
            using SqlConnection con = new SqlConnection(_cs);
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
            DataSet ds = new DataSet();
            adapter.Fill(ds, "Employee");

            foreach (DataRow row in ds.Tables["Employee"].Rows)
                Console.WriteLine($"Id:{row["Id"]}  {row["Name"]}  {row["Role"]}  {row["Salary"]}");
        }

        static int InsertRecord()
        {
            Console.Write("Name: ");   string name   = Console.ReadLine();
            Console.Write("Role: ");   string role   = Console.ReadLine();
            Console.Write("Email: ");  string email  = Console.ReadLine();
            Console.Write("Salary: "); int    salary = Convert.ToInt32(Console.ReadLine());

            using SqlConnection con = new SqlConnection(_cs);
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
            DataSet ds = new DataSet();
            adapter.Fill(ds, "Employee");

            DataRow newRow = ds.Tables["Employee"].NewRow();
            newRow["Name"] = name; newRow["Role"] = role;
            newRow["Email"] = email; newRow["Salary"] = salary;
            ds.Tables["Employee"].Rows.Add(newRow);

            SqlCommand cmd = new SqlCommand(
                "INSERT INTO Employee(Name,Role,Email,Salary) VALUES(@Name,@Role,@Email,@Salary)", con);
            cmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100, "Name");
            cmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100, "Role");
            cmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100, "Email");
            cmd.Parameters.Add("@Salary", SqlDbType.Int,     0,   "Salary");
            adapter.InsertCommand = cmd;

            return adapter.Update(ds, "Employee");
        }

        static int UpdateRecord()
        {
            Console.Write("Employee Id: "); int id = Convert.ToInt32(Console.ReadLine());
            Console.Write("Name: ");   string name   = Console.ReadLine();
            Console.Write("Role: ");   string role   = Console.ReadLine();
            Console.Write("Email: ");  string email  = Console.ReadLine();
            Console.Write("Salary: "); int    salary = Convert.ToInt32(Console.ReadLine());

            using SqlConnection con = new SqlConnection(_cs);
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
            DataSet ds = new DataSet();
            adapter.Fill(ds, "Employee");

            DataTable table = ds.Tables["Employee"];
            table.PrimaryKey = new DataColumn[] { table.Columns["Id"] };
            DataRow row = table.Rows.Find(id);
            if (row == null) { Console.WriteLine("Not found."); return 0; }

            row["Name"] = name; row["Role"] = role;
            row["Email"] = email; row["Salary"] = salary;

            SqlCommand cmd = new SqlCommand(
                "UPDATE Employee SET Name=@Name,Role=@Role,Email=@Email,Salary=@Salary WHERE Id=@Id", con);
            cmd.Parameters.Add("@Name",   SqlDbType.VarChar, 100, "Name");
            cmd.Parameters.Add("@Role",   SqlDbType.VarChar, 100, "Role");
            cmd.Parameters.Add("@Email",  SqlDbType.VarChar, 100, "Email");
            cmd.Parameters.Add("@Salary", SqlDbType.Int,     0,   "Salary");
            SqlParameter idParam = cmd.Parameters.Add("@Id", SqlDbType.Int);
            idParam.SourceColumn = "Id";
            idParam.SourceVersion = DataRowVersion.Original;
            adapter.UpdateCommand = cmd;

            return adapter.Update(ds, "Employee");
        }

        static int DeleteRecord()
        {
            Console.Write("Employee Id to delete: "); int id = Convert.ToInt32(Console.ReadLine());

            using SqlConnection con = new SqlConnection(_cs);
            SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
            DataSet ds = new DataSet();
            adapter.Fill(ds, "Employee");

            DataTable table = ds.Tables["Employee"];
            table.PrimaryKey = new DataColumn[] { table.Columns["Id"] };
            DataRow row = table.Rows.Find(id);
            if (row == null) { Console.WriteLine("Not found."); return 0; }

            row.Delete();

            SqlCommand cmd = new SqlCommand("DELETE FROM Employee WHERE Id=@Id", con);
            SqlParameter idParam = cmd.Parameters.Add("@Id", SqlDbType.Int);
            idParam.SourceColumn = "Id";
            idParam.SourceVersion = DataRowVersion.Original;
            adapter.DeleteCommand = cmd;

            return adapter.Update(ds, "Employee");
        }
    }
}
```

---

## 🔷 All 4 CRUD — Side-by-Side Quick Reference

| Operation        | Connected                              | Disconnected                                            |
| ---------------- | -------------------------------------- | ------------------------------------------------------- |
| **INSERT** | `ExecuteNonQuery()`with INSERT SQL   | `dt.NewRow()`→`Rows.Add()`→`adapter.Update()`   |
| **READ**   | `ExecuteReader()`→`SqlDataReader` | `adapter.Fill()`→`DataTable`/`DataSet`           |
| **UPDATE** | `ExecuteNonQuery()`with UPDATE SQL   | `Rows.Find()`→ change values →`adapter.Update()`  |
| **DELETE** | `ExecuteNonQuery()`with DELETE SQL   | `Rows.Find()`→`row.Delete()`→`adapter.Update()` |

---

## ⚠️ Common Mistakes

| Mistake                                            | What Happens                    | Fix                                                    |
| -------------------------------------------------- | ------------------------------- | ------------------------------------------------------ |
| Using `Rows.Remove()`instead of `row.Delete()` | Adapter doesn't fire DELETE SQL | Use `row.Delete()`for disconnected delete            |
| `Rows.Find()`without setting `PrimaryKey`      | Exception — Find requires PK   | Always set `table.PrimaryKey`before using `Find()` |
| Forgetting `DataRowVersion.Original`             | Deletes wrong row or no row     | `idParam.SourceVersion = DataRowVersion.Original`    |
| Not checking if row is null                        | NullReferenceException          | Always check `if (row == null)`after `Rows.Find()` |

---

## ⭐ Interview Quick-Fire

| Question                                              | Answer                                                                                                          |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Connected DELETE uses which method?                   | `ExecuteNonQuery()`                                                                                           |
| Disconnected DELETE — which RowState triggers it?    | `Deleted`— set when `row.Delete()`is called                                                                |
| `row.Delete()`vs `Rows.Remove(row)`?              | `Delete()`= soft delete (RowState=Deleted, adapter syncs to DB).`Remove()`= immediate removal, no DB sync   |
| Why `DataRowVersion.Original`for DELETE?            | `row.Delete()`makes current values inaccessible — Original still holds the Id to identify the correct DB row |
| What happens if `Rows.Find()`returns null?          | Record doesn't exist — must handle gracefully with null check                                                  |
| After `adapter.Update()`for delete — what to call? | `ds.AcceptChanges()`— permanently removes Deleted rows from collection                                       |
