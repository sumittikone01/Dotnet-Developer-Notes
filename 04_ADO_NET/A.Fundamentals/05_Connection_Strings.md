
# 05 — Connection Strings

---

## 🎯 One-Line Definition

> **A connection string is a text string that tells ADO.NET three things: WHERE is the server, WHICH database to use, and HOW to authenticate.**

---

## 🔷 The Basic Structure

```csharp
string cs = @"Server=.\SQLEXPRESS; Database=EmployeeDB; Trusted_Connection=True; TrustServerCertificate=True;";
SqlConnection con = new SqlConnection(cs);
```

Every connection string is a  **semicolon-separated list of key=value pairs** .

---

## 🔷 Keyword 1 — Server (Where is SQL Server?)

```text
Server=localhost;               ← Local machine, default instance
Server=.;                       ← Same as localhost (shortcut)
Server=.\SQLEXPRESS;            ← Local machine, named instance SQLEXPRESS
Server=DESKTOP-ABC\SQLEXPRESS;  ← Specific PC name + named instance
Server=192.168.1.10;            ← Remote server by IP address
Server=myserver.com,1433;       ← Remote server with custom port
```

> 📌 `Server` and `Data Source` are **identical** — both work.

---

## 🔷 Keyword 2 — Database (Which database?)

```text
Database=EmployeeDB;            ← Use this
Initial Catalog=EmployeeDB;     ← Same thing — both work
```

---

## 🔷 Keyword 3 — Authentication (How to login?)

### ✅ Method A — Windows Authentication (Best for Development)

```text
Trusted_Connection=True;
Integrated Security=True;       ← Same as Trusted_Connection — both work
```

* Uses your **Windows login** — no username/password needed
* ❌ Do NOT combine with `User Id` / `Password`

```text
Server=.\SQLEXPRESS; Database=EmployeeDB; Trusted_Connection=True; TrustServerCertificate=True;
```

---

### ✅ Method B — SQL Server Authentication (For Production / Remote)

```text
User Id=sa; Password=YourPassword;
```

* Uses a **SQL Server login** — explicit username and password
* ❌ Do NOT combine with `Trusted_Connection=True`

```text
Server=.\SQLEXPRESS; Database=EmployeeDB; User Id=sa; Password=123; Encrypt=True;
```

---

### ❌ Never Mix Both — Causes Exception

```text
Trusted_Connection=True; User Id=sa; Password=123;
```

🚫 **Invalid — throws SqlException at runtime**

---

## 🔷 Windows Auth vs SQL Auth — When to Use Which

|                  | Windows Auth                | SQL Auth                          |
| ---------------- | --------------------------- | --------------------------------- |
| Uses             | Windows login               | SQL Server username/password      |
| Best for         | Development, LAN apps       | Production, remote servers, cloud |
| Password in code | ❌ None needed              | ⚠️ Must be secured              |
| Keyword          | `Trusted_Connection=True` | `User Id=x; Password=y`         |

---

## 🔷 Keyword 4 — Security Keywords

| Keyword                         | Purpose                                           | When to Use                         |
| ------------------------------- | ------------------------------------------------- | ----------------------------------- |
| `Encrypt=True`                | Encrypts all data sent to SQL Server              | Always — especially production     |
| `TrustServerCertificate=True` | Skips SSL certificate validation                  | Development only (self-signed cert) |
| `Persist Security Info=False` | Prevents exposing password after connection opens | Default — leave it                 |

---

## 🔷 Keyword 5 — Timeout

|                              | Set On            | Keyword                     | Default    |
| ---------------------------- | ----------------- | --------------------------- | ---------- |
| **Connection Timeout** | Connection string | `Connection Timeout=30`   | 15 seconds |
| **Command Timeout**    | SqlCommand object | `cmd.CommandTimeout = 60` | 30 seconds |

```csharp
// Connection Timeout — in the connection string
string cs = "Server=.; Database=EmployeeDB; Trusted_Connection=True; Connection Timeout=30;";

// Command Timeout — on the command object (not in connection string)
SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con);
cmd.CommandTimeout = 60;   // wait max 60 seconds for this query
```

---

## 🔷 Keyword 6 — Connection Pooling

**Connection Pooling** reuses existing database connections instead of creating a new one every time — saves significant time and resources.

