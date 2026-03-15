
# 09 — SqlParameters & Parameterized Queries

---

## 🎯 One-Line Definition

> **`SqlParameter` passes values safely into your SQL query using placeholders like `@Name` — it completely prevents SQL injection and is the only correct way to include user input in database commands.**

---

## 🔷 The Problem — SQL Injection

Without parameters, you might build SQL by concatenating strings:

```csharp
// ❌ DANGEROUS — never do this
string name = userInput;   // user enters: ' OR '1'='1
string sql  = "SELECT * FROM Employee WHERE Name = '" + name + "'";

// The query becomes:
// SELECT * FROM Employee WHERE Name = '' OR '1'='1'
// Returns ALL employees — attacker bypassed your filter!

// Even worse — destructive injection:
// user enters: '; DROP TABLE Employee; --
// SELECT * FROM Employee WHERE Name = ''; DROP TABLE Employee; --
// Your Employee table is GONE
```

With parameters:

```csharp
// ✅ SAFE — parameter value is NEVER interpreted as SQL
string sql = "SELECT * FROM Employee WHERE Name = @Name";
cmd.Parameters.AddWithValue("@Name", userInput);
// @Name is sent as data, not SQL — injection impossible
```

---

## 🔷 How Parameters Work Under the Hood

```
WITHOUT parameters:
  You send: "SELECT * FROM Employee WHERE Name = 'O''Reilly'"
  SQL Server: parses the whole string as one SQL statement
  Risk: injected SQL runs as commands

WITH parameters:
  You send: "SELECT * FROM Employee WHERE Name = @Name"  (query)
  You send: @Name = "O'Reilly"                           (value, separately)
  SQL Server: query compiled first, value inserted as DATA — never parsed as SQL
  Result: ZERO injection risk, PLUS query plan caching
```

---

## 🔷 Three Ways to Add Parameters

### Method 1 — `AddWithValue` (quickest, most common)

```csharp
cmd.Parameters.AddWithValue("@Name",   "Alice");
cmd.Parameters.AddWithValue("@Salary", 65000);
cmd.Parameters.AddWithValue("@Id",     5);
```

> ⚠️ `AddWithValue` infers the SQL type from the C# type. Usually fine, but can cause type mismatch issues with dates or decimals — use Method 2 for precision.

---

### Method 2 — `Add` with explicit SqlDbType (most precise)

```csharp
cmd.Parameters.Add("@Name",    SqlDbType.NVarChar, 100).Value = "Alice";
cmd.Parameters.Add("@Salary",  SqlDbType.Decimal).Value       = 65000m;
cmd.Parameters.Add("@HireDate",SqlDbType.DateTime).Value      = DateTime.Today;
cmd.Parameters.Add("@IsActive",SqlDbType.Bit).Value           = true;
cmd.Parameters.Add("@Id",      SqlDbType.Int).Value           = 5;
```

---

### Method 3 — Create `SqlParameter` object manually (most control)

```csharp
SqlParameter param = new SqlParameter();
param.ParameterName = "@Name";
param.SqlDbType     = SqlDbType.NVarChar;
param.Size          = 100;
param.Value         = "Alice";

cmd.Parameters.Add(param);
```

---

## 🔷 SqlDbType — C# to SQL Type Mapping

| C# Type      | `SqlDbType`                  | SQL Server Type  |
| ------------ | ------------------------------ | ---------------- |
| `int`      | `SqlDbType.Int`              | INT              |
| `long`     | `SqlDbType.BigInt`           | BIGINT           |
| `string`   | `SqlDbType.NVarChar`         | NVARCHAR         |
| `string`   | `SqlDbType.VarChar`          | VARCHAR          |
| `decimal`  | `SqlDbType.Decimal`          | DECIMAL, MONEY   |
| `double`   | `SqlDbType.Float`            | FLOAT            |
| `bool`     | `SqlDbType.Bit`              | BIT              |
| `DateTime` | `SqlDbType.DateTime`         | DATETIME         |
| `DateTime` | `SqlDbType.Date`             | DATE             |
| `Guid`     | `SqlDbType.UniqueIdentifier` | UNIQUEIDENTIFIER |
| `byte[]`   | `SqlDbType.VarBinary`        | VARBINARY        |

---

## 🔷 Handling NULL Parameters

```csharp
// ❌ WRONG — C# null does not become DB NULL automatically
cmd.Parameters.AddWithValue("@Email", null);
// Throws ArgumentNullException

// ✅ CORRECT — use DBNull.Value for database NULLs
string email = null;
cmd.Parameters.AddWithValue("@Email", (object)email ?? DBNull.Value);

// ✅ Alternative — explicit check
cmd.Parameters.AddWithValue("@Phone",
    string.IsNullOrEmpty(phone) ? (object)DBNull.Value : phone);

// ✅ With Add method
cmd.Parameters.Add("@ManagerId", SqlDbType.Int).Value =
    managerId.HasValue ? (object)managerId.Value : DBNull.Value;
```

---

## 🔷 Complete CRUD with Parameters

