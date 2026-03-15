
# 02 — BEGIN, COMMIT, ROLLBACK

---

## 🎯 One-Line Definition

> **`BEGIN TRANSACTION` marks the start of a transaction, `COMMIT` saves all changes permanently, and `ROLLBACK` undoes everything back to the `BEGIN` point — these three commands are the complete lifecycle of any transaction.**

---

## 🔷 The Three Commands — Visual

```
Timeline of a transaction:

  ────────────────────────────────────────────────────────────
  Normal DB       BEGIN TRANSACTION        COMMIT or ROLLBACK
  state           ↓                        ↓
  ────────────────●────────────────────────●────────────────
                  │  ← changes pending,    │
                  │    not yet visible      │
                  │    to other queries     │
                  │                        │
                  │   INSERT...            COMMIT  → changes saved ✅
                  │   UPDATE...             OR
                  │   DELETE...            ROLLBACK → changes undone ❌
  ────────────────────────────────────────────────────────────
```

---

## 🔷 SQL Server Side — T-SQL Syntax

These commands can be run directly in SQL Server (SSMS):

```sql
-- Start a transaction
BEGIN TRANSACTION
-- or shorthand:
BEGIN TRAN

-- All SQL inside the transaction
INSERT INTO Employee (Name, Role, Salary) VALUES ('Alice', 'Dev', 75000)
UPDATE Employee SET Salary = 80000 WHERE Id = 1
DELETE FROM Employee WHERE Id = 5

-- Option A: Save all changes
COMMIT TRANSACTION
-- or shorthand:
COMMIT

-- Option B: Undo all changes
ROLLBACK TRANSACTION
-- or shorthand:
ROLLBACK
```

---

## 🔷 C# ADO.NET — How They Map

| SQL T-SQL Command     | C# ADO.NET Equivalent                          |
| --------------------- | ---------------------------------------------- |
| `BEGIN TRANSACTION` | `SqlTransaction tx = con.BeginTransaction()` |
| `COMMIT`            | `tx.Commit()`                                |
| `ROLLBACK`          | `tx.Rollback()`                              |

---

## 🔷 The Full Lifecycle in C#

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    // ── BEGIN TRANSACTION ─────────────────────────────────────────
    SqlTransaction tx = con.BeginTransaction();

    try
    {
        using (SqlCommand cmd1 = new SqlCommand(
            "INSERT INTO Employee (Name, Salary) VALUES (@Name, @Salary)", con, tx))
        {
            cmd1.Parameters.AddWithValue("@Name",   "Alice");
            cmd1.Parameters.AddWithValue("@Salary", 75000);
            cmd1.ExecuteNonQuery();
            Console.WriteLine("Step 1: INSERT executed — pending");
        }

        using (SqlCommand cmd2 = new SqlCommand(
            "UPDATE Department SET HeadCount = HeadCount + 1 WHERE Name = 'IT'", con, tx))
        {
            cmd2.ExecuteNonQuery();
            Console.WriteLine("Step 2: UPDATE executed — pending");
        }

        // ── COMMIT ────────────────────────────────────────────────
        tx.Commit();
        Console.WriteLine("✅ COMMIT — both changes saved permanently.");
    }
    catch (Exception ex)
    {
        // ── ROLLBACK ──────────────────────────────────────────────
        tx.Rollback();
        Console.WriteLine($"❌ ROLLBACK — all changes undone. Error: {ex.Message}");
        throw;
    }
}
```

---

## 🔷 COMMIT — What Happens

```
Before COMMIT:
  Changes exist only in the transaction's "undo log"
  Other connections CAN'T see these changes (ReadCommitted)
  SQL Server holds locks on the affected rows

After COMMIT:
  Changes written to the actual database permanently
  Other connections CAN now see the changes
  Locks released
  Transaction ends — tx object no longer usable
```

---

## 🔷 ROLLBACK — What Happens

```
When ROLLBACK is called:
  SQL Server reads the undo log
  Reverses every change made since BEGIN TRANSACTION
  INSERT undone (row removed)
  UPDATE undone (old values restored)
  DELETE undone (row brought back)
  Locks released
  Transaction ends — database back to its state before BEGIN
