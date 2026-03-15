
# 07 — ExecuteNonQuery

---

## 🎯 One-Line Definition

> **`ExecuteNonQuery()` executes a SQL command that does NOT return rows — INSERT, UPDATE, DELETE, or DDL statements — and returns the number of rows affected as an `int`.**

---

## 🔷 What is ExecuteNonQuery?

`ExecuteNonQuery()` is the method you call when you want to **change data** — not read it.
It sends the SQL to the database and tells you: **"how many rows did that affect?"**

> 💡 If `ExecuteReader()` is asking a question and waiting for an answer, `ExecuteNonQuery()` is **giving an order** — and the response is just "done, X rows changed."

---

## 🔷 Basic Syntax

```csharp
using (SqlCommand cmd = new SqlCommand(sql, con))
{
    cmd.Parameters.AddWithValue("@Name", "Alice");

    int rowsAffected = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rowsAffected} row(s) affected.");
}
```

---

## 🔷 Return Value — `int`

```
Returns: int = number of rows affected

  INSERT one row    →  returns 1
  UPDATE three rows →  returns 3
  DELETE two rows   →  returns 2
  No rows matched   →  returns 0   ← important: not -1, not null
  DDL (CREATE TABLE)→  returns -1  ← DDL does not affect rows
```

> ✅ Check return value to confirm the operation actually did something.

---

## 🔷 When to Use ExecuteNonQuery

```
✅ INSERT  — add a new row
✅ UPDATE  — modify existing rows
✅ DELETE  — remove rows
✅ TRUNCATE — clear a table (returns -1)
✅ CREATE, DROP, ALTER (DDL) — returns -1
✅ Stored procedures that don't SELECT
❌ SELECT — never use for queries that return rows → use ExecuteReader / ExecuteScalar
```

---

## 🔷 INSERT — Add a New Row

```csharp
public void InsertEmployee(string name, string role, string email, decimal salary)
{
    string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                   VALUES (@Name, @Role, @Email, @Salary)";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",   name);
            cmd.Parameters.AddWithValue("@Role",   role);
            cmd.Parameters.AddWithValue("@Email",  email);
            cmd.Parameters.AddWithValue("@Salary", salary);

            int rows = cmd.ExecuteNonQuery();

            if (rows > 0)
                Console.WriteLine($"✅ Employee inserted successfully.");
            else
                Console.WriteLine($"⚠️ No rows inserted.");
        }
    }
}
```

---

## 🔷 UPDATE — Modify Existing Rows

```csharp
public int UpdateSalary(int id, decimal newSalary)
{
    string sql = "UPDATE Employee SET Salary = @Salary WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Salary", newSalary);
            cmd.Parameters.AddWithValue("@Id",     id);

            int rows = cmd.ExecuteNonQuery();

            if (rows > 0)
                Console.WriteLine($"✅ Salary updated — {rows} row(s) affected.");
            else
                Console.WriteLine($"⚠️ Employee Id {id} not found.");

            return rows;
        }
    }
}
```

---

## 🔷 DELETE — Remove a Row

```csharp
public bool DeleteEmployee(int id)
{
    string sql = "DELETE FROM Employee WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            int rows = cmd.ExecuteNonQuery();

            if (rows > 0)
            {
                Console.WriteLine($"🗑️ Employee deleted.");
                return true;
            }
            else
            {
                Console.WriteLine($"⚠️ Employee Id {id} not found.");
                return false;
            }
        }
    }
}
```

---

## 🔷 Bulk UPDATE — Multiple Rows at Once

```csharp
public int ApplyRaise(decimal raisePercent, string department)
{
    string sql = @"UPDATE Employee
                   SET Salary = Salary * (1 + @RaisePct)
                   WHERE Department = @Dept";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@RaisePct", raisePercent / 100);
            cmd.Parameters.AddWithValue("@Dept",     department);

            int rows = cmd.ExecuteNonQuery();
            Console.WriteLine($"✅ {rows} employees received a raise in {department}.");
            return rows;
        }
    }
}
```

---

## 🔷 INSERT and Get the New ID — Combined

A common pattern: INSERT a row and immediately get its generated ID.

```csharp
public int InsertEmployeeAndGetId(string name, decimal salary)
{
    // Two statements in one — INSERT then immediately ask for the new ID
    string sql = @"INSERT INTO Employee (Name, Salary)
                   VALUES (@Name, @Salary);
                   SELECT SCOPE_IDENTITY();";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",   name);
            cmd.Parameters.AddWithValue("@Salary", salary);

            // ExecuteScalar here — because SELECT SCOPE_IDENTITY() returns a value
            int newId = Convert.ToInt32(cmd.ExecuteScalar());
            Console.WriteLine($"✅ Inserted. New Id = {newId}");
            return newId;
        }
    }
}
```

> 📌 When you need the new ID back — switch to `ExecuteScalar()` because `SELECT SCOPE_IDENTITY()` returns a value.

---

## 🔷 ExecuteNonQuery with Stored Procedure

```csharp
public void DeleteEmployeeSP(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_DeleteEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Id", id);

            int rows = cmd.ExecuteNonQuery();
            Console.WriteLine($"🗑️ Deleted — {rows} row(s) affected.");
        }
    }
}
```

---

## 🔷 Return Value Meanings

| Return Value | Meaning                                                           |
| ------------ | ----------------------------------------------------------------- |
| `1`        | Exactly one row affected (typical for INSERT/UPDATE/DELETE by ID) |
| `> 1`      | Multiple rows affected (UPDATE/DELETE without specific ID)        |
| `0`        | No rows matched — WHERE condition found nothing                  |
| `-1`       | DDL command (CREATE, DROP, ALTER) or stored procedure with no DML |

---

## 🔷 Three Execute Methods — Final Comparison

| Method                | Use For                             | Returns                  |
| --------------------- | ----------------------------------- | ------------------------ |
| `ExecuteReader()`   | SELECT — multiple rows             | `SqlDataReader`        |
| `ExecuteScalar()`   | SELECT — single value (COUNT, MAX) | `object`— cast needed |
| `ExecuteNonQuery()` | INSERT, UPDATE, DELETE, DDL         | `int`— rows affected  |

---

## ⚠️ Common Mistakes

| Mistake                               | What Happens                        | Fix                                           |
| ------------------------------------- | ----------------------------------- | --------------------------------------------- |
| Using `ExecuteNonQuery()`for SELECT | Returns `-1`, no data             | Use `ExecuteReader()`or `ExecuteScalar()` |
| Not checking return value             | Silently fails when `Id`not found | Always check `if (rows > 0)`                |
| No parameters (string concatenation)  | SQL Injection risk                  | Always use `@param`parameters               |
| Forgetting `con.Open()`             | `InvalidOperationException`       | Open connection before `ExecuteNonQuery()`  |

---

## ⭐ Interview Quick-Fire

| Question                               | Answer                                                  |
| -------------------------------------- | ------------------------------------------------------- |
| What does `ExecuteNonQuery()`return? | `int`— number of rows affected                       |
| When to use `ExecuteNonQuery()`?     | INSERT, UPDATE, DELETE, DDL                             |
| What does `0`mean as return value?   | No rows matched the WHERE condition                     |
| What does `-1`mean?                  | DDL statement or stored procedure with no row count     |
| Can you use it for SELECT?             | ❌ No — use `ExecuteReader()`or `ExecuteScalar()`  |
| How to get the new ID after INSERT?    | Use `SELECT SCOPE_IDENTITY()`with `ExecuteScalar()` |
| How to confirm delete was successful?  | Check if return value `> 0`                           |
