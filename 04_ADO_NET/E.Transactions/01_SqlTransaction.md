
# 01 — SqlTransaction

---

## 🎯 One-Line Definition

> **`SqlTransaction` groups multiple SQL commands into one atomic unit — either ALL of them succeed together (COMMIT) or ALL of them are undone together (ROLLBACK), so the database never ends up in a half-done state.**

---

## 🔷 The Problem SqlTransaction Solves

```
SCENARIO: Transfer ₹50,000 from Alice's account to Bob's account.

WITHOUT a transaction (two separate commands):
──────────────────────────────────────────────────────────────
Step 1: UPDATE Accounts SET Balance = Balance - 50000 WHERE Id = 1  ✅ Success
Step 2: UPDATE Accounts SET Balance = Balance + 50000 WHERE Id = 2  ❌ CRASH!

Result:
  Alice lost ₹50,000  ❌
  Bob never received  ❌
  Money vanished      ❌  ← data is corrupted

WITH a transaction:
──────────────────────────────────────────────────────────────
BEGIN TRANSACTION
  Step 1: Debit Alice   ✅ (held pending — not committed yet)
  Step 2: Credit Bob    ❌ (error occurs)
ROLLBACK                ← BOTH steps undone automatically

Result:
  Alice still has her money  ✅
  Bob unchanged              ✅
  Database consistent        ✅
```

---

## 🔷 The ACID Properties

| Letter      | Property    | Meaning                                                    |
| ----------- | ----------- | ---------------------------------------------------------- |
| **A** | Atomicity   | All operations succeed, or none do — no partial results   |
| **C** | Consistency | Database moves from one valid state to another valid state |
| **I** | Isolation   | Concurrent transactions don't interfere with each other    |
| **D** | Durability  | Once committed, data survives crashes and restarts         |

> 📌 `SqlTransaction` primarily provides **Atomicity** — the A in ACID.

---

## 🔷 How SqlTransaction Works

```
Step 1: con.BeginTransaction()    → SQL Server starts holding changes pending
                ↓
Step 2: Execute commands           → changes held, not visible to other queries yet
                ↓
Step 3a: tx.Commit()              → all changes applied permanently ✅
                OR
Step 3b: tx.Rollback()            → all changes undone, DB unchanged ❌
```

---

## 🔷 Key Methods

| Method                     | What It Does                                       |
| -------------------------- | -------------------------------------------------- |
| `con.BeginTransaction()` | Starts a transaction — returns `SqlTransaction` |
| `tx.Commit()`            | Applies all changes permanently                    |
| `tx.Rollback()`          | Undoes all changes since `BeginTransaction()`    |
| `cmd.Transaction = tx`   | Assigns the transaction to a command               |

> ⚠️ Every `SqlCommand` inside a transaction **must have its `Transaction` property set** to `tx` — otherwise the command runs outside the transaction and won't be rolled back on failure.

---

## 🔷 Basic Syntax — The Template

```csharp
using (SqlConnection con = new SqlConnection(_cs))
{
    con.Open();
    SqlTransaction tx = con.BeginTransaction();

    try
    {
        // Pass 'tx' as the 3rd argument to every command
        using (SqlCommand cmd1 = new SqlCommand(sql1, con, tx))
            cmd1.ExecuteNonQuery();

        using (SqlCommand cmd2 = new SqlCommand(sql2, con, tx))
            cmd2.ExecuteNonQuery();

        tx.Commit();   // ← all succeeded — make permanent
    }
    catch (Exception ex)
    {
        tx.Rollback();   // ← something failed — undo everything
        Console.WriteLine($"Rolled back: {ex.Message}");
        throw;
    }
}
```

---

## 🔷 Complete Example — Transfer Between Two Employees

```csharp
public bool TransferSalary(int fromId, int toId, decimal amount)
{
    string debitSql  = "UPDATE Employee SET Salary = Salary - @Amount WHERE Id = @Id";
    string creditSql = "UPDATE Employee SET Salary = Salary + @Amount WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();

        try
        {
            // Step 1: Debit
            using (SqlCommand cmd = new SqlCommand(debitSql, con, tx))
            {
                cmd.Parameters.AddWithValue("@Amount", amount);
                cmd.Parameters.AddWithValue("@Id",     fromId);
                cmd.ExecuteNonQuery();
            }

            // Step 2: Credit
            using (SqlCommand cmd = new SqlCommand(creditSql, con, tx))
            {
                cmd.Parameters.AddWithValue("@Amount", amount);
                cmd.Parameters.AddWithValue("@Id",     toId);
                cmd.ExecuteNonQuery();
            }

            tx.Commit();
            Console.WriteLine($"✅ Transfer of {amount:C0} successful.");
            return true;
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"❌ Transfer failed. Rolled back: {ex.Message}");
            return false;
        }
    }
}
```

