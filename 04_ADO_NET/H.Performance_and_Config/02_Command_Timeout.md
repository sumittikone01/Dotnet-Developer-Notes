
# 02 — Command Timeout

---

## 🎯 One-Line Definition

> **`CommandTimeout` sets how many seconds ADO.NET waits for a SQL command to finish before giving up — if the query takes longer, a `SqlException` is thrown, preventing your app from hanging forever on a slow query.**

---

## 🔷 The Problem Without a Timeout

```
WITHOUT timeout:
──────────────────────────────────────────────────────────────────
User clicks "Generate Annual Report"
App runs: SELECT * FROM Orders JOIN ... (heavy query — 5 minutes)

Thread is blocked for 5 minutes
User's browser shows spinner for 5 minutes
100 users click the same button = 100 blocked threads
Server runs out of threads → entire app unresponsive

WITH timeout:
──────────────────────────────────────────────────────────────────
User clicks "Generate Annual Report"
App runs: SELECT * FROM Orders JOIN ...
After 30 seconds → SqlException: "Timeout expired"
App catches it → returns friendly error message
Thread freed → other requests served normally
```

---

## 🔷 Two Different Timeouts — Know the Difference

```
┌────────────────────────────────────────────────────────────────┐
│  Connection Timeout                                            │
│  → Set in the CONNECTION STRING                                │
│  → How long to WAIT to OPEN a connection to SQL Server        │
│  → Default: 15 seconds                                        │
│  → Keyword: Connect Timeout=15                                 │
│                                                                │
│  Command Timeout                                               │
│  → Set on the SQLCOMMAND OBJECT                                │
│  → How long to WAIT for a QUERY to FINISH executing           │
│  → Default: 30 seconds                                        │
│  → Property: cmd.CommandTimeout = 30                          │
└────────────────────────────────────────────────────────────────┘
```

|                  | Connection Timeout      | Command Timeout             |
| ---------------- | ----------------------- | --------------------------- |
| Controls         | Time to open connection | Time for query to execute   |
| Set on           | Connection string       | `SqlCommand`object        |
| Default          | 15 seconds              | 30 seconds                  |
| Exception        | `SqlException`        | `SqlException`            |
| Keyword/Property | `Connect Timeout=15`  | `cmd.CommandTimeout = 30` |

---

## 🔷 Setting CommandTimeout

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
    {
        // Default is 30 seconds — change it:
        cmd.CommandTimeout = 60;   // wait up to 60 seconds
        //                    ↑
        //                  seconds (not milliseconds)

        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read()) { ... }
        }
    }
}
```

---

## 🔷 CommandTimeout Values and When to Use Each

```csharp
// 0 = No timeout — waits forever (never recommended in production)
cmd.CommandTimeout = 0;

// Quick reads — short timeout
cmd.CommandTimeout = 10;    // login check, dropdown data, single record lookup

// Standard operations — default works fine
cmd.CommandTimeout = 30;    // default — most CRUD operations

// Reports and heavy queries — longer timeout
cmd.CommandTimeout = 120;   // 2 minutes for complex report queries

// Bulk operations — very long timeout
cmd.CommandTimeout = 300;   // 5 minutes for large data migrations
```

---

## 🔷 Setting Timeout Per Operation Type

```csharp
public class EmployeeDAL
{
    private readonly string _cs;
    public EmployeeDAL(IConfiguration config)
        => _cs = config.GetConnectionString("DefaultConnection");