```

---

## 🔷 Rollback on Specific Business Rule Failure

```csharp
public bool ProcessPayroll(int employeeId, decimal bonus)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();

        try
        {
            // Check budget before making changes
            using (SqlCommand checkCmd = new SqlCommand(
                "SELECT BudgetRemaining FROM Department WHERE Id = " +
                "(SELECT DeptId FROM Employee WHERE Id = @Id)", con, tx))
            {
                checkCmd.Parameters.AddWithValue("@Id", employeeId);
                decimal budget = Convert.ToDecimal(checkCmd.ExecuteScalar());

                if (budget < bonus)
                {
                    // Business rule violation — rollback manually
                    tx.Rollback();
                    Console.WriteLine("❌ Insufficient budget. Transaction rolled back.");
                    return false;
                }
            }

            // Deduct from department budget
            using (SqlCommand deductCmd = new SqlCommand(
                "UPDATE Department SET BudgetRemaining = BudgetRemaining - @Bonus " +
                "WHERE Id = (SELECT DeptId FROM Employee WHERE Id = @EmpId)", con, tx))
            {
                deductCmd.Parameters.AddWithValue("@Bonus", bonus);
                deductCmd.Parameters.AddWithValue("@EmpId", employeeId);
                deductCmd.ExecuteNonQuery();
            }

            // Add bonus to employee salary
            using (SqlCommand bonusCmd = new SqlCommand(
                "UPDATE Employee SET Salary = Salary + @Bonus WHERE Id = @Id", con, tx))
            {
                bonusCmd.Parameters.AddWithValue("@Bonus", bonus);
                bonusCmd.Parameters.AddWithValue("@Id",    employeeId);
                bonusCmd.ExecuteNonQuery();
            }

            // Log the transaction
            using (SqlCommand logCmd = new SqlCommand(
                "INSERT INTO PayrollLog (EmployeeId, BonusAmount, ProcessedAt) " +
                "VALUES (@EmpId, @Bonus, GETDATE())", con, tx))
            {
                logCmd.Parameters.AddWithValue("@EmpId", employeeId);
                logCmd.Parameters.AddWithValue("@Bonus", bonus);
                logCmd.ExecuteNonQuery();
            }

            tx.Commit();
            Console.WriteLine($"✅ Bonus of {bonus:C0} processed for Employee {employeeId}.");
            return true;
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"❌ Payroll processing failed. Rolled back: {ex.Message}");
            return false;
        }
    }
}
```

---

## 🔷 Commit vs Rollback — What Gets Undone

```
BEGIN TRANSACTION
  INSERT INTO Employee ... → added to DB (pending)
  UPDATE Employee ...      → changed in DB (pending)
  DELETE FROM Employee ... → removed from DB (pending)

COMMIT:
  All three changes → PERMANENT in database ✅

ROLLBACK:
  INSERT undone → row removed
  UPDATE undone → old values restored
  DELETE undone → row brought back
  DB is exactly as it was before BEGIN ✅
```

---

## 🔷 Always Use try-catch-finally Pattern

```csharp
SqlTransaction tx = null;

using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();

    try
    {
        tx = con.BeginTransaction();

        // ... execute commands with tx ...

        tx.Commit();
    }
    catch (Exception ex)
    {
        // Only rollback if transaction was started and not already completed
        if (tx != null)
        {
            try { tx.Rollback(); }
            catch (Exception rbEx)
            {
                Console.Error.WriteLine($"Rollback also failed: {rbEx.Message}");
            }
        }

        Console.Error.WriteLine($"Transaction failed: {ex.Message}");
        throw;
    }
    finally
    {
        tx?.Dispose();
    }
}
```

---

## ⚠️ Common Mistakes

| Mistake                                           | What Happens                                   | Fix                                           |
| ------------------------------------------------- | ---------------------------------------------- | --------------------------------------------- |
| Calling `Commit()`then `Rollback()`on same tx | Exception — transaction already completed     | Call only one of them                         |
| No `try-catch`around the transaction            | On exception, transaction never rolled back    | Always wrap in try-catch                      |
| Rollback after connection already closed          | Exception                                      | Keep connection open until after rollback     |
| Forgetting `COMMIT`— leaving transaction open  | Locks held indefinitely, other queries blocked | Always Commit or Rollback — never leave open |

---

## ⭐ Interview Quick-Fire

| Question                                     | Answer                                                    |
| -------------------------------------------- | --------------------------------------------------------- |
| What does `BEGIN TRANSACTION`do in SQL?    | Marks the start — all following changes are held pending |
| What does `COMMIT`do?                      | Permanently saves all pending changes                     |
| What does `ROLLBACK`do?                    | Undoes all changes back to the `BEGIN TRANSACTION`point |
| C# equivalent of `BEGIN TRANSACTION`?      | `SqlTransaction tx = con.BeginTransaction()`            |
| C# equivalent of `COMMIT`?                 | `tx.Commit()`                                           |
| C# equivalent of `ROLLBACK`?               | `tx.Rollback()`                                         |
| Can you call both Commit and Rollback?       | ❌ No — only one per transaction                         |
| What happens to locks after Commit/Rollback? | Released — other queries can proceed                     |