---

## 🔷 Complete Example — Order with Multiple Items

```csharp
public int CreateOrderWithItems(Order order, List<OrderItem> items)
{
    string insertOrderSql = @"INSERT INTO Orders (CustomerId, OrderDate, Total)
                               VALUES (@CustId, @Date, @Total);
                               SELECT SCOPE_IDENTITY();";

    string insertItemSql  = @"INSERT INTO OrderItems (OrderId, ProductId, Qty, Price)
                               VALUES (@OrderId, @ProductId, @Qty, @Price)";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();

        try
        {
            // Insert order header — get new OrderId
            int orderId;
            using (SqlCommand cmd = new SqlCommand(insertOrderSql, con, tx))
            {
                cmd.Parameters.AddWithValue("@CustId", order.CustomerId);
                cmd.Parameters.AddWithValue("@Date",   order.OrderDate);
                cmd.Parameters.AddWithValue("@Total",  order.Total);
                orderId = Convert.ToInt32(cmd.ExecuteScalar());
            }

            // Insert each order item
            foreach (var item in items)
            {
                using (SqlCommand cmd = new SqlCommand(insertItemSql, con, tx))
                {
                    cmd.Parameters.AddWithValue("@OrderId",   orderId);
                    cmd.Parameters.AddWithValue("@ProductId", item.ProductId);
                    cmd.Parameters.AddWithValue("@Qty",       item.Quantity);
                    cmd.Parameters.AddWithValue("@Price",     item.Price);
                    cmd.ExecuteNonQuery();
                }
            }

            tx.Commit();
            Console.WriteLine($"✅ Order {orderId} created with {items.Count} items.");
            return orderId;
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"❌ Order creation failed. Rolled back: {ex.Message}");
            return -1;
        }
    }
}
```

---

## 🔷 Transaction Isolation Levels

```csharp
// Specify isolation level on BeginTransaction
SqlTransaction tx = con.BeginTransaction(IsolationLevel.ReadCommitted);
```

| Isolation Level     | Dirty Read  | Non-Repeatable Read | Phantom Read | Use When                         |
| ------------------- | ----------- | ------------------- | ------------ | -------------------------------- |
| `ReadUncommitted` | ✅ Possible | ✅ Possible         | ✅ Possible  | Read-only, approximate reporting |
| `ReadCommitted`   | ❌ No       | ✅ Possible         | ✅ Possible  | **Default — most apps**   |
| `RepeatableRead`  | ❌ No       | ❌ No               | ✅ Possible  | Re-read same data multiple times |
| `Serializable`    | ❌ No       | ❌ No               | ❌ No        | Strictest — financial systems   |

> 📌 `ReadCommitted` is SQL Server's default — the right choice for most ADO.NET applications.

---

## ⚠️ Common Mistakes

| Mistake                                | What Happens                                                   | Fix                                         |
| -------------------------------------- | -------------------------------------------------------------- | ------------------------------------------- |
| Not passing `tx`to `SqlCommand`    | Command runs outside transaction — NOT rolled back on failure | Always `new SqlCommand(sql, con, tx)`     |
| Not calling `Rollback()`in `catch` | Transaction stays open, DB locks held                          | Always `tx.Rollback()`in catch block      |
| Rollback after connection closes       | Exception — connection disposed                               | Rollback inside catch before `using`exits |

---

## ⭐ Interview Quick-Fire

| Question                                | Answer                                                      |
| --------------------------------------- | ----------------------------------------------------------- |
| What is SqlTransaction?                 | Groups multiple SQL commands — all commit or all rollback  |
| What are ACID properties?               | Atomicity, Consistency, Isolation, Durability               |
| How to start a transaction?             | `SqlTransaction tx = con.BeginTransaction()`              |
| How to link a command to a transaction? | Pass `tx`as 3rd argument:`new SqlCommand(sql, con, tx)` |
| Where does `Commit()`go?              | After all commands succeed, in `try`block                 |
| Where does `Rollback()`go?            | In the `catch`block                                       |
| Default SQL Server isolation level?     | `ReadCommitted`                                           |
