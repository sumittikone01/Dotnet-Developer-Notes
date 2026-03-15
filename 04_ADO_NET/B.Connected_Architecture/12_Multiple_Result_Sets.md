
# 12 — Multiple Result Sets

---

## 🎯 One-Line Definition

> **A single `SqlCommand` can return multiple SELECT result sets — `reader.NextResult()` advances from one result set to the next, letting you fetch several tables of data in a single database round-trip.**

---

## 🔷 What are Multiple Result Sets?

When a stored procedure or SQL batch contains  **more than one SELECT** , it produces multiple result sets — one after the other in the same response stream.

> 💡 Think of it like ordering a meal — instead of three separate trips to the kitchen (three separate DB calls), you get the starter, main, and dessert all on one tray (one DB call, three result sets).

---

## 🔷 When to Use Multiple Result Sets

```
✅ Load a form that needs data from several tables
   (e.g. employee details + their department + their manager)

✅ Dashboard with multiple metrics in one call
   (e.g. total employees + recent hires + salary stats)

✅ Reduce round-trips to the database
   (one call instead of three = faster page load)

✅ Related data needed together
   (e.g. order header + order lines)
```

---

## 🔷 How NextResult() Works

```
SQL returns stream:
  [Result Set 1: Employees] → [Result Set 2: Departments] → [Result Set 3: Stats]

reader starts on:  Result Set 1
reader.Read()      → reads rows from Result Set 1

reader.NextResult() → moves to Result Set 2 (returns true if there is one)
reader.Read()       → reads rows from Result Set 2

reader.NextResult() → moves to Result Set 3 (returns true)
reader.Read()       → reads rows from Result Set 3

reader.NextResult() → returns false — no more result sets
```

---

## 🔷 Step 1 — SQL / Stored Procedure

### Inline SQL batch — two SELECTs

```sql
SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name;
SELECT Id, Name FROM Department ORDER BY Name;
```

### Stored Procedure — multiple SELECTs

```sql
CREATE PROCEDURE sp_GetFormData
    @EmployeeId INT
AS
BEGIN
    -- Result Set 1: Employee details
    SELECT Id, Name, Role, Email, Salary, HireDate
    FROM Employee
    WHERE Id = @EmployeeId

    -- Result Set 2: All departments (for dropdown)
    SELECT Id, Name
    FROM Department
    ORDER BY Name

    -- Result Set 3: Employee's project assignments
    SELECT p.Id, p.Title, p.Status, p.StartDate
    FROM Projects p
    INNER JOIN EmployeeProjects ep ON p.Id = ep.ProjectId
    WHERE ep.EmployeeId = @EmployeeId
END
```

---

## 🔷 Step 2 — Read Multiple Result Sets in C#

### Basic pattern — two result sets

```csharp
string sql = @"SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name;
               SELECT Id, Name FROM Department ORDER BY Name;";

using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    using (SqlCommand cmd = new SqlCommand(sql, con))
    {
        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            // ── Result Set 1: Employees ───────────────────────────
            var employees = new List<Employee>();
            while (reader.Read())
            {
                employees.Add(new Employee
                {
                    Id     = reader.GetInt32(0),
                    Name   = reader.GetString(1),
                    Role   = reader.GetString(2),
                    Salary = reader.GetDecimal(3)
                });
            }

            // ── Move to Result Set 2 ──────────────────────────────
            reader.NextResult();

            // ── Result Set 2: Departments ─────────────────────────
            var departments = new List<Department>();
            while (reader.Read())
            {
                departments.Add(new Department
                {
                    Id   = reader.GetInt32(0),
                    Name = reader.GetString(1)
                });
            }

            Console.WriteLine($"Employees:   {employees.Count}");
            Console.WriteLine($"Departments: {departments.Count}");
        }
    }
}
```

---

### With Stored Procedure — three result sets

```csharp
public (Employee emp, List<Department> depts, List<Project> projects)
    GetFormData(int employeeId)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetFormData", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@EmployeeId", employeeId);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                // ── Result Set 1: Employee ────────────────────────
                Employee emp = null;
                if (reader.Read())
                {
                    emp = new Employee
                    {
                        Id       = reader.GetInt32(reader.GetOrdinal("Id")),
                        Name     = reader.GetString(reader.GetOrdinal("Name")),
                        Role     = reader.GetString(reader.GetOrdinal("Role")),
                        Email    = reader["Email"] as string,
                        Salary   = reader.GetDecimal(reader.GetOrdinal("Salary")),
                        HireDate = reader.GetDateTime(reader.GetOrdinal("HireDate"))
                    };
                }

                // ── Result Set 2: Departments ─────────────────────
                reader.NextResult();
                var depts = new List<Department>();
                while (reader.Read())
                {
                    depts.Add(new Department
                    {
                        Id   = reader.GetInt32(0),
                        Name = reader.GetString(1)
                    });
                }

                // ── Result Set 3: Projects ────────────────────────
                reader.NextResult();
                var projects = new List<Project>();
                while (reader.Read())
                {
                    projects.Add(new Project
                    {
                        Id        = reader.GetInt32(reader.GetOrdinal("Id")),
                        Title     = reader.GetString(reader.GetOrdinal("Title")),
                        Status    = reader.GetString(reader.GetOrdinal("Status")),
                        StartDate = reader.GetDateTime(reader.GetOrdinal("StartDate"))
                    });
                }

                return (emp, depts, projects);
            }
        }
    }
}
```

