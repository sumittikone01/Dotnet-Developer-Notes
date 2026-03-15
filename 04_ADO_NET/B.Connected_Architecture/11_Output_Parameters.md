
# 11 — Output Parameters

---

## 🎯 One-Line Definition

> **Output parameters let a stored procedure send values BACK to C# — the SP sets them during execution and C# reads them after the command runs, without needing a SELECT statement.**

---

## 🔷 What are Output Parameters?

Normal parameters flow **one way** — from C# into SQL.
Output parameters flow **both ways** — C# passes values in, SQL sets values on the way out.

```
NORMAL INPUT PARAMETER:
  C# ──→ @Id = 5 ──→ SQL Server
         (only goes in)

OUTPUT PARAMETER:
  C# ──→ @NewId ──→ SQL Server (sets it during execution)
         @NewId ←── C# reads after execution
         (goes in empty, comes back with a value)

INPUT/OUTPUT PARAMETER:
  C# ──→ @Count = 0 ──→ SQL Server (increments it)
         @Count ←── 47  C# reads after execution
         (goes in with value, comes back modified)
```

---

## 🔷 Three Parameter Directions

| Direction       | `ParameterDirection`             | Flow                  |
| --------------- | ---------------------------------- | --------------------- |
| Input (default) | `ParameterDirection.Input`       | C# → SQL only        |
| Output          | `ParameterDirection.Output`      | SQL → C# only        |
| InputOutput     | `ParameterDirection.InputOutput` | C# → SQL → C#       |
| ReturnValue     | `ParameterDirection.ReturnValue` | SP RETURN value → C# |

---

## 🔷 When to Use Output Parameters

```
✅ Get the new ID after INSERT (alternative to SCOPE_IDENTITY())
✅ Get a count, total, or calculation result from an SP
✅ Get a status code or message from an SP
✅ Return multiple scalar values without a SELECT
✅ Confirm success/failure with a message
```

---

## 🔷 Step 1 — Create SP with Output Parameter in SQL Server

### Example 1 — INSERT and return new ID

```sql
CREATE PROCEDURE sp_InsertEmployee_WithId
    @Name     NVARCHAR(100),
    @Role     NVARCHAR(50),
    @Salary   DECIMAL(10,2),
    @NewId    INT OUTPUT        -- ← OUTPUT keyword marks it
AS
BEGIN
    INSERT INTO Employee (Name, Role, Salary)
    VALUES (@Name, @Role, @Salary)

    SET @NewId = SCOPE_IDENTITY()   -- ← SP sets the output param
END
```

### Example 2 — Get count by department

```sql
CREATE PROCEDURE sp_GetDeptCount
    @Department  NVARCHAR(50),
    @Count       INT OUTPUT         -- ← OUTPUT keyword
AS
BEGIN
    SELECT @Count = COUNT(*)
    FROM Employee
    WHERE Department = @Department
END
```

### Example 3 — Insert with status message

```sql
CREATE PROCEDURE sp_InsertWithStatus
    @Name      NVARCHAR(100),
    @Email     NVARCHAR(150),
    @Message   NVARCHAR(200) OUTPUT,
    @Success   BIT          OUTPUT
AS
BEGIN
    -- Check if email already exists
    IF EXISTS (SELECT 1 FROM Employee WHERE Email = @Email)
    BEGIN
        SET @Message = 'Email already exists'
        SET @Success = 0
        RETURN
    END

    INSERT INTO Employee (Name, Email)
    VALUES (@Name, @Email)

    SET @Message = 'Employee inserted successfully'
    SET @Success = 1
END
```

---

## 🔷 Step 2 — Read Output Parameters in C#

### Pattern — Four steps every time

```
1. Add parameter with Direction = ParameterDirection.Output
2. Execute the command (ExecuteNonQuery or ExecuteReader)
3. AFTER execution — read the parameter value from cmd.Parameters
4. Cast to correct type
```

---

### Example 1 — Get new ID after INSERT

```csharp
public int InsertEmployeeGetId(string name, string role, decimal salary)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_InsertEmployee_WithId", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            // ── Input parameters ──────────────────────────────────
            cmd.Parameters.Add("@Name",   SqlDbType.NVarChar, 100).Value = name;
            cmd.Parameters.Add("@Role",   SqlDbType.NVarChar, 50).Value  = role;
            cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value       = salary;

            // ── Output parameter ──────────────────────────────────
            SqlParameter outputParam = new SqlParameter("@NewId", SqlDbType.Int);
            outputParam.Direction = ParameterDirection.Output;  // ← key line
            cmd.Parameters.Add(outputParam);

            // ── Execute ───────────────────────────────────────────
            cmd.ExecuteNonQuery();

            // ── Read output AFTER execution ───────────────────────
            int newId = (int)cmd.Parameters["@NewId"].Value;
            Console.WriteLine($"✅ Inserted. New ID = {newId}");
            return newId;
        }
    }
}
```

---

### Example 2 — Get count from SP