```csharp
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // ── INSERT ────────────────────────────────────────────────────
    public int Insert(Employee emp)
    {
        string sql = @"INSERT INTO Employee (Name, Role, Email, Salary, HireDate, IsActive)
                       VALUES (@Name, @Role, @Email, @Salary, @HireDate, @IsActive);
                       SELECT SCOPE_IDENTITY();";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.Add("@Name",     SqlDbType.NVarChar, 100).Value = emp.Name;
                cmd.Parameters.Add("@Role",     SqlDbType.NVarChar, 50).Value  = emp.Role;
                cmd.Parameters.Add("@Email",    SqlDbType.NVarChar, 150).Value =
                                    (object)emp.Email ?? DBNull.Value;
                cmd.Parameters.Add("@Salary",   SqlDbType.Decimal).Value       = emp.Salary;
                cmd.Parameters.Add("@HireDate", SqlDbType.DateTime).Value      = emp.HireDate;
                cmd.Parameters.Add("@IsActive", SqlDbType.Bit).Value           = emp.IsActive;

                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }
    }

    // ── SELECT BY ID ──────────────────────────────────────────────
    public Employee GetById(int id)
    {
        string sql = "SELECT Id, Name, Role, Salary FROM Employee WHERE Id = @Id";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Id", id);

                using (SqlDataReader reader = cmd.ExecuteReader())
                {
                    if (reader.Read())
                    {
                        return new Employee
                        {
                            Id     = reader.GetInt32(0),
                            Name   = reader.GetString(1),
                            Role   = reader.GetString(2),
                            Salary = reader.GetDecimal(3)
                        };
                    }
                    return null;
                }
            }
        }
    }

    // ── UPDATE ────────────────────────────────────────────────────
    public int Update(Employee emp)
    {
        string sql = @"UPDATE Employee
                       SET Name = @Name, Role = @Role, Salary = @Salary
                       WHERE Id = @Id";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Name",   emp.Name);
                cmd.Parameters.AddWithValue("@Role",   emp.Role);
                cmd.Parameters.AddWithValue("@Salary", emp.Salary);
                cmd.Parameters.AddWithValue("@Id",     emp.Id);

                return cmd.ExecuteNonQuery();
            }
        }
    }

    // ── DELETE ────────────────────────────────────────────────────
    public int Delete(int id)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "DELETE FROM Employee WHERE Id = @Id", con))
            {
                cmd.Parameters.AddWithValue("@Id", id);
                return cmd.ExecuteNonQuery();
            }
        }
    }
}
```

---

## 🔷 Search with Multiple Optional Filters

A real pattern — dynamic search where filters are optional:

```csharp
public List<Employee> Search(string name, string department, decimal? minSalary)
{
    string sql = @"SELECT Id, Name, Department, Salary FROM Employee
                   WHERE (@Name IS NULL OR Name LIKE '%' + @Name + '%')
                     AND (@Dept IS NULL OR Department = @Dept)
                     AND (@MinSalary IS NULL OR Salary >= @MinSalary)";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",
                string.IsNullOrEmpty(name) ? (object)DBNull.Value : name);

            cmd.Parameters.AddWithValue("@Dept",
                string.IsNullOrEmpty(department) ? (object)DBNull.Value : department);

            cmd.Parameters.AddWithValue("@MinSalary",
                minSalary.HasValue ? (object)minSalary.Value : DBNull.Value);

            var list = new List<Employee>();
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    list.Add(new Employee
                    {
                        Id         = reader.GetInt32(0),
                        Name       = reader.GetString(1),
                        Department = reader.GetString(2),
                        Salary     = reader.GetDecimal(3)
                    });
                }
            }
            return list;
        }
    }
}
```

---

## 🔷 AddWithValue vs Add — When to Use Which

|                | `AddWithValue`                 | `Add`with SqlDbType                  |
| -------------- | -------------------------------- | -------------------------------------- |
| Code length    | Shorter                          | Longer                                 |
| Type inference | Automatic (can cause mismatches) | Explicit — always correct             |
| For strings    | ⚠️ May infer wrong length      | ✅ Specify exact length                |
| For dates      | ⚠️ Can cause type mismatch     | ✅ Use `SqlDbType.DateTime`          |
| Best for       | Quick int/bool parameters        | Strings, dates, decimals in production |

---

## ⚠️ Common Mistakes

| Mistake                                    | What Happens                                | Fix                                           |
| ------------------------------------------ | ------------------------------------------- | --------------------------------------------- |
| String concatenation instead of `@param` | SQL injection vulnerability                 | Always use parameters                         |
| Passing C#`null`directly                 | `ArgumentNullException`                   | Use `(object)value ?? DBNull.Value`         |
| Wrong `@`prefix                          | Parameter not found at execution            | Name must start with `@`exactly             |
| Parameter name mismatch                    | `SqlException`at runtime                  | `@Name`in SQL must match `"@Name"`in code |
| Adding same parameter twice                | `ArgumentException`— duplicate parameter | Clear parameters or reuse the existing one    |

---

## ⭐ Interview Quick-Fire

| Question                                                       | Answer                                                                        |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| What is a SqlParameter?                                        | Class that safely passes values into SQL to prevent injection                 |
| Why use parameters instead of string concat?                   | Prevents SQL injection — value is sent as data, never as SQL                 |
| Three ways to add parameters?                                  | `AddWithValue`,`Add`with SqlDbType, manual `SqlParameter`object         |
| How to pass NULL to a parameter?                               | `(object)value ?? DBNull.Value`or `DBNull.Value`directly                  |
| What is `DBNull.Value`?                                      | Represents SQL NULL in C# — not the same as C#`null`                       |
| Is `AddWithValue`always safe?                                | Safe from injection, but may infer wrong SQL type — use `Add`for precision |
| Can you reuse a `SqlCommand`with different parameter values? | ✅ Yes — change `cmd.Parameters["@Id"].Value = newId`and execute again     |
