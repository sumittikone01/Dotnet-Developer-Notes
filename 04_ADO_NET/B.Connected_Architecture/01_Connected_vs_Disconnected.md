
# 01 — Connected vs Disconnected Architecture

---

## 🎯 One-Line Definition

> **Connected keeps the database connection open while you work with data. Disconnected fetches data into memory, closes the connection immediately, and lets you work offline.**

---

## 🔷 The Core Difference — One Visual

```
CONNECTED                              DISCONNECTED
──────────────────────────────         ──────────────────────────────────
Open Connection                        Open Connection
      ↓                                      ↓
Execute Query                          Fetch ALL data → DataTable
      ↓                                      ↓
Read Row by Row  ← connection          Close Connection  ← connection FREE!
      ↓            STILL OPEN               ↓
Process Each Row                       Edit / Work with data in memory
      ↓                                      ↓
Close Connection                       Reopen → Save changes → Close
```

---

## 🔷 Connected Architecture

### What It Is

Your application keeps the DB connection **open the entire time** it reads or writes data.

> 💡 Like a **live phone call** — you stay on the line while exchanging information. The moment you hang up, the conversation is over.

### Key Classes

| Class              | Role                                                   |
| ------------------ | ------------------------------------------------------ |
| `SqlConnection`  | Opens and closes the DB connection                     |
| `SqlCommand`     | Sends SQL to the database                              |
| `SqlDataReader`  | Reads result rows one-by-one (forward-only, read-only) |
| `SqlTransaction` | Groups multiple commands (optional)                    |

### How It Works

```
Step 1 → Open Connection       SqlConnection.Open()
Step 2 → Write SQL             string query = "SELECT..."
Step 3 → Create Command        new SqlCommand(query, con)
Step 4 → Execute               .ExecuteReader() / .ExecuteNonQuery() / .ExecuteScalar()
Step 5 → Process results       reader.Read() loop
Step 6 → Close Connection      automatic via using block
```

### Characteristics

|             |                                          |
| ----------- | ---------------------------------------- |
| Connection  | Open throughout entire operation         |
| Data        | Real-time — always fresh from DB        |
| Reading     | SqlDataReader — forward-only, read-only |
| Memory      | 🟢 Low — one row at a time              |
| Speed       | ⚡ Fast                                  |
| Scalability | 🔴 Lower — connection held open         |

### ✅ Use When

* Real-time, live data (login check, live dashboard)
* Single quick operation (insert one row, count rows)
* Reading large result sets without loading all into memory
* Fast sequential processing of many rows

### ❌ Avoid When

* You need to edit data offline → use Disconnected
* High-traffic scalable web app → connections held too long
* Passing data across layers → DataReader requires open connection

---

## 🔷 Disconnected Architecture

### What It Is

Data is  **fetched once** , stored in memory (`DataTable`/`DataSet`), and the connection is  **closed immediately** .

> 💡 Like **filling a bottle from the tap** — you fill it, close the tap, carry the bottle wherever you want, and open the tap only when you need to refill.

### Key Classes

| Class              | Role                                                 |
| ------------------ | ---------------------------------------------------- |
| `SqlDataAdapter` | Fetches data into memory AND saves changes back      |
| `DataSet`        | In-memory mini-database — holds multiple DataTables |
| `DataTable`      | Single in-memory table (rows + columns)              |
| `DataRow`        | One record inside a DataTable                        |

### How It Works

```
Step 1 → Open Connection
Step 2 → SqlDataAdapter.Fill(dataTable)  — fetches all rows
Step 3 → Connection CLOSES automatically
Step 4 → Edit / work with DataTable in memory (no DB needed)
Step 5 → Reopen → SqlDataAdapter.Update(dataTable) → Close
```

### Characteristics

|             |                                           |
| ----------- | ----------------------------------------- |
| Connection  | Closed immediately after fetch            |
| Data        | Snapshot — not real-time                 |
| Reading     | Full DataTable in memory — random access |
| Memory      | 🟡 Higher — all rows loaded              |
| Speed       | Slightly slower (loads all rows)          |
| Scalability | 🟢 Higher — connection freed instantly   |

### ✅ Use When

* Kendo Grid (load data, close connection, serve to browser)
* Reports (fetch once, display)
* Edit forms (fetch employee, edit fields, save)
* High-traffic web apps — connection freed in milliseconds

### ❌ Avoid When

* You need real-time live data
* Very large data sets you don't need fully in memory

---

## 🔷 Side-by-Side Comparison

|                | Connected         | Disconnected              |
| -------------- | ----------------- | ------------------------- |
| Connection     | Stays open        | Closes after fetch        |
| Primary class  | `SqlDataReader` | `DataTable`/`DataSet` |
| Bridge class   | `SqlCommand`    | `SqlDataAdapter`        |
| Data editing   | ❌ Read-only      | ✅ Full edit in memory    |
| Memory         | Low — one row    | Higher — all rows        |
| Scalability    | Lower             | Higher                    |
| Real-time data | ✅ Yes            | ❌ Snapshot               |
| Best for       | Login, live reads | Kendo Grids, reports      |

---

## ❓ Interview Questions

**Q: What is Connected Architecture?**

> A model where the database connection stays open while reading data row-by-row using `SqlDataReader`. Fast and memory-efficient but less scalable — the connection is tied up until all rows are read.

**Q: What is Disconnected Architecture?**

> Data is fetched into a `DataTable`/`DataSet` using `SqlDataAdapter`, the connection closes immediately, and you work with the in-memory data offline. More scalable because the connection is freed in milliseconds.

**Q: Why is Disconnected Architecture more scalable for web apps?**

> In a web app, hundreds of users make requests simultaneously. Connections are limited. Disconnected model frees the connection the instant data is fetched — it's available for the next request in milliseconds. Connected holds the connection open for the entire read operation.

**Q: When would you use Connected over Disconnected?**

> When you need real-time data (login authentication, live status checks) or when reading large streams of data you don't want to load fully into memory. For any write operation (INSERT/UPDATE/DELETE), Connected is always used.
>