    // ── Quick read — short timeout ────────────────────────────────
    public Employee GetById(int id)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "SELECT * FROM Employee WHERE Id = @Id", con))
            {
                cmd.CommandTimeout = 10;   // simple lookup — 10s is plenty
                cmd.Parameters.AddWithValue("@Id", id);

                using (SqlDataReader reader = cmd.ExecuteReader())
                {
                    if (reader.Read()) return MapReader(reader);
                }
            }
        }
        return null;
    }

    // ── Standard CRUD — default timeout ───────────────────────────
    public int Insert(Employee emp)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "INSERT INTO Employee(Name,Role,Salary) VALUES(@N,@R,@S)", con))
            {
                // cmd.CommandTimeout = 30;  ← default — not needed to set explicitly
                cmd.Parameters.AddWithValue("@N", emp.Name);
                cmd.Parameters.AddWithValue("@R", emp.Role);
                cmd.Parameters.AddWithValue("@S", emp.Salary);
                return cmd.ExecuteNonQuery();
            }
        }
    }

    // ── Heavy report query — longer timeout ───────────────────────
    public DataTable GetAnnualReport(int year)
    {
        string sql = @"SELECT e.Name, e.Department,
                              SUM(p.Amount) AS TotalPayroll,
                              COUNT(a.Id)   AS Attendance
                       FROM Employee e
                       LEFT JOIN Payroll p ON p.EmpId = e.Id AND YEAR(p.Date) = @Year
                       LEFT JOIN Attendance a ON a.EmpId = e.Id AND YEAR(a.Date) = @Year
                       GROUP BY e.Name, e.Department
                       ORDER BY e.Department, TotalPayroll DESC";

        DataTable dt = new DataTable();
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.CommandTimeout = 120;  // ← 2 minutes for heavy report
                cmd.Parameters.AddWithValue("@Year", year);

                using (SqlDataAdapter da = new SqlDataAdapter(cmd))
                {
                    da.Fill(dt);
                }
            }
        }
        return dt;
    }

    // ── Bulk migration — very long timeout ────────────────────────
    public void ArchiveOldRecords(int beforeYear)
    {
        string sql = @"INSERT INTO EmployeeArchive
                       SELECT * FROM Employee
                       WHERE YEAR(HireDate) < @Year;
                       DELETE FROM Employee
                       WHERE YEAR(HireDate) < @Year";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.CommandTimeout = 300;  // ← 5 minutes for bulk operation
                cmd.Parameters.AddWithValue("@Year", beforeYear);
                cmd.ExecuteNonQuery();
            }
        }
    }
}
```

---

## 🔷 Handling the Timeout Exception

```csharp
public List<Employee> GetAll()
{
    var list = new List<Employee>();

    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
            {
                cmd.CommandTimeout = 30;

                using (SqlDataReader reader = cmd.ExecuteReader())
                {
                    while (reader.Read())
                        list.Add(MapReader(reader));
                }
            }
        }
    }
    catch (SqlException ex) when (ex.Number == -2)
    {
        // SqlException.Number = -2 specifically means timeout
        Console.Error.WriteLine("Query timed out — query took too long.");
        throw new TimeoutException("Database query timed out. Please try again.", ex);
    }
    catch (SqlException ex)
    {
        // Other SQL errors
        Console.Error.WriteLine($"SQL Error {ex.Number}: {ex.Message}");
        throw;
    }

    return list;
}
```

---

## 🔷 Connection Timeout — Set in Connection String

```csharp
// Connection timeout — in the connection string
string cs = @"Server=.\SQLEXPRESS;
              Database=EmployeeDB;
              Trusted_Connection=True;
              TrustServerCertificate=True;
              Connect Timeout=30;";   // ← wait 30s to open the connection

using (SqlConnection con = new SqlConnection(cs))
{
    // If SQL Server doesn't respond in 30s → SqlException
    con.Open();
}

// Can also read it:
Console.WriteLine(con.ConnectionTimeout);   // 30
```

---

## 🔷 Timeout Quick Reference

| Operation                            | Recommended Timeout   |
| ------------------------------------ | --------------------- |
| Login / authentication check         | 10 seconds            |
| Single record lookup by ID           | 10 seconds            |
| Standard CRUD (Insert/Update/Delete) | 30 seconds (default)  |
| List / grid data load                | 30–60 seconds        |
| Complex report with JOINs            | 60–120 seconds       |
| Bulk insert / data migration         | 300+ seconds or `0` |
| Stored procedure (known fast)        | 15–30 seconds        |

---

## ⚠️ Common Mistakes

| Mistake                                           | What Happens                                                           | Fix                                                          |
| ------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------ |
| `CommandTimeout = 0`in production               | App hangs forever on slow query — thread never freed                  | Always set a reasonable timeout                              |
| Same timeout for all queries                      | Reports always time out OR simple queries given wastefully long window | Set per-operation based on expected duration                 |
| Catching timeout without checking `ex.Number`   | Treats all SQL errors as timeouts                                      | Check `ex.Number == -2`for timeout specifically            |
| Confusing Connection Timeout with Command Timeout | Wrong property tuned, problem persists                                 | Connection = connection string. Command = cmd.CommandTimeout |

---

## ⭐ Interview Quick-Fire

| Question                                           | Answer                                                                                                                                         |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| What is `CommandTimeout`?                        | Seconds ADO.NET waits for a query to complete before throwing exception                                                                        |
| Default `CommandTimeout`value?                   | 30 seconds                                                                                                                                     |
| What exception is thrown on timeout?               | `SqlException`with `Number = -2`                                                                                                           |
| How do you set `CommandTimeout`?                 | `cmd.CommandTimeout = 60`— on the `SqlCommand`object                                                                                      |
| What is the difference from `ConnectionTimeout`? | `ConnectionTimeout`= waiting to open the connection (in connection string).`CommandTimeout`= waiting for a query to finish (on SqlCommand) |
| Should you ever set `CommandTimeout = 0`?        | Only for known bulk operations where time is unpredictable — never for normal queries                                                         |
