
# 10 — Stored Procedures with ADO.NET

---

## 🎯 One-Line Definition

> **A Stored Procedure is a named, pre-compiled block of SQL that lives in SQL Server — you call it from C# by name, pass parameters, and it executes — faster, safer, and more maintainable than inline SQL.**

---

## 🔷 What is a Stored Procedure?

A Stored Procedure (SP) is SQL code **saved inside SQL Server** with a name.
Instead of sending raw SQL from C#, you send just the  **procedure name** .

> 💡 Think of a Stored Procedure like a **function in SQL Server** — you define it once, call it by name whenever you need it, from any application.

---

## 🔷 Inline SQL vs Stored Procedure — The Difference

```
INLINE SQL (CommandType.Text):
  C# app sends:  "SELECT Id, Name FROM Employee WHERE Id = 5"
  SQL Server:    receives SQL → parses → compiles → executes → returns result
                 (parsing + compiling happens EVERY time)

STORED PROCEDURE (CommandType.StoredProcedure):
  SQL Server:    sp_GetEmployee already parsed + compiled → stored in cache
  C# app sends:  "sp_GetEmployee" + @Id = 5
  SQL Server:    finds compiled plan → executes → returns result
                 (no parsing/compiling — already done)
```

---

## 🔷 Why Use Stored Procedures?

```
✅ Performance        → Pre-compiled, execution plan cached
✅ Security          → Grant EXECUTE permission only — users never touch tables
✅ Maintainability   → SQL lives in DB, not scattered in C# code
✅ Reusability       → Same SP called from .NET, SSMS, Python, any client
✅ Separation        → Business logic stays in DB, C# stays thin
✅ Reduce network    → One call instead of sending a long SQL string
```

---

## 🔷 Step 1 — Create the Stored Procedure in SQL Server

### SELECT — Get All Employees

```sql
CREATE PROCEDURE sp_GetAllEmployees
AS
BEGIN
    SELECT Id, Name, Role, Email, Salary, HireDate
    FROM Employee
    ORDER BY Name
END
```

### SELECT — Get Employee by ID

```sql
CREATE PROCEDURE sp_GetEmployeeById
    @Id INT
AS
BEGIN
    SELECT Id, Name, Role, Email, Salary, HireDate
    FROM Employee
    WHERE Id = @Id
END
```

### INSERT — Add New Employee

```sql
CREATE PROCEDURE sp_InsertEmployee
    @Name     NVARCHAR(100),
    @Role     NVARCHAR(50),
    @Email    NVARCHAR(150),
    @Salary   DECIMAL(10,2),
    @HireDate DATETIME
AS
BEGIN
    INSERT INTO Employee (Name, Role, Email, Salary, HireDate)
    VALUES (@Name, @Role, @Email, @Salary, @HireDate)
END
```

### UPDATE — Update Employee

```sql
CREATE PROCEDURE sp_UpdateEmployee
    @Id       INT,
    @Name     NVARCHAR(100),
    @Role     NVARCHAR(50),
    @Salary   DECIMAL(10,2)
AS
BEGIN
    UPDATE Employee
    SET Name   = @Name,
        Role   = @Role,
        Salary = @Salary
    WHERE Id = @Id
END
```

### DELETE — Delete Employee

```sql
CREATE PROCEDURE sp_DeleteEmployee
    @Id INT
AS
BEGIN
    DELETE FROM Employee WHERE Id = @Id
END
```

---

## 🔷 Step 2 — Call Stored Procedures from C#

### Two mandatory requirements:

```
1. CommandText  = stored procedure name (just the name — no SQL)
2. CommandType  = CommandType.StoredProcedure  (MANDATORY — no default)
```

---

### Call SP that returns multiple rows (ExecuteReader)

```csharp
public List<Employee> GetAllEmployees()
{
    var employees = new List<Employee>();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetAllEmployees", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;  // ← required

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    employees.Add(new Employee
                    {
                        Id       = reader.GetInt32(0),
                        Name     = reader.GetString(1),
                        Role     = reader.GetString(2),
                        Email    = reader["Email"] as string,
                        Salary   = reader.GetDecimal(4),
                        HireDate = reader.GetDateTime(5)
                    });
                }
            }
        }
    }

    return employees;
}
```

---

### Call SP with input parameter (ExecuteReader)

```csharp
public Employee GetEmployeeById(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetEmployeeById", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                if (reader.Read())
                {
                    return new Employee
                    {
                        Id       = reader.GetInt32(0),
                        Name     = reader.GetString(1),
                        Role     = reader.GetString(2),
                        Salary   = reader.GetDecimal(4)
                    };
                }
                return null;
            }
        }
    }
}
```

---

### Call SP for INSERT (ExecuteNonQuery)