```csharp
public int GetDeptCount(string department)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetDeptCount", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            // Input
            cmd.Parameters.AddWithValue("@Department", department);

            // Output — shortcut style
            cmd.Parameters.Add("@Count", SqlDbType.Int).Direction =
                ParameterDirection.Output;

            cmd.ExecuteNonQuery();

            // Read after execution
            int count = (int)cmd.Parameters["@Count"].Value;
            Console.WriteLine($"Employees in {department}: {count}");
            return count;
        }
    }
}
```

---

### Example 3 — Multiple output parameters

```csharp
public (bool success, string message) InsertWithStatus(string name, string email)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_InsertWithStatus", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            // Input parameters
            cmd.Parameters.AddWithValue("@Name",  name);
            cmd.Parameters.AddWithValue("@Email", email);

            // Output: message string
            cmd.Parameters.Add("@Message", SqlDbType.NVarChar, 200).Direction =
                ParameterDirection.Output;

            // Output: success flag
            cmd.Parameters.Add("@Success", SqlDbType.Bit).Direction =
                ParameterDirection.Output;

            cmd.ExecuteNonQuery();

            // Read both outputs after execution
            string message = cmd.Parameters["@Message"].Value.ToString();
            bool   success = (bool)cmd.Parameters["@Success"].Value;

            Console.WriteLine($"Message: {message}");
            Console.WriteLine($"Success: {success}");

            return (success, message);
        }
    }
}
```

---

## 🔷 Return Value — SP RETURN Statement

Every SP can return an integer status code using `RETURN`.
This is different from an output parameter.

```sql
CREATE PROCEDURE sp_CheckEmployee
    @Id INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Employee WHERE Id = @Id)
        RETURN 1   -- found
    ELSE
        RETURN 0   -- not found
END
```

```csharp
public int CheckEmployee(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_CheckEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Id", id);

            // Return value parameter — must be added BEFORE executing
            SqlParameter returnParam = new SqlParameter("@RetVal", SqlDbType.Int);
            returnParam.Direction = ParameterDirection.ReturnValue;
            cmd.Parameters.Add(returnParam);

            cmd.ExecuteNonQuery();

            // Read RETURN value after execution
            int returnValue = (int)cmd.Parameters["@RetVal"].Value;
            Console.WriteLine(returnValue == 1 ? "Found" : "Not found");
            return returnValue;
        }
    }
}
```

---

## 🔷 Output vs Return Value — Difference

|             | Output Parameter              | Return Value                       |
| ----------- | ----------------------------- | ---------------------------------- |
| SQL keyword | `@Param TYPE OUTPUT`        | `RETURN value`                   |
| Can hold    | Any data type                 | `int`only                        |
| Multiple    | ✅ Yes — many output params  | ❌ Only one RETURN                 |
| Direction   | `ParameterDirection.Output` | `ParameterDirection.ReturnValue` |
| Best for    | Data values, IDs, messages    | Status codes (0=fail, 1=success)   |

---

## 🔷 Shortcut — AddWithValue Style for Output

```csharp
// Longer style (full control):
SqlParameter p = new SqlParameter("@NewId", SqlDbType.Int);
p.Direction    = ParameterDirection.Output;
cmd.Parameters.Add(p);

// Shorter style (same result):
cmd.Parameters.Add("@NewId", SqlDbType.Int).Direction = ParameterDirection.Output;
```

---

## ⚠️ Common Mistakes

| Mistake                                         | What Happens                               | Fix                                                         |
| ----------------------------------------------- | ------------------------------------------ | ----------------------------------------------------------- |
| Reading output param BEFORE `ExecuteNonQuery` | Value is null or 0 — SP hasn't run yet    | Always read AFTER execution                                 |
| Forgetting `Direction = Output`               | Param treated as Input — SQL can't set it | Set `ParameterDirection.Output`                           |
| Wrong `SqlDbType`for output                   | Cast fails after execution                 | Match SQL type exactly                                      |
| Missing size for string output (`NVarChar`)   | Truncated or error                         | `Add("@Msg", SqlDbType.NVarChar, 200)`— always give size |

---

## ⭐ Interview Quick-Fire

| Question                                     | Answer                                                                                |
| -------------------------------------------- | ------------------------------------------------------------------------------------- |
| What is an output parameter?                 | A parameter where SQL Server sets the value during SP execution, C# reads it after    |
| How do you mark a parameter as output in C#? | `param.Direction = ParameterDirection.Output`                                       |
| When do you read the output value?           | AFTER `ExecuteNonQuery()`or `ExecuteReader()`— never before                      |
| What is `ParameterDirection.ReturnValue`?  | Captures the SP's `RETURN`integer — status code                                    |
| Difference: Output vs ReturnValue?           | Output can be any type and there can be many; ReturnValue is `int`only and just one |
| What if you forget to set Direction?         | Parameter behaves as Input — SQL cannot set it, value stays null                     |
| How to read the output value?                | `cmd.Parameters["@ParamName"].Value`— cast to correct type                         |