---

## 🔷 Check for Result Set Existence — `NextResult()` Return Value

`NextResult()` returns:

* `true` — another result set is available
* `false` — no more result sets

```csharp
using (SqlDataReader reader = cmd.ExecuteReader())
{
    // Result set 1
    while (reader.Read()) { ... }

    // Check if result set 2 exists before reading
    if (reader.NextResult())
    {
        while (reader.Read()) { ... }   // result set 2
    }

    // Check for result set 3
    if (reader.NextResult())
    {
        while (reader.Read()) { ... }   // result set 3
    }
}
```

---

## 🔷 Real-World — Dashboard with Stats + List

A common MVC pattern: one SP loads everything the page needs:

```sql
CREATE PROCEDURE sp_GetEmployeeDashboard
AS
BEGIN
    -- Result Set 1: Summary stats
    SELECT
        COUNT(*)         AS TotalEmployees,
        AVG(Salary)      AS AvgSalary,
        MAX(Salary)      AS MaxSalary,
        MIN(Salary)      AS MinSalary,
        SUM(Salary)      AS TotalPayroll
    FROM Employee

    -- Result Set 2: Recent hires (last 30 days)
    SELECT Id, Name, Role, HireDate
    FROM Employee
    WHERE HireDate >= DATEADD(DAY, -30, GETDATE())
    ORDER BY HireDate DESC

    -- Result Set 3: Top 5 earners
    SELECT TOP 5 Id, Name, Role, Salary
    FROM Employee
    ORDER BY Salary DESC
END
```

```csharp
public DashboardViewModel GetDashboard()
{
    var vm = new DashboardViewModel();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeDashboard", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                // ── Stats (single row) ────────────────────────────
                if (reader.Read())
                {
                    vm.TotalEmployees = (int)reader["TotalEmployees"];
                    vm.AvgSalary      = Convert.ToDecimal(reader["AvgSalary"]);
                    vm.MaxSalary      = Convert.ToDecimal(reader["MaxSalary"]);
                    vm.TotalPayroll   = Convert.ToDecimal(reader["TotalPayroll"]);
                }

                // ── Recent hires ──────────────────────────────────
                reader.NextResult();
                vm.RecentHires = new List<Employee>();
                while (reader.Read())
                {
                    vm.RecentHires.Add(new Employee
                    {
                        Id       = reader.GetInt32(0),
                        Name     = reader.GetString(1),
                        Role     = reader.GetString(2),
                        HireDate = reader.GetDateTime(3)
                    });
                }

                // ── Top earners ───────────────────────────────────
                reader.NextResult();
                vm.TopEarners = new List<Employee>();
                while (reader.Read())
                {
                    vm.TopEarners.Add(new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Salary = reader.GetDecimal(3)
                    });
                }
            }
        }
    }

    return vm;
}
```

---

## 🔷 Multiple Result Sets vs Multiple Calls

```
THREE SEPARATE CALLS:
  Call 1: SqlConnection.Open → Execute → Read → Close
  Call 2: SqlConnection.Open → Execute → Read → Close
  Call 3: SqlConnection.Open → Execute → Read → Close
  = 3 round-trips to the database
  = 3 connections opened/closed
  = slower

ONE CALL WITH MULTIPLE RESULT SETS:
  Call 1: SqlConnection.Open → Execute → Read RS1 → NextResult → Read RS2 → NextResult → Read RS3 → Close
  = 1 round-trip to the database
  = 1 connection opened/closed
  = faster ✅
```

---

## 🔷 `NextResult()` — Summary

| State                             | `NextResult()`  | What to Do                              |
| --------------------------------- | ----------------- | --------------------------------------- |
| More result sets exist            | Returns `true`  | Read with `while (reader.Read())`     |
| No more result sets               | Returns `false` | Stop — no more data                    |
| Called before reading current set | Skips current set | Always finish reading current set first |

---

## ⚠️ Common Mistakes

| Mistake                                              | What Happens                                    | Fix                                                      |
| ---------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------- |
| Forgetting `reader.NextResult()`                   | Only first result set read                      | Always call `NextResult()`between sets                 |
| Calling `NextResult()`before finishing current set | Current rows skipped                            | Read all rows of current set before `NextResult()`     |
| Wrong column names for different result sets         | `IndexOutOfRangeException`                    | Each result set has its own schema — use `GetOrdinal` |
| Not checking `NextResult()`return value            | NullRef if SP returned fewer sets than expected | Use `if (reader.NextResult())`                         |

---

## ⭐ Interview Quick-Fire

| Question                                           | Answer                                                               |
| -------------------------------------------------- | -------------------------------------------------------------------- |
| What are multiple result sets?                     | Multiple SELECT results returned from one command, read sequentially |
| Method to advance to next result set?              | `reader.NextResult()`                                              |
| What does `NextResult()`return?                  | `true`if another result set exists,`false`if none                |
| Benefit over multiple DB calls?                    | One round-trip instead of many — faster, fewer connections          |
| Can inline SQL return multiple result sets?        | ✅ Yes — multiple SELECTs separated by `;`                        |
| Can stored procedures return multiple result sets? | ✅ Yes — multiple SELECT statements inside the SP                   |
| When should you use this?                          | Form load (entity + dropdown data), dashboard (stats + lists)        |