```
WITHOUT pooling (slow):
  Request 1: Create connection → Use → DESTROY
  Request 2: Create connection → Use → DESTROY   ← expensive!
  Request 3: Create connection → Use → DESTROY

WITH pooling (fast):
  Request 1: Create connection → Use → Return to POOL
  Request 2: Reuse from POOL  → Use → Return to POOL  ← fast!
  Request 3: Reuse from POOL  → Use → Return to POOL
```

```text
Pooling=True;           ← Default: ON (don't disable it)
Min Pool Size=0;        ← Minimum connections kept alive in pool
Max Pool Size=100;      ← Maximum connections allowed (default: 100)
```

> ✅ Always use `using` blocks — they call `con.Close()` which **returns** the connection to the pool, not destroys it.

---

## 🔷 Connection String in ASP.NET Core MVC — Correct Pattern

### Step 1 — Store in `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.\\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

> 📌 In JSON, backslash must be **doubled** (`\\`) — `\` is a JSON escape character.

---

### Step 2 — Read in Repository / DAL via `IConfiguration`

```csharp
using System.Data;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;

public class EmployeeRepository
{
    private readonly string _cs;

    // IConfiguration reads from appsettings.json automatically
    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
        //                                ↑ must match key in appsettings.json
    }

    public int GetEmployeeCount()
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();

            // Optional: check connection state before using
            if (con.State == ConnectionState.Open)
            {
                Console.WriteLine($"Connected to: {con.Database}");
            }

            using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
            {
                return (int)cmd.ExecuteScalar();
            }
        }   // ← con.Close() called automatically here
    }
}
```

> 💡 The constructor uses **dependency injection** — ASP.NET Core passes `IConfiguration` automatically when the repository is created.

---

### Step 3 — Register in `Program.cs`

```csharp
builder.Services.AddScoped<EmployeeRepository>();
// Now ASP.NET Core injects IConfiguration into the constructor automatically
```

---

## 🔷 Complete Real Examples

```text
✅ Windows Auth — Development (most common for local work):
Server=.\SQLEXPRESS; Database=EmployeeDB; Trusted_Connection=True; Encrypt=True; TrustServerCertificate=True;

✅ Windows Auth — Short Form (localhost default instance):
Server=.; Database=EmployeeDB; Trusted_Connection=True; TrustServerCertificate=True;

✅ SQL Auth — Production / Remote Server:
Server=192.168.1.10; Database=EmployeeDB; User Id=sa; Password=SecurePass; Encrypt=True; TrustServerCertificate=True;

✅ Azure SQL Database:
Server=myserver.database.windows.net; Database=EmployeeDB; User Id=admin@myserver; Password=SecurePass; Encrypt=True;
```

---

## ⚠️ Common Mistakes

| Mistake                                    | What Happens                                                    | Fix                                           |
| ------------------------------------------ | --------------------------------------------------------------- | --------------------------------------------- |
| `Trusted_Connection=True`+`User Id=sa` | SqlException — invalid combination                             | Use only one auth method                      |
| Single backslash `\SQLEXPRESS`in JSON    | JSON parse error                                                | Use `\\SQLEXPRESS`in JSON                   |
| Not calling `con.Close()`                | Connection leak — pool exhausted over time                     | Always use `using`block                     |
| Hardcoding connection string in code       | Security risk if code is shared                                 | Store in `appsettings.json`                 |
| Wrong `DefaultConnection`key name        | `GetConnectionString()`returns null → NullReferenceException | Key in `appsettings.json`must match exactly |

---

## ⭐ Interview Quick-Fire

| Question                                    | Answer                                                                         |
| ------------------------------------------- | ------------------------------------------------------------------------------ |
| What is a connection string?                | Text with server location, database name, and auth details                     |
| Two authentication methods?                 | Windows Auth (`Trusted_Connection=True`) and SQL Auth (`User Id/Password`) |
| Best auth for development?                  | Windows Authentication                                                         |
| Best auth for production/remote?            | SQL Server Authentication                                                      |
| Where to store connection string in MVC?    | `appsettings.json`→`ConnectionStrings`section                             |
| How to read it in code?                     | `config.GetConnectionString("DefaultConnection")`                            |
| What is connection pooling?                 | Reusing existing connections instead of creating new ones                      |
| Does `con.Close()`destroy the connection? | No — returns it to the pool for reuse                                         |
| `Server=.`means what?                     | Local machine, default SQL Server instance                                     |
| Why double backslash in JSON?               | `\`is a JSON escape character — must be written as `\\`                   |
