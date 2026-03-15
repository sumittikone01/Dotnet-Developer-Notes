
# 02 — Read with ADO.NET

---

## 🎯 One-Line Definition

> **READ fetches data from the database — use `SqlDataReader` + `ExecuteReader()` in Connected Architecture for fast row-by-row streaming, or `SqlDataAdapter` + `Fill()` in Disconnected Architecture to load everything into memory.**

---

## 🔷 Two Ways to READ

```
┌────────────────────────────────────────────────────────────────────┐
│  WAY 1 — CONNECTED  (SqlDataReader)                                │
│  Streams rows one at a time → forward-only → fast → low memory    │
│  Best for: read-only lists, login checks, large result sets        │
├────────────────────────────────────────────────────────────────────┤
│  WAY 2 — DISCONNECTED (SqlDataAdapter + DataSet/DataTable)         │
│  All rows loaded into memory → connection closed → work offline    │
│  Best for: Kendo Grids, reports, editable data                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Way 1 — Connected: SqlDataReader

### Read All Rows into a List

```csharp
public List<Employee> GetAll()
{
    var employees = new List<Employee>();
    string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee ORDER BY Name";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    employees.Add(new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Email  = reader["Email"] as string,   // nullable
                        Salary = reader.GetInt32(4)
                    });
                }
            }
        }
    }

    return employees;
}
```

### Read Single Row by ID

```csharp
public Employee GetById(int id)
{
    string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                if (reader.Read())   // ← if, not while — single row
                {
                    return new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Email  = reader["Email"] as string,
                        Salary = reader.GetInt32(4)
                    };
                }
                return null;   // not found
            }
        }
    }
}
```

### Read Count (ExecuteScalar)

```csharp
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
```

---

## 🔷 Way 2 — Disconnected: SqlDataAdapter + DataSet

### Read All into DataSet (from your working code)

```csharp
public void ShowAllRecords()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        SqlDataAdapter adapter = new SqlDataAdapter("SELECT * FROM Employee", con);
        DataSet ds = new DataSet();

        adapter.Fill(ds, "Employee");
        // Fill: opens connection → executes SELECT → loads all rows → closes connection

        foreach (DataRow row in ds.Tables["Employee"].Rows)
        {
            Console.WriteLine(
                $"Id: {row["Id"]}, Name: {row["Name"]}, " +
                $"Role: {row["Role"]}, Email: {row["Email"]}, Salary: {row["Salary"]}");
        }
    }
}
```

### Read into DataTable (simpler — single table)

```csharp
public DataTable GetAllAsTable()
{
    DataTable dt = new DataTable();
    string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee ORDER BY Name";

    using (SqlDataAdapter da = new SqlDataAdapter(sql, _cs))
    {
        da.Fill(dt);
    }

    return dt;
}
```

### Read with Filter

```csharp
public DataTable GetByDepartment(string dept)
{
    DataTable dt = new DataTable();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        string sql = "SELECT * FROM Employee WHERE Department = @Dept";
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
```

---

## 🔷 Accessing DataRow Values — Three Ways

```csharp
foreach (DataRow row in ds.Tables["Employee"].Rows)
{
    // By column name (readable)
    string name = row["Name"].ToString();
    int salary  = Convert.ToInt32(row["Salary"]);

    // By column index (fastest)
    int id      = (int)row[0];
    string nm   = row[1].ToString();

    // Null-safe string
    string email = row["Email"] == DBNull.Value ? null : row["Email"].ToString();
}
```

---

## 🔷 Connected vs Disconnected READ — When to Use Which

|            | Connected (DataReader)   | Disconnected (DataAdapter)  |
| ---------- | ------------------------ | --------------------------- |
| Memory     | Low — one row at a time | Higher — all rows loaded   |
| Speed      | ⚡ Faster for reading    | Slightly slower (loads all) |
| Editing    | ❌ Read-only             | ✅ Can edit offline         |
| Connection | Open while reading       | Closed after Fill()         |
| Returns    | `List<T>`/ object      | `DataTable`/`DataSet`   |
| Best for   | Login, list display      | Kendo Grid, reports, forms  |

---

## ⭐ Interview Quick-Fire

| Question                                            | Answer                                                      |
| --------------------------------------------------- | ----------------------------------------------------------- |
| Connected READ uses which class?                    | `SqlDataReader`via `cmd.ExecuteReader()`                |
| Disconnected READ uses which class?                 | `SqlDataAdapter`via `da.Fill()`                         |
| `while (reader.Read())`vs `if (reader.Read())`? | `while`for multiple rows,`if`for single row             |
| Does `Fill()`need manual `con.Open()`?          | ❌ No — Fill opens and closes automatically                |
| How to read COUNT?                                  | `ExecuteScalar()`→ cast to `int`                       |
| Which is faster for read-only?                      | `SqlDataReader`— streams without loading all into memory |
