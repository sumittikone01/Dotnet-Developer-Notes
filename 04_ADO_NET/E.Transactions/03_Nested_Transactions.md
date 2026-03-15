
# 03 — Nested Transactions (Savepoints)

---

## 🎯 One-Line Definition

> **A Savepoint marks a checkpoint inside a transaction — `ROLLBACK TO SAVEPOINT` undoes only the work done after that point, while keeping everything before it intact, giving you partial rollback without losing all your work.**

---

## 🔷 The Problem Savepoints Solve

```
Imagine a transaction with 4 steps:

BEGIN TRANSACTION
  Step 1: Insert Order Header     ← want to KEEP this ✅
  Step 2: Insert Order Items      ← want to KEEP this ✅
  SAVEPOINT AfterItems            ← checkpoint saved here
  Step 3: Update Inventory        ← might fail ❌
  Step 4: Send notification       ← might fail ❌

WITHOUT savepoint:
  If Step 3 fails → ROLLBACK → Steps 1, 2, 3, 4 ALL undone
  You lose the order completely — must start over

WITH savepoint:
  If Step 3 fails → ROLLBACK TO AfterItems
  → Steps 1 and 2 KEPT ✅  Steps 3 and 4 UNDONE ❌
  → Order exists, inventory not updated, no notification
  → You can retry just the inventory update
```

---

## 🔷 True Nested Transactions vs Savepoints

```
TRUE NESTED TRANSACTIONS:
  SQL Server does NOT support true nested transactions.
  A BEGIN TRANSACTION inside another BEGIN TRANSACTION
  does NOT create an independent inner transaction.
  The inner Commit does nothing — only the OUTERMOST Commit saves.
  The inner Rollback rolls back EVERYTHING (outer too).

SAVEPOINTS (what actually works in SQL Server):
  tx.Save("savepointName")          → creates a checkpoint
  tx.Rollback("savepointName")      → rolls back to that checkpoint ONLY
  tx.Commit()                       → commits everything
  This is the correct way to do partial rollbacks in ADO.NET.
```

---

## 🔷 Savepoint Methods in ADO.NET

| Method                  | What It Does                                               |
| ----------------------- | ---------------------------------------------------------- |
| `tx.Save("name")`     | Creates a named savepoint at this point in the transaction |
| `tx.Rollback("name")` | Rolls back to the savepoint (does NOT end the transaction) |
| `tx.Rollback()`       | Rolls back the entire transaction (no savepoint name)      |
| `tx.Commit()`         | Commits everything — savepoints are cleared               |

---

## 🔷 SQL Server T-SQL — Savepoint Syntax

```sql
BEGIN TRANSACTION

  -- Step 1
  INSERT INTO Orders (CustomerId) VALUES (1)

  -- Step 2
  INSERT INTO OrderItems (OrderId, ProductId) VALUES (1, 10)

  -- Create a savepoint
  SAVE TRANSACTION AfterOrderItems

  -- Step 3 — risky operation
  UPDATE Inventory SET Qty = Qty - 5 WHERE ProductId = 10

  -- If Step 3 needs to be undone but Steps 1 and 2 kept:
  ROLLBACK TRANSACTION AfterOrderItems
  -- (transaction still open — Steps 1 and 2 still pending)

COMMIT   -- saves Steps 1 and 2 only
```

---

## 🔷 C# ADO.NET — Savepoint Example

```csharp
public int CreateOrderWithInventoryUpdate(Order order, List<OrderItem> items)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();
        int orderId = -1;

        try
        {
            // ── PHASE 1: Create the order ─────────────────────────
            using (SqlCommand cmd = new SqlCommand(
                @"INSERT INTO Orders (CustomerId, OrderDate, Total)
                  VALUES (@CustId, GETDATE(), @Total);
                  SELECT SCOPE_IDENTITY();", con, tx))
            {
                cmd.Parameters.AddWithValue("@CustId", order.CustomerId);
                cmd.Parameters.AddWithValue("@Total",  order.Total);
                orderId = Convert.ToInt32(cmd.ExecuteScalar());
            }

            // ── PHASE 2: Insert order items ───────────────────────
            foreach (var item in items)
            {
                using (SqlCommand cmd = new SqlCommand(
                    "INSERT INTO OrderItems (OrderId, ProductId, Qty) VALUES (@OId, @PId, @Qty)",
                    con, tx))
                {
                    cmd.Parameters.AddWithValue("@OId", orderId);
                    cmd.Parameters.AddWithValue("@PId", item.ProductId);
                    cmd.Parameters.AddWithValue("@Qty", item.Quantity);
                    cmd.ExecuteNonQuery();
                }
            }

            // ── SAVEPOINT: Order is confirmed — mark it ───────────
            tx.Save("AfterOrderCreation");
            Console.WriteLine($"📌 Savepoint created. Order {orderId} safe.");

            // ── PHASE 3: Update inventory (may fail for some items) ─
            foreach (var item in items)
            {
                try
                {
                    using (SqlCommand cmd = new SqlCommand(
                        @"UPDATE Inventory SET Stock = Stock - @Qty
                          WHERE ProductId = @PId AND Stock >= @Qty", con, tx))
                    {
                        cmd.Parameters.AddWithValue("@Qty", item.Quantity);
                        cmd.Parameters.AddWithValue("@PId", item.ProductId);

                        int updated = cmd.ExecuteNonQuery();

                        if (updated == 0)
                        {
                            // Not enough stock — rollback ONLY inventory updates
                            tx.Rollback("AfterOrderCreation");
                            Console.WriteLine($"⚠️ Insufficient stock for Product {item.ProductId}.");
                            Console.WriteLine("🔄 Rolled back to savepoint — order kept, inventory unchanged.");

                            // Order still committed without inventory update
                            tx.Commit();
                            return orderId;
                        }
                    }
                }
                catch (SqlException)
                {
                    // Inventory update failed — rollback to savepoint, keep order
                    tx.Rollback("AfterOrderCreation");
                    tx.Commit();
                    return orderId;
                }
            }

            // ── PHASE 4: All inventory updated — commit everything ─
            tx.Commit();
            Console.WriteLine($"✅ Order {orderId} and inventory both committed.");
            return orderId;
        }
        catch (Exception ex)
        {
            // Phase 1 or 2 failed — rollback everything
            tx.Rollback();
            Console.WriteLine($"❌ Order creation failed completely. Rolled back: {ex.Message}");
            return -1;
        }
    }
}
```