```csharp
public void InsertEmployee(Employee emp)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_InsertEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.Add("@Name",     SqlDbType.NVarChar, 100).Value = emp.Name;
            cmd.Parameters.Add("@Role",     SqlDbType.NVarChar, 50).Value  = emp.Role;
            cmd.Parameters.Add("@Email",    SqlDbType.NVarChar, 150).Value =
                                (object)emp.Email ?? DBNull.Value;
            cmd.Parameters.Add("@Salary",   SqlDbType.Decimal).Value       = emp.Salary;
            cmd.Parameters.Add("@HireDate", SqlDbType.DateTime).Value      = emp.HireDate;

            int rows = cmd.ExecuteNonQuery();
            Console.WriteLine($"✅ Inserted — {rows} row(s) affected.");
        }
    }
}
```

---

### Call SP for UPDATE (ExecuteNonQuery)

```csharp
public int UpdateEmployee(Employee emp)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_UpdateEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.AddWithValue("@Id",     emp.Id);
            cmd.Parameters.AddWithValue("@Name",   emp.Name);
            cmd.Parameters.AddWithValue("@Role",   emp.Role);
            cmd.Parameters.AddWithValue("@Salary", emp.Salary);

            return cmd.ExecuteNonQuery();
        }
    }
}
```

---

### Call SP for DELETE (ExecuteNonQuery)

```csharp
public int DeleteEmployee(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_DeleteEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.AddWithValue("@Id", id);

            return cmd.ExecuteNonQuery();
        }
    }
}
```

---

## 🔷 SP with Business Logic Inside

Stored procedures are powerful because they can contain complex SQL logic:

```sql
CREATE PROCEDURE sp_GetEmployeesByDepartmentWithStats
    @Department NVARCHAR(50)
AS
BEGIN
    -- Get employees
    SELECT Id, Name, Role, Salary
    FROM Employee
    WHERE Department = @Department
    ORDER BY Salary DESC

    -- Also return department summary
    SELECT COUNT(*)      AS TotalCount,
           AVG(Salary)   AS AvgSalary,
           MAX(Salary)   AS MaxSalary,
           MIN(Salary)   AS MinSalary
    FROM Employee
    WHERE Department = @Department
END
```

```csharp
// This SP returns TWO result sets — handled with NextResult()
public void GetDeptStats(string department)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(
            "sp_GetEmployeesByDepartmentWithStats", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Department", department);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                // First result set — employee list
                Console.WriteLine($"── Employees in {department} ──");
                while (reader.Read())
                    Console.WriteLine($"{reader["Name"]}  {reader["Salary"]}");

                // Second result set — department stats
                reader.NextResult();
                if (reader.Read())
                {
                    Console.WriteLine($"Count: {reader["TotalCount"]}");
                    Console.WriteLine($"Avg:   {reader["AvgSalary"]}");
                    Console.WriteLine($"Max:   {reader["MaxSalary"]}");
                }
            }
        }
    }
}
```

---

## 🔷 Inline SQL vs Stored Procedure — Full Comparison

|                       | Inline SQL (`CommandType.Text`) | Stored Procedure               |
| --------------------- | --------------------------------- | ------------------------------ |
| SQL lives in          | C# source code                    | SQL Server database            |
| Pre-compiled          | ❌ Compiled every call            | ✅ Compiled once, cached       |
| Security              | Table-level permissions           | Execute permission only        |
| Reusable              | ❌ One app only                   | ✅ Any client/app              |
| Maintainability       | Hard — in C# strings             | Easy — edit in SSMS           |
| Best for              | Simple/dynamic queries            | Business logic, production     |
| `CommandType`needed | `Text`(default)                 | `StoredProcedure`(mandatory) |

---

## ⚠️ Common Mistakes

| Mistake                                    | What Happens                                  | Fix                                              |
| ------------------------------------------ | --------------------------------------------- | ------------------------------------------------ |
| Forgetting `CommandType.StoredProcedure` | SP name treated as SQL → SqlException        | Always set CommandType                           |
| Using SQL keywords in CommandText for SP   | Error — CommandText must be just the name    | `cmd.CommandText = "sp_Name"`only              |
| Wrong parameter name                       | `SqlException`— parameter not found        | `@Name`in C# must exactly match `@Name`in SP |
| Wrong number of parameters                 | `SqlException`— SP expects different count | Check SP signature in SSMS                       |

---

## ⭐ Interview Quick-Fire

| Question                                        | Answer                                                      |
| ----------------------------------------------- | ----------------------------------------------------------- |
| What is a stored procedure?                     | Pre-compiled SQL block stored in SQL Server, called by name |
| Why use SPs over inline SQL?                    | Pre-compiled (faster), more secure, reusable, maintainable  |
| What goes in CommandText for SP?                | Just the SP name — no SQL                                  |
| Is CommandType mandatory for SP?                | ✅ Yes — must set `CommandType.StoredProcedure`          |
| Which execute method for SP that INSERTs?       | `ExecuteNonQuery()`                                       |
| Which execute method for SP that SELECTs?       | `ExecuteReader()`                                         |
| Which execute method for SP that returns COUNT? | `ExecuteScalar()`                                         |
| Can an SP return multiple result sets?          | ✅ Yes — use `reader.NextResult()`to advance             |