---

## 🔷 Multiple Savepoints in One Transaction

```csharp
public void ProcessBatchWithSavepoints(List<Employee> employees)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();
        int successCount = 0;

        try
        {
            foreach (var emp in employees)
            {
                // Create a savepoint before each employee
                string savept = $"BeforeEmp_{emp.Id}";
                tx.Save(savept);

                try
                {
                    using (SqlCommand cmd = new SqlCommand(
                        "INSERT INTO Employee (Name, Role, Salary) VALUES (@N, @R, @S)",
                        con, tx))
                    {
                        cmd.Parameters.AddWithValue("@N", emp.Name);
                        cmd.Parameters.AddWithValue("@R", emp.Role);
                        cmd.Parameters.AddWithValue("@S", emp.Salary);
                        cmd.ExecuteNonQuery();
                        successCount++;
                        Console.WriteLine($"✅ {emp.Name} inserted.");
                    }
                }
                catch (Exception)
                {
                    // This employee failed — roll back just this one
                    tx.Rollback(savept);
                    Console.WriteLine($"⚠️ Failed to insert {emp.Name} — skipped.");
                    // Transaction still active — continue with next employee
                }
            }

            // Commit all that succeeded
            tx.Commit();
            Console.WriteLine($"✅ Batch complete. {successCount}/{employees.Count} inserted.");
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"❌ Batch failed completely: {ex.Message}");
        }
    }
}
```

---

## 🔷 How Savepoints Work — Visual

```
BEGIN TRANSACTION
│
├── INSERT Order         ✅
├── INSERT OrderItems    ✅
│
├── SAVE "AfterOrder"   ←── checkpoint
│
├── UPDATE Inventory    ❌  fails
│
├── ROLLBACK TO "AfterOrder"
│   └── Inventory update undone — Order INSERT still intact
│
├── (transaction still open)
│
└── COMMIT              ✅  Order saved, inventory not updated
```

---

## 🔷 Savepoint vs Full Rollback — Comparison

|                    | Full `ROLLBACK`          | `ROLLBACK TO savepointName`     |
| ------------------ | -------------------------- | --------------------------------- |
| Undoes             | Everything since `BEGIN` | Only from savepoint to now        |
| Transaction        | Ends — cannot continue    | Stays open — can continue        |
| Savepoint cleared? | All cleared                | Only later savepoints             |
| Use when           | Total failure              | Partial failure — keep some work |

---

## ⚠️ Common Mistakes

| Mistake                                                           | What Happens                                          | Fix                                                                  |
| ----------------------------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------- |
| `tx.Rollback("name")`— typo in savepoint name                  | Exception — savepoint not found                      | Savepoint name must match exactly                                    |
| Committing after `Rollback("name")`expecting both phases        | Works — commit saves whatever is left                | This is correct — Rollback to savepoint doesn't end the transaction |
| Using nested `BeginTransaction()`expecting independent inner tx | Inner commit does nothing, inner rollback kills outer | Use savepoints for "nested" behaviour                                |
| Forgetting to commit after `Rollback("savepoint")`              | Transaction stays open, locks held                    | Always commit or fully rollback after partial rollback               |

---

## ⭐ Interview Quick-Fire

| Question                                                   | Answer                                                                                     |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Does SQL Server support true nested transactions?          | ❌ No — use savepoints for partial rollback                                               |
| What is a savepoint?                                       | A named checkpoint inside a transaction                                                    |
| C# method to create a savepoint?                           | `tx.Save("savepointName")`                                                               |
| C# method to rollback to a savepoint?                      | `tx.Rollback("savepointName")`— transaction stays open                                  |
| `Rollback("name")`vs `Rollback()`?                     | With name = partial rollback to checkpoint. Without name = full rollback, transaction ends |
| After `Rollback("name")`, is the transaction still open? | ✅ Yes — you can continue executing commands                                              |
| When to use savepoints?                                    | Batch operations where some failures should be skipped, not cause full rollback            |
